### LaunchedEffect에 key를 왜 넣나요?

- LaunchedEffect는 리컴포지션마다 호출되는 것이 아니라 UI와 별개로 동작하며 코루틴을 이용한 SideEffect를 위해서 만들어진 Composable
- 이 때 LaunchedEffect에 넣어준 key 값이 변경될 때 마다 LauncehdEffect의 블록이 새롭게 호출되는 방식임
- 즉 특정한 값의 변경을 트리거로 SideEffect를 만들고싶다면 LaunchedEffect에 해당 값을 key로 넣어주면 된다.

<br><br><br>

### Recomposition 횟수를 알 수 있는 방법과 이를 조절하는 방식에는 어떤 것이 있나요?

- LayoutInspetor를 이용해서 리컴포지션 횟수 측정
- 혹은 SideEffect를 이용해서 별도로 stateful하게 측정할 수도 있을 것 같긴한데 굳이 ?!

<br><br><br>

### Compose에서 Stable, Immutable Annotation를 추가하는 이유를 알고 있나요?

- 해당 어노테이션을 사용하지 않을 경우, UI를 그리는 모듈의 외부 모듈과 내부적으로 가변적인 객체가 있을경우 Non-Stable하다고 판단하여 항상 Restartable로 판단하게 됨
- 내부적으로 가변 객체가 있더라도 Copy() 연산을 통해서 값이 바뀐다거나 바뀔일이 없다면 위 어노테이션을 사용함으로써 해당 Composable을 Skippable 하게 만들수 있음.

https://blog.naver.com/tgyuu_/223720324026

<br><br><br>

### CompositionLocal에 대해서 설명해주세요. StaticCompositionLocal과 다른 점은 무엇인가요?

- PropsDrilling 방식으로 데이터를 전달하는 것이 아닌, DI를 할 때 IoC 컨테이너를 사용하는 것처럼 CompositionLocal을 사용하면 하위 컴포저블에서 원할 때 언제던지 해당 값을 가져와서 사용할 수 있음
- StaticCompositionLocal는 가능하면 데이터가 자주 변하지 않는 곳에 사용해야하는데, 왜냐하면 StaticCompositionLocal로 제공하는 값이 변경될 경우 하위 컴포저블 중 한 개라도 해당 값을 참조하고 있다면 스코프 아래에 있는 모든 컴포저블이 리컴포지션 됨
- 반면 CompsositionLocal은 해당 값을 참조하고 있는 Composable **만** 리컴포지션 되는데, 그렇기 때문에 값이 변할 가능성이 있다면 CompositionLocal을 쓰는 것이 바람직함.
- 그럼 왜 StaticCompositionLocal을 사용하느냐고 반문할 수 있는데, Immutable한 객체를 제공하면 컴포즈 컴파일러 내부적으로 더 공격적인 최적화를 할 수 있으며 Recomposition Tracking을 할 필요가 없기 때문임.
- 또한 테마나 언어 설정의 경우는 하위 모든 Composable이 변경되어야만 하므로 staticCompositionLocal을 사용했을 때 더 원하는 결과가 나올 수 있음.

<br><br><br>

### Compose의 렌더링 과정에 대해서 설명해주세요.

- Mesaurable -> Placeable -> LaidOut
- Measureable 단계에서는 해당 컴포저블의 크기를 측정하며 Placeable에서는 해당 컴포저블이 배치되는 위치를 측정하며 마지막 LaidOut 단계에서 UI를 그려냄
- Measure 단계에서는 Bottom-Up 방식으로 컴포저블을 측저하고, LaidOut 단계에서는 Top-Down 방식으로 UI를 그려나감

<br><br><br>

### 호이스팅에 대해서 설명해주세요.

- Composable은 파라미터를 바탕으로 Compossable이 Skippable일 때 파라미터가 동일한 값일 경우 Skip할 수 있는데, 호이스팅 없이 모두 Root 수준으로 데이터를 올려서 Props Drilling 하게 되면 불필요한 리컴포지션이 많아질 수도 있고 보일러 플레이트 코드가 많이 생길 수 있음
- 또한 외부에서 값을 주입하는 방식으로 UI를 그리게되면 해당 Composable은 stateless하게 그릴 수 있으므로 Preview 및 테스트 코드를 작성하는데도 효율적임.
- 그렇기 때문에 공식문서에서 추천하는 방식은 여러 Composable에서 공유하거나 값을 핸들링해야 할 경우 최소한으로 올려서 상태를 호이스팅하는 방식을 추천함.

<br><br><br>

### Compose의 Composable은 stateless, stateful을 어떤 기준으로 구분하고 적용하셨는 지 설명해주세요.

- State가 빈번하게 변경되지 않을경우 stateless하게 관리하려고 노력하였으며, 만약 내부적으로 값의 변경이 잦거나 props drilling 해야하는 Compose의 트리 계층이 깊다면 내부적으로 stateful하게 값을 hold하고 있는 다음 특정 Intent에서 holding한 값을 ViewModel로 전송하려고 하였음. 
- 이 방식은 보일러 플레이트 코드도 많이 줄일 수 있지만 팀 내에서 stateless와 stateful을 규칙 과 약속 없이 사용하게 되면 디버깅이 힘들어질 수도 있고 Preview나 UI테스트를 작성하는 데 어려워 질 수 있으므로 트레이드 오프를 잘 고려하여 작성해야 함.