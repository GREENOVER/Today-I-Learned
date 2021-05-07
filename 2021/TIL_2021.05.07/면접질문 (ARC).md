# 면접 질문 학습

### ARC (Fork by 제르시)

  - ARC란 무엇인지 설명하시오.
    - Automatic Reference Counting
    - 앱 성능을 위해 자동으로 참조 횟수를 추적하여 메모리를 관리해주는 모델
    - 힙 영역 메모리 관리 (스택 메모리는 자동 제거됨)
    - 컴파일러가 알아서 retain, release를 넣어 메모리 관리
    - MRC에 비해 코드양 적어지고 안정적으로 메모리 관리 가능
    - RC: 참조횟수로 인스턴스를 가진 객체의 수를 저장한 것
    - RC 1 증가시키면 retain 호출하고 해제 시키면 release (nil로 저장하는게 소유권 포기하는것)
    - Strong, Weak, Unowned
    - 참조로 클래스의 인스턴스만 적용 (컴파일 타임에 자동으로 분석 후 RC 코드 삽입)
- Retain Count 방식에 대해 설명하시오.
  - 인스턴스 생성
  - 수동으로 retain 메서드 호출하여 증가
- Strong 과 Weak 참조 방식에 대해 설명하시오.
  - Strong: 해당 인스턴스에 대한 소유권 가짐, retain count 증가시킴, 값 지정 시 reatain, 참조 종료 시 release, 메모리 누수 가능
  - Weak: 해당 인스턴스에 대한 소유권 안가짐, 주소값만 가짐, retain count 증가 안함
- ARC 대신 Manual Reference Count 방식으로 구현할 때 꼭 사용해야 하는 메서드들을 쓰고 역할을 설명하시오.
  - retain: RC값 증가시킴, 인스턴스 생성 시에는 자동으로 증가시킴으로 제외
  - release: RC값 감소시킴
- retain 과 assign 의 차이점을 설명하시오.
  - 리테인은 set/get 시 RC올려주고 소유권 획득, 어사인은 값만 set/get하고 RC 증가시키지 않음(unowed 같은것)
- 순환 참조에 대하여 설명하시오.
  - 참조 타입의 프로퍼티가 서로를 강한참조할때 각 인스턴스 해제가 되지 않아 메모리에서 사라지지 않음
- 특정 객체를 autorelease 하기 위해 필요한 사항과 과정을 설명하시오.
  - ARC 사용할때 @autoreleasepool 블록 사용
- Autorelease Pool을 사용해야 하는 상황을 두 가지 이상 예로 들어 설명하시오.
  - UI 프레임워크 기반으로 하지 않는 파운데이션 기반인 프로그램 시
  - 수많은 임시 객체 생성하는 루프 작성, 보조 스레드 생성
    

### 참고자료
  - https://github.com/JeaSungLEE/iOSInterviewquestions