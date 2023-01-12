## TIL (Today I Learned)

### 1월 12일 (목)   

- ### [학습내용] ArraySlice
    이전에 Swift에서 문자열 처리에 대해 학습해본적이 있습니다.
    split 메서드로 문자열을 분리할때 반환되는 타입이 SubSequence Array 타입이였어요.
    즉 여기서 다음 포스팅에서 짚고 넘어가자고한 SubSequence라는것은 ArraySlice의 별칭이기에 ArraySlice에 대해 간단히 알아보겠습니다🥸
    혹시 문자열 처리를 먼저 보고 오고 싶으시면 아래 포스팅을 참고해주세요!
    https://green1229.tistory.com/318
    
    #### ArraySlice?

    공식문서에 의하면 ArraySlice는 Array, ContiguousArray 혹은 ArraySlice 인스턴스의 slice입니다.
    여기서 알 수 있는것은 ArraySlice 인스턴스 조차도 ArraySlice의 요소인것이죠.
    ```swift
    @frozen struct ArraySlice<Element>
    ```
    선언은 요렇게 구조체로 제네릭하게 데이터 타입을 받아올 수 있도록 되어있습니다.
    해당 타입의 장점은 배열이 주어졌을때 섹션에서 작업을 빠르고 효율적으로 수행할 수 있어요.
    즉, 어떠한 배열이 주어지고 이에 대해 범위를 가지고 쪼개거나 했을때 새로운 Array를 만든다면 메모리를 별도로 할당 받고 잡아먹게 되죠.
    그런데 ArraySlice를 이용하면 새 스토리지로 복사하는 대신 인스턴스는 기존 Array에 대한 뷰를 제공합니다.
    그렇기에 일반적으로 동일 인터페이스를 제공하기에 기존 Array에서 수행하는것과 동일하게 해줄 수 있습니다.
    핵심은 메모리 공간 절약이라고 생각됩니다😲

    그럼 한번 예시를 간단히 들어보죠!

    #### ArraySlice 사용 예시
    ```swift
    let absences = [0, 2, 0, 4, 0, 3, 1, 0]
    let midpoint = absences.count / 2

    let firstHalf = absences[..<midpoint]
    let secondHalf = absences[midpoint...]
    ```
    absences라는 기존 배열이 존재하고 이를 반으로 나는 firstHalf와 secondHalf가 있어요.
    이 때 두 타입은 기존 배열의 범위를 가져왔기에 ArraySlice가 됩니다.
    그렇기에 firstHalf와 secondHalf은 별도 메모리 공간을 따로 갖지 않고 기존 absences를 참조합니다.
    즉, 메모리를 절약할 수 있어요.
    또한 아래와 같이 활용될 수 있습니다.
    ```swift
    let firstHalfSum = firstHalf.reduce(0, +)
    let secondHalfSum = secondHalf.reduce(0, +)

    if firstHalfSum > secondHalfSum {
        print("More absences in the first half.")
    } else {
        print("More absences in the second half.")
    }
    // Prints "More absences in the first half."
    ```
    새로운 배열을 만들고 이를 판별하는것보다 이런 로직을 구성하여 판별해줘야할때 새로 메모리를 만들 필요가 없죠.
    메모리를 절약하면서 로직을 구현할 수 있게 됩니다.

    반면 ArraySlice가 메모리 성능에 도움이 되는것은 확실한데 어떠한 경우에는 조심히 사용해야 됩니다.

    #### ArraySlice 사용 유의사항

    첫째로, 인스턴스를 오래 저장하는것에는 권장되지 않습니다.
    기존 Array의 수명이 끝난 후에도 ArraySlice로 표시하는 부분을 넘어 기존 Array의 전체 스토리지에 대한 참조를 보유하고 있게 됩니다.
    따라서, 기존 Array의 수명이 끝났음에도 사용하고 있다면 액세스 할 수 없으며 메모리 릭이 발생할 여지가 다분해집니다.

    둘째로, Array와는 달리 인스턴스의 시작 index가 항상 0이 시작 index가 아닙니다.
    기존 Array를 참조하며 동일한 index를 유지하게 됨으로 시작 index가 생성되는 방법에 따라 달라지게 됩니다.
    예시를 한번 보시죠🙋🏻
    ```swift
    let basicArr = ["GREEN", "YELLOW", "RED", "BLUE"]
    let firstArr = basicArr[..<2]
    let secondArr = basicArr[2...]

    print(firstArr[0]) // "GREEN"
    print(secondArr[0]) // Fatal error: Index out of bounds
    ```
    이렇게 secondArr는 기존 배열의 2 index부터 사용했기에 0번째 인덱스로 접근하게 되면 제일 무서운 파탈 에러를 발생시킵니다.
    즉, 기존 Array를 참조하기에 그렇습니다.
    만약 해당 secondArr의 첫 요소를 추출하고 싶다면 아래와 같이 변경할 수 있습니다.
    ```swift
    print(secondArr.first) // Optional("GREEN")
    ```
    first는 없을 수도 있으니 옵셔널 타입입니다.
    또한 해당 인덱스의 시작과 끝이 참조되는 어떤 인덱스인지 가져오고 싶다면 아래와 같이도 할 수 있습니다.
    ```swift
    print(secondArr.startIndex) // 2
    print(secondArr.endIndex) // 3
    ```
    이렇게 기존 배열의 참조된 범위의 인덱스를 가져와 알맞게 사용할 수도 있습니다.

    #### 마무리

    메모리 성능을 위해 이런것까지 고려해본다는게 재밌고 더 많이 Swift에 대해 알아가는것 같네요🕺🏻

    #### [참고 자료]
    https://green1229.tistory.com/321   
    https://developer.apple.com/documentation/swift/arrayslice
