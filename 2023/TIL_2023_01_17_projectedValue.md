## TIL (Today I Learned)

### 1월 17일 (화)    

- ### [학습내용] projectedValue
    #### projectedValue
    인스턴스 프로퍼티로 상태 값에 대한 바인딩입니다.

    선언은 아래와 같아요.
    ```swift
    var projectedValue: Binding<Value> { get }
    ```
    SwiftUI에서 사용을 먼저 봐볼께요.
    ```swift
    struct PlayerView: View {
      var episode: Episode
      @State private var isPlaying: Bool = false

      var body: some View {
        VStack {
          Text(episode.title)
            .foregroundStyle(isPlaying ? .primary : .secondary)
          PlayButton(isPlaying: $isPlaying)
        }
      }
    }
    ```
    @State 프로퍼티 isPlaying이 있습니다.

    해당 값을 하위 PlayButton 뷰에 전달해줄때 우리는 $를 사용해서 바인딩 된 값을 전달해줘요.

    그런데 이 바인딩 된 값은 어떻게 나오게 된걸까요?

    그것이 바로 projectedValue입니다.

    선언을 봤듯이 Binding 타입이고 Value가 제네릭하게 들어가게 됩니다.

    즉 바인딩할 수 있는 값으로 변환 시켜주는것이죠.

    projectedValue를 사용할때는 $를 붙여 사용합니다.



    잠깐 그럼 이게 끝?
    SwiftUI에서의 projectedValue 사용은 그렇다는것이고 개념적으로 한단계 더 들어가볼께요.




    우선 projectedValue는 Property Wrapper에서 사용하게 됩니다.

    즉, 해당 구현 내부에서 projectedValue를 가지고 원하는 형태와 값을 만들어내고 이를 $ 키워드를 붙여 사용할 수 있게 해줍니다.

    그런데 우리는 Property Wrapper에서 wrappedValue에 대해서 이전 포스팅에서 알아봤어요.

    어떤 차이가 있을까요?



    wrappedValue는 Property Wrapper로 구현 시 내부 변수 값에 접근하게 됩니다.

    즉 Property Wrapper로 정의해둔 기능과 값을 쓰는것이죠.

    반면 projectedValue는 해당 Property Wrapper에서 정의한 값을 반환해주는 차이가 있습니다.



    이에 대해 예시로 한번 보겠습니다🙌
    ```swift
    @propertyWrapper
    struct Person {
      let name: String
      var wrappedValue = 20
      var projectedValue: String { self.name }

      init(name: String) {
        self.name = name
      }
    }

    @Person(name: "green") var green
    print(green) // 20
    print($green) // "green"
    ```
    이런 차이가 있습니다.

    projectedValue는 이미 Property Wrapper를 사용한 Person 타입을 만들때 정의해뒀습니다.

    green이라는 @Person 프로퍼티를 만들고 $로 접근하게 되면 해당 정의된 값을 반환하게 됩니다.

    반면 green이라고만 접근하면 Property Wrapper에서 지정해둔 wrappedValue를 반환하게 되죠!



    그렇기에 처음으로 쭈욱 돌아가보면 SwiftUI에서 @State 내부에 projectedValue는 해당 값을 바인딩으로 감싼 타입으로 지정되어 있게 됩니다.

    즉 우리는 하위 뷰에 해당 값을 넘겨줄때 projectedValue를 통해 바인딩된 값을 넘겨야함으로 $를 붙여주게 되었습니다🙌 



    #### 마무리
    $를 왜 붙여서 바인딩을 표현하는지 사실 근본적으로 궁금했었는데 이번 기회에 도움이 된것 같습니다😄



    #### [참고 자료]
    https://green1229.tistory.com/322
    https://developer.apple.com/documentation/swiftui/state/projectedvalue
    https://ios-development.tistory.com/895