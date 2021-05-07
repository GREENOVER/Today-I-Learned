# 면접 질문 학습

### AutoLayout (Fork by 제르시)

  - 오토레이아웃을 코드로 작성하는 방법은 무엇인가? (3가지)
    - NSLayoutConstraint: 인스턴스로 구현되지만 권장되지 않음 (메모리적 측면에서 제약을걸때마다 호출함으로 비효율적이라 생각함), 레이아웃 영향이 없더라도 각 매개변수 값 지정해야함
    - Layout Anchor: 앵커를 통해 구현
    - Visul Format Language: 문자열과 같은 아스키 특정 형식을 이용
  - hugging, resistance에 대해서 설명하시오.
    - hugging: Content hugging, 늘어나지 않으려는 힘, 최대 크기에 제한
    - resistance: compression resistance, 버티는 힘, 최소 크기에 제한
    - 허깅 기본값 250 레지스턴스 기본값 750으로 뷰를 줄이는것보다 늘리는게 쉬움
  - Intrinsic Size에 대해서 설명하시오.
    - 컨텐츠의 본질적 크기
    - 버튼, 레이블, 텍스트필드: 높이 넓이 크기 가짐
    - 뷰의 현재 컨텐츠 기반을 둠 (제약사항을 주지 않아도 크기 가짐)
  - 스토리보드를 이용했을때의 장단점을 설명하시오.
    - 장점: 시각화, 직관적, 앱 흐름 보기 편함, 미리보기, 오토레이아웃 디버깅 편함
    - 단점: 머지 컨플릭, 큰 용량의 앱에선 스토리보드 빌드 오래걸림, MVC 패턴 따름, 현업 불편, 재사용성, 번거로움, 스토리보드로 구현 못하는것도 있있음 (Border Radius)
  - Safearea에 대해서 설명하시오.
    - 컨텐츠가 가려지지 않는것이 보장되는 영역
    - 상태바, 네비게이션바, 홈 인디케이터 등 제외된 부분
  - Left Constraint 와 Leading Constraint 의 차이점을 설명하시오.
    - Left: 사용자가 바라보는 뷰의 왼쪽
    - Leading: 문자가 시작되는 방향으로 아랍권에서는 우측에서부터 RWL로 시작함으로 레프트와 다름
      

### 참고자료
  - https://github.com/JeaSungLEE/iOSInterviewquestions