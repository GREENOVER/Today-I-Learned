## TIL (Today I Learned)

### 3월 3일 (금)    

- ### [학습내용] Swift Concurrency - Task (4)
    #### Task 자동 재시도

    우리는 네트워크 통신을 할때 한번에 응답되지 않는 상황들도 있습니다.
    클라의 잘못된 요청이 아닌 서버에서 내부적인 아주 잠깐의 이슈라던지 하는 문제가 발생할 수 있죠.
    즉 이런 경우 동일하게 실패한 비동기 Task 요청을 자동으로 재시도 해야합니다.

    만약 Swift Concurrency를 사용하지 않고 우리가 그전에 익숙한 Combine을 사용한다면 아래와 같이 retry로 에러 처리 전 최대 3번의 재시도를 거치게 되죠!
    ```swift
    struct SettingsLoader {
        var url: URL
        var urlSession = URLSession.shared
        var decoder = JSONDecoder()

        func load() -> AnyPublisher<Settings, Error> {
            urlSession
                .dataTaskPublisher(for: url)
                .map(\.data)
                .decode(type: Settings.self, decoder: decoder)
                .retry(3)
                .eraseToAnyPublisher()
        }
    }
    ```
    즉 이런식으로 말이죠!
    SettingLoader라는 서비스 구현체에서 load를 통해 데이터를 받아오는데 여기서 실패할 경우 최대 3번까지 재시도 할 수 있죠.
    여기서는 에러의 종류와는 전혀 관계없이 무조건 로드 작업을 재시도 하게 됩니다.

    만약 그렇다면 Task를 사용하는 Swift Concurrency에서는 어떻게 구현할까요?

    Combine에서는 Publisher 프로토콜 위에 retry라는 연산자가 내장 API로 포함되어 있지만 Concurrency에서는 아쉽게도 이런것을 쉽게 사용하도록 연산자나 메서드를 제공하진 않습니다😭

    그렇기에 직접 코드를 작성해봐야하죠!

    어렵지 않은것이 Concurrency에서의 좋은 측면 중 하나는 명령문 및 루프에서 async await와 같은 비동기 호출을 혼합하여 사용할 수 있다는 것입니다.
    즉 자동 재시도를 구현하는 방법은 아래처럼 반복되는 루프 내에서 비동기 코드를 실행하도록 배치하는 것입니다.
    ```swift
    struct SettingsLoader {
        var url: URL
        var urlSession = URLSession.shared
        var decoder = JSONDecoder()

        func load() async throws -> Settings {
            // Perform 3 attempts, and retry on any failure:
            for _ in 0..<3 {
                do {
                    return try await performLoading()
                } catch {
                    // This 'continue' statement isn't technically
                    // required, but makes our intent more clear:
                    continue
                }
            }

            // The final attempt (which throws its error if it fails):
            return try await performLoading()
        }

        private func performLoading() async throws -> Settings {
            let (data, _) = try await urlSession.data(from: url)
            return try decoder.decode(Settings.self, from: data)
        }
    }
    ```
    보시면 for 문안에 3번까지 돌면서 do를 시키죠.
    여기서 performLoading을 호출하는데 실패한다면 3번까지 다시 루프를 돌며 실행할 것이고 성공한다면 return으로 탈출 하게 됩니다.
    여기서 최악의 경우에는 총 4번 실행되요.
    이유는 루프를 돌며 3번 실행하고 거기서도 실패하면 루프를 탈출해서 만나는 동일한 performLoading을 실행시키죠.
    즉 3번 재시도, 최악의 경우 4번 실행 이 루트입니다!

    이것도 보시면 이렇게 for문으로 배치해서 매번 사용하기 번거롭잖아요?

    그렇기에 이전 Task 지연에서도 Task를 확장해 구현해놓은것처럼 편리하게 확장시켜볼 수 있습니다.
    ```swift
    extension Task where Failure == Error {
        @discardableResult
        static func retrying(
            priority: TaskPriority? = nil,
            maxRetryCount: Int = 3,
            operation: @Sendable @escaping () async throws -> Success
        ) -> Task {
            Task(priority: priority) {
                for _ in 0..<maxRetryCount {
                    try Task<Never, Never>.checkCancellation()

                    do {
                        return try await operation()
                    } catch {
                        continue
                    }
                }

                try Task<Never, Never>.checkCancellation()
                return try await operation()
            }
        }
    }
    ```
    checkCancellation은 Task에서 제공해주기에 적절히 사용할 수 있습니다.
    로직은 동일하게 루프를 돌며 취소되었는지 먼저 확인하고 아니라면 비동기 실행을 해줍니다.

    여기서 한단계 더 나아가서 만약 서버에서 내부적으로 아주 잠깐의 이슈였다면 재시도 사이의 조금의 지연을 주는것이 좋을것 같아요.

    이전에 배운 Task.sleep으로 이걸 더 보완해볼 수 있습니다.
    ```swift
    extension Task where Failure == Error {
        @discardableResult
        static func retrying(
            priority: TaskPriority? = nil,
            maxRetryCount: Int = 3,
            retryDelay: TimeInterval = 1,
            operation: @Sendable @escaping () async throws -> Success
        ) -> Task {
            Task(priority: priority) {
                for _ in 0..<maxRetryCount {
                    do {
                        return try await operation()
                    } catch {
                        let oneSecond = TimeInterval(1_000_000_000)
                let delay = UInt64(oneSecond * retryDelay)
                try await Task<Never, Never>.sleep(nanoseconds: delay)

                        continue
                    }
                }

                try Task<Never, Never>.checkCancellation()
                return try await operation()
            }
        }
    }
    ```
    실패하여 catch로 떨어지면 지연을 걸어주는 것이죠.

    추가로 이 메서드를 요렇게 사용할 수 있겠네요!
    ```swift
    struct SettingsLoader {
        var url: URL
        var urlSession = URLSession.shared
        var decoder = JSONDecoder()

        func load() async throws -> Settings {
            try await Task.retrying {
                let (data, _) = try await urlSession.data(from: url)
                return try decoder.decode(Settings.self, from: data)
            }
            .value
        }
    }
    ```
    retrying 메서드를 호출하여 사용하는데 마지막에 .value가 붙은게 보이시죠?
    이 value 속성을 사용해 해당 클로저의 반환된 값을 관찰할 수 있습니다.
    즉, 클로저에서 디코딩 후 리턴한 값에 대해 얻어올 수 있다는 얘기죠.

    #### 결론

    Task를 확장하여 더 많은 발전된 사용을 가져갈 수 있습니다.
    재시도도 마찬가지로 재시도 시 더 다양한 동작을 구현해줄 수도 있죠👍

    #### 마무리

    Task에 대해 1~4까지 조금씩 끊어서 알아보면서 직접 바로 상용 프로덕트에 적용해볼 구석이 많이 보였습니다!
    아니까 되게 편리하네요ㅎㅎ

    #### [참고 자료]
    https://green1229.tistory.com/335   
    https://www.swiftbysundell.com/articles/retrying-an-async-swift-task/