## TIL (Today I Learned)

### 1월 6일 (금)   

- ### [학습내용] 
   ### Swift의 문자열 처리 (split vs components)

    Swift 언어의 특성상 문자열을 타 언어보다 다루기가 조금 까다로워요🥲
    쉽게 Subscript를 통해 Int 인덱스 참조가 불가능하여 쉽게 문자열에서의 문자를 빼올 수 도 없고 index를 통해서 가능하게 해주죠.
    이와 관련해서 Swift에서는 도대체 왜 그런지 궁금하시다면 아래 포스팅을 먼저 보고 오셔도 좋습니다!
    https://green1229.tistory.com/286

    그럼 이어서 오늘 할 주제는 Swift의 문자열을 분리시키는 두 메서드를 알아보겠습니다.
    문자열을 분리시킨다는 목적은 동일한데 실제로 어떻게 다른지 하나씩 짚어볼께요🕺🏻

    #### split(separator:maxSplits:omittingEmptySubsequences:)

    우선 공식문서에서는 split은 지정된 요소와 동일한 요소 주변에서 컬렉션의 가능한 가장 긴 하위 시퀀스를 순서대로 반환한다고 합니다.
    String 타입의 인스턴스 메서드이구요.
    지정된 요소라 치면 앞으로 알아볼 구분자를 뜻하는것 같습니다.
    선언은 아래와 같습니다.
    ```swift
    func split(
        separator: Self.Element,
        maxSplits: Int = Int.max,
        omittingEmptySubsequences: Bool = true
    ) -> [Self.SubSequence]
    ```
    보시면 세개의 파라미터를 받아요. 하나씩 어떤 역할을 하는지 보겠습니다ㅎㅎ

     1. separator

    문자열 분리를 위해 어떤 구분자를 가지고 분리를 해줄 지에 대한 요소입니다.
    예를들어 아래 코드를 보시죠.
    ```swift
    let str = "I am Green"
    let splitedStr = str.split(separator: " ")

    print(splitedStr) // ["I", "am", "Green"]
    ```
    띄어쓰기 즉 공백을 구분자로 두었을때 3개의 문자열 서브시퀀스로 나눠집니다.

    그럼 하나 더 보시죠!
    ```swift
    let str = "I am Green"
    let splitedStr = str.split(separator: "am")

    // error
    ```
    이렇게 만약 문자열을 구분자로 두면 String.Element의 구분자 타입이 아님으로 컴파일 에러가 발생합니다.
    즉, 구분자에는 Character 타입만 들어가야 됩니다🙋🏻

    그럼 공백이 아닌 속한 문자를 넣는다면 어떻게 될까요?
    ```swift
    let str = "I am Green"
    let splitedStr = str.split(separator: "a")

    print(splitedStr) // ["I ", "m Green"]
    ```
    이렇게 구분자 기준으로 분리가 됩니다.

    마지막으로 문자열에 들어있지 않는 구분자 문자를 주게 되면 어떻게 될까요?
    ```swift
    let str = "I am Green"
    let splitedStr = str.split(separator: "b")

    print(splitedStr) // ["I am Green"]
    ```
    문자열 자체를 서브시퀀스 배열로 리턴해줍니다!

    그럼 얼추 separator에 대해서는 감이 오셨을거라 생각하고 두번째 요소로 넘어가겠습니다🕺🏻

    2. maxSplits

    분리할 최대 횟수를 지정할 수 있습니다.
    기본값으로는 거의 무한정인 Int max값이에요.
    즉 따로 지정하지 않으면 구분자를 통해 모두 분리시킵니다.
    그럼 지정해준다면 어떨지 한번 보시죠!
    ```swift
    let str = "I am Green"
    let splitedStr = str.split(separator: " ", maxSplits: 1)

    print(splitedStr) // ["I", "am Green"]
    ```
    요렇게 공백은 2개 존재하지만 맥스값으로준 1의 공백 갯수 만큼만 분리시킵니다.
    am과 Green 사이에도 공백이 있지만 이미 맥스값 1개를 넘었기에 합쳐서 분리가 됩니다.

    자 이것도 아주 쉽죠! 그럼 조금 처음보는 omittingEmptySubsequences에 대해 알아보겠습니다🕺🏻

    3. omittingEmptySubsequences

    omit이라는 단어는 생략을 의미합니다.
    즉 빈 서브시퀀스를 생략한다는 옵션 같아요.
    우선 Bool 타입으로 기본 값은 true입니다.
    true로 되있기에 별도 설정이 없다면 마지막 요소에 빈 서브시퀀스를 제외하고 반환해주게 되요.
    즉 아래와 같이 우리가 쉽게 볼 수 있는 형식이죠.
    ```swift
    let str = "I am Green "
    let splitedStr = str.split(separator: " ", omittingEmptySubsequences: true)

    print(splitedStr) // ["I", "am", "Green"]
    ```
    만약 false라면 마지막 요소에 빈 서브시퀀스를 추가하여 반환해줍니다.
    이렇게 되겠죠?
    ```swift
    let str = "I am Green "
    let splitedStr = str.split(separator: " ", omittingEmptySubsequences: false)

    print(splitedStr) // ["I", "am", "Green", ""]
    ```
    마지막 요소에 빈 서브시퀀스가 추가된것을 확인하실 수 있습니다ㅎㅎ
    정리해볼께요!
    true면 비어 있지 않은 하위 시퀀스만 반환하고 false면 비어있는 시퀀스도 반환해줍니다🙌

    split의 반환 타입은 SubSequence 타입으로 서브시퀀스는 ArraySlice의 별칭입니다.
    ArraySlice에 대해서는 조만간 포스팅을 해보겠습니다ㅎㅎ

    참고로 split의 시간 복잡도는 컬렉션의 길이만큼인 O(n)을 가지게 됩니다🥸

    그럼 이어서 components에 대해 알아보시죠📣

    #### components(separatedBy:)

    split과 역할은 동일하지만 조금 더 간단할 수 있습니다.
    파라미터가 보시면 하나밖에 없거든요ㅎㅎ
    공식문서에서는 지정된 구분자로 나눠진 하위 문자열을 포함하는 문자열 배열을 반환한다고 합니다.
    ```swift
    func components(separatedBy separator: String) -> [String]
    ```

    메서드 시그니쳐를 보시면 Split에서는 Character 타입을 구분자로 받았지만 components는 String, 즉 문자열을 구분자로 받습니다.
    split에서는 아래와 같이 문자열로 쪼갤 수 없었다면 components는 가능합니다.
    ```swift
    let str = "I am Green"
    let splitedStr = str.components(separatedBy: "am")

    print(splitedStr) // ["I ", " Green"]
    ```
    다만 components는 Foundation 프레임워크에 속해 있기에 Foundation을 임포트 해줘야합니다!


    #### split vs components의 성능차이

    둘의 성능적으로의 큰 차이는 구분자를 빈 문자열로 두고 문자열 내에 연속된 빈 공백 문자가 많을때 차이가 납니다.
    왜냐하면 split에서는 omittingEmptySubsequences라는 옵션이 있어 빈 시퀀스를 제외해줄 수 있죠.
    아래 예시를 보시죠!
    ```swift
    let str = "I am Green         "
    let splitedStr = str.split(separator: " ")

    print(splitedStr) // ["I", "am", "Green"]
    ```
    Green 뒤에 많은 공백이 들어가도 빈 시퀀스를 제거해줍니다.
    그런데 components는 어떨까요?
    ```swift
    let str = "I am Green       "
    let splitedStr = str.components(separatedBy: " ")

    print(splitedStr) // ["I", "am", "Green", "", "", "", "", "", "", ""]
    ```
    이렇게 빈 시퀀스를 제외하지 않고 있습니다.
    즉 이러한 빈 시퀀스가 많아진다면 components의 성능은 당연히 split보다 낮아질것입니다👀
    이러한 부분들을 참고해서 골라 사용하면 좋을것 같네요!

    #### 마무리

    split과 components를 다루는것에 있어 딥하게 들어가면 정말 많은 차이점을 가지고 있었네요.
    사실 프로젝트를 진행하면서 다뤘던것보다 알고리즘을 학습하고 문제를 풀면서 다뤄봤던 경험이 더 많은것 같아요.
    그래도 명확한 차이를 이번에 확실히 짚고 넘어가게 되었습니다🥸

    #### [참고 자료]

   https://green1229.tistory.com/318 https://developer.apple.com/documentation/swift/string/split(separator:maxsplits:omittingemptysubsequences:) 
   https://developer.apple.com/documentation/foundation/nsstring/1413214-components