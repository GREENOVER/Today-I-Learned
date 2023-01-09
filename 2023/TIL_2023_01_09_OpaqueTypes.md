## TIL (Today I Learned)

### 1월 9일 (월)   

- ### [학습내용] 
   ### Opaque Types
    우선 SwiftUI에서 가장 많이 접해볼 수 있는 키워드가 있습니다.
    바로 some이라는 키워드인데요.
    ```swift
    struct CustomView: View {
      var body: some View {
        // CustomView 구현
      }
    }
    ```
    여기에 View 프로토콜 앞에 some이라는 키워드 보이시죠?
    이게 오늘 배워볼 opaque type입니다.
    Swift 5.1에서 새롭게 나온 Swift의 기능이고 iOS 13부터 적용되죠.

    자 그럼 서론은 여기까지고 Opaque Types이 뭔지 이제 알아보겠습니다🕺🏻

    #### Opaque Types?

    opque의 사전적인 의미는 불투명하다는 뜻을 가지고 있습니다.
    그래서 오늘 학습해볼 타입을 추측해보면 불투명한 타입에 대해 알아보겠다는걸 추측할 수 있습니다.

    Swift 프로그래밍 가이드에서는 이렇게 소개하고 있어요.
    불투명한 반환 유형이 있는 함수 혹은 메서드는 반환 값의 타입 정보를 숨깁니다.
    즉, 구체적인 반환 타입을 가리고 프로토콜 혹은 프로토콜을 채택하는 타입을 반환하는 역할을 합니다.
    이러한 역할을 통해 정보를 숨기는것이 모듈과 모듈을 호출하는 코드 사이의 경계에 유용합니다.
    타입이 프로토콜 유형인 값을 반환하는것과 반대로 opaque type은 타입 ID를 유지합니다.
    컴파일러는 타입 정보에 엑세스할 수 있지만 모듈의 클라이언트는 액세스할 수 없습니다.

    자 그럼 Opaque Type으로 어떤 이점을 가져올 수 있는지 봐보시죠!


    우선 프로토콜을 사용한 예시를 보겠습니다.
    ```swift
    protocol Shape {
      func draw() -> String
    }

    struct Triangle: Shape {
      var size: Int
      func draw() -> String {
        var result: [String] = []
        for length in 1...size {
          result.append(String(repeating: "*", count: length))
        }
        return result.joined(separator: "\n")
      }
    }
    let smallTriangle = Triangle(size: 3)
    print(smallTriangle.draw())
    // *
    // **
    // ***
    ```
    별을 찍는 draw 함수를 가진 Shape이라는 프로토콜이 있을때 이 프로토콜을 채택하는 Triangle이라는 타입의 구조체를 만듭니다.
    이 구조체 인스턴스의 draw 메서드를 호출하여 사용할 수 있죠.
    이 모습은 Swift 언어 사용에 있어 익숙한 그림일거에요.
    만약 이 그림에서 반전된 형태의 그림이 있다고 쳐봅시다.
    ```swift
    struct FlippedShape<T: Shape>: Shape {
      var shape: T
      func draw() -> String {
        let lines = shape.draw().split(separator: "\n")
        return lines.reversed().joined(separator: "\n")
      }
    }
    let flippedTriangle = FlippedShape(shape: smallTriangle)
    print(flippedTriangle.draw())
    // ***
    // **
    // *
    이럴때 shape 프로퍼티는 Shape의 제네릭한 타입으로 지정될 수 있습니다.
    또한 이렇게 제네릭에 다양한 타입들을 동시에 담아 구현할 수도 있죠.
    struct JoinedShape<T: Shape, U: Shape>: Shape {
      var top: T
      var bottom: U
      func draw() -> String {
        return top.draw() + "\n" + bottom.draw()
      }
    }
    let joinedTriangles = JoinedShape(top: smallTriangle, bottom: flippedTriangle)
    print(joinedTriangles.draw())
    // *
    // **
    // ***
    // ***
    // **
    // *
    ```
    이렇다면 사실 인스턴스 생성 시 내부 정보를 노출하기에 반환 타입을 명시하게 되고 공용 인터페이스의 일부가 아닌 타입이 누출됩니다.
    Shape을 사용하는 모듈 외부의 코드는 변환에 대한 구현 세부 사항들을 설명할 필요는 사실 없는데 이에 위배되게 됩니다.

    이를 Opaque Type으로 해결해볼 수 있습니다.


    Opaque Type을 제네릭 형식의 역순으로 생각해볼 수 있습니다.
    제네릭을 사용하면 위와 같이 코드에서 해당 함수의 매개 변수에 대한 형식을 선택하고 함수 구현에서는 추상화된 방식으로 리턴 값을 반환할 수 있습니다.
    이런 역할이 Opaque Type이 있는 함수의 경우에는 반전됩니다.
    Opaque Type를 사용해 함수 구현이 함수를 호출하는 코드에서 추상화된 방식으로 리턴 값의 유형을 선택할 수 있게 됩니다.
    아래 예시를 한번 보시죠!
    ```swift
    struct Square: Shape {
      var size: Int
      func draw() -> String {
        let line = String(repeating: "*", count: size)
        let result = Array<String>(repeating: line, count: size)
        return result.joined(separator: "\n")
      }
    }

    func makeTrapezoid() -> some Shape {
      let top = Triangle(size: 2)
      let middle = Square(size: 2)
      let bottom = FlippedShape(shape: top)
      let trapezoid = JoinedShape(
        top: top,
        bottom: JoinedShape(top: middle, bottom: bottom)
      )
      return trapezoid
    }
    let trapezoid = makeTrapezoid()
    print(trapezoid.draw())
    // *
    // **
    // **
    // **
    // **
    // *
    ```
    해당 구현을 통하면 함수는 특정 Struct의 유형을 지정하지 않고 프로토콜을 준수하는 리턴 타입의 값을 반환합니다.
    즉 공용 인터페이스의 일부에서 Shape을 생성하는 특정 타입을 만들지 않아도 makeTrapezoid()를 작성하면 공용 인터페이스의 기본적인 측면을 표현할 수 있게 됩니다.
    이렇게 되면 결국 리턴 타입을 변경하지 않아도 다양한 방법으로 다르게 Shape을 그릴 수 있는 함수를 작성할 수 있습니다.
    some을 이용해서 말이죠!
    처음 위에 프로토콜을 이용하던것을 아래와 같이 바꿀 수도 있게 됩니다.
    ```swift
    func flip<T: Shape>(_ shape: T) -> some Shape {
      return FlippedShape(shape: shape)
    }
    func join<T: Shape, U: Shape>(_ top: T, _ bottom: U) -> some Shape {
      JoinedShape(top: top, bottom: bottom)
    }

    let opaqueJoinedTriangles = join(smallTriangle, flip(smallTriangle))
    print(opaqueJoinedTriangles.draw())
    // *
    // **
    // ***
    // ***
    // **
    // *
    ```
    주의할 점은 만약 Opaque Tpye의 함수가 여러 위치에서 반환되는 경우에는 모든 반환 값의 타입을 동일하게 맞춰줘야합니다.
    즉 단일 유형으로 동일하게 리턴을 시켜줘야합니다.
    ```
    func invalidFlip<T: Shape>(_ shape: T) -> some Shape {
      if shape is Square {
        return shape // Error: return types don't match
      }
      return FlippedShape(shape: shape) // Error: return types don't match
    }
    ```
    위의 경우 이를 위반한 경우인데 에러가 나게 됩니다.
    Shape 타입과 FlippedShape 타입이 다르기에 위배가 되죠.

    그럼 마지막으로 Opaque Type과 프로토콜 타입의 차이점을 짚고 정리해보겠습니다🥸

    #### Opaque Type vs Protocol Type

    Opaque Type은 하나의 특정 타입을 참조하지만 함수 호출자는 타입을 확인할 수 없습니다.
    반면 프로토콜 타입은 프로토콜을 준수하는 모든 타입을 참조할 수 있습니다.
    프로토콜 타입은 저장하는 값의 기본 타입에 대해 더 많은 유연성을 제공하고 Opaque Type을 사용하면 이러한 기본 타입에 대해 더 강력하게 보증을 할 수 있습니다.
    예시로 프로토콜 타입을 리턴 타입으로 사용하는걸 보시죠.
    ```swift
    func protoFlip<T: Shape>(_ shape: T) -> Shape {
      return FlippedShape(shape: shape)
    }
    위 코드는 매번 동일한 타입의 값을 반환합니다.
    즉 Shape 프로토콜을 준수하기만 하면 됩니다.
    func protoFlip<T: Shape>(_ shape: T) -> Shape {
      if shape is Square {
        return shape
      }

      return FlippedShape(shape: shape)
    }
    ```
    이렇게 수정하더라도 어떤 Shape가 전달되었는지에 따라 Squre의 인스턴스 혹은 FlippedShape의 인스턴스를 반환합니다.
    다만 이 함수는 타입 정보에 의존하는 다른 많은 작업을 할 수 없습니다.
    예시로 == (equal) 메서드를 통해 리턴 값을 비교할 수 없게 됩니다.
    ```swift
    let protoFlippedTriangle = protoFlip(smallTriangle)
    let sameThing = protoFlip(smallTriangle)
    protoFlippedTriangle == sameThing  // Error
    ```
    Shape가 프로토콜 요구 사항에서 == 메서드를 포함하지 않았기에 발생하는 문제죠.

    반면 Opaque Type은 기본 타입의 ID를 유지합니다.
    Swift는 프로토콜 타입을 반환 값으로 사용할 수 없는 위치에서 Opaque Type 반환 값을 사용할 수 있도록 관련 타입을 유추할 수 있습니다.
    즉 이러한 것들의 사용을 가능케 해주죠.
    ```swift
    protocol Container {
      associatedtype Item
      var count: Int { get }
      subscript(i: Int) -> Item { get }
    }
    extension Array: Container { }

    func makeOpaqueContainer<T>(item: T) -> some Container {
      return [item]
    }
    let opaqueContainer = makeOpaqueContainer(item: 12)
    let twelve = opaqueContainer[0]
    print(type(of: twelve))
    // Prints "Int"
    ```
    이러한 특성들을 통해 SwiftUI의 View 프로토콜에서는 some View의 Opaque Type을 사용하여 View 프로토콜 자체를 반환 타입으로 지정하기 위해 some 키워드를 사용하는것입니다.
    들어갈 구조체들이 UI를 그리기 위한 컴포넌트 즉 요소로 사용됨을 보장하고 이용하게 되는것이죠.

    #### 마무리

    Opaque Type을 막연하게 내부를 숨기는 정도로만 알았지 어떤 근거를 가지고 동작하는지에 대해 깊게 고민을 하지 못하였는데 이번 포스팅을 통해 아직 다 통달한건 아니지만 큰 성과가 있었습니다.
    Swift의 다양한 면모들을 하나씩 파해치는 느낌이랄까요..?

    #### [참고 자료]
    https://green1229.tistory.com/320   
    https://docs.swift.org/swift-book/LanguageGuide/OpaqueTypes.html