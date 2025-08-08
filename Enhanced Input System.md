> `Enhanced Input System`은 언리얼 엔진의 현대적인 표준 입력 시스템입니다. 기존 레거시 입력 시스템의 한계를 극복하고, 더 유연하고, 확장 가능하며, 데이터 기반의 입력 처리를 제공하기 위해 설계되었습니다. 시스템의 핵심은 **입력의 '정의(Data)', '처리(Processing)', '실행(Action)'을 명확하게 분리**하는 것입니다.

### **1. 데이터 계층: 입력의 정의 (Data Assets)**
> 코드 변경 없이 에디터에서 입력 방식을 정의하고 수정할 수 있는 데이터 에셋들입니다.
* [[UInputAction]] (IA):
	'점프', '이동', '공격'과 같이 **추상적인 입력 행위 자체**를 정의하는 데이터 에셋입니다. 이 액션이 반환할 값의 타입(`bool`, `float`, `Vector2D` 등)을 지정합니다.
* [[UInputMappingContext]] (IMC):
	**'규칙의 집합'** 또는 '키 매핑 테이블'입니다. 특정 하드웨어 입력(예: 키보드 `W` 키, 게임패드 왼쪽 스틱)을 어떤 [[UInputAction|IA]]에 연결할지 매핑 정보를 담고 있습니다.
* [[UInputModifier]]: 
	입력의 원시 값을 가공하는 역할을 합니다. (예: `Swizzle`, `Dead Zone`, `Negate`) [[UInputMappingContext|IMC]] 내에서 특정 매핑에 적용할 수 있습니다.
* [[UInputTrigger]]:
	입력이 언제 '트리거' 상태가 될지를 결정하는 조건입니다. (예: `Pressed`, `Released`, `Held`, `Tap`) `IMC` 내에서 특정 매핑에 적용할 수 있습니다.

### **2. 처리 계층: 입력의 해석 및 발행 (Processing & Publishing)**
> 하드웨어 입력을 수신하여, 데이터 계층의 규칙에 따라 해석하고, 게임플레이 시스템이 사용할 수 있는 이벤트로 만들어 발행합니다.
* [[UEnhancedInputLocalPlayerSubsystem]]:
	**입력 처리의 중앙 허브**입니다. [[ULocalPlayer]]에 종속되어 있으며, 활성화된 [[UInputMappingContext|IMC]] 목록을 관리하고, 하드웨어 입력을 [[UInputMappingContext|IMC]] 규칙에 따라 해석하여 최종적으로 처리된 [[UInputAction|IA]]의 상태 변화를 **발행(Publish)**합니다.

### **3. 실행 계층: 이벤트의 구독 및 실행 (Subscription & Execution)**
> 발행된 이벤트를 받아 실제 게임플레이 로직을 실행합니다.
* [[UEnhancedInputComponent]]:
	[[APlayerController]]가 소유하며, [[APawn]](캐릭터)에 의해 설정되는 컴포넌트입니다. 발행된 액션 이벤트를 **구독(Subscribe)**하는 역할을 합니다.
* `BindAction(const UInputAction* Action, ETriggerEvent TriggerEvent, ...)`:
	**구독 신청**을 하는 핵심 함수입니다. SetupPlayerInputComponent 내에서 호출되며, "어떤 [[UInputAction|Action]]의 어떤 `TriggerEvent`가 발생하면, 어떤 객체의 어떤 함수를 호출하라"고 [[UEnhancedInputComponent]]에 등록합니다.
* 액션 이벤트 함수 (예: `Move(const FInputActionValue& Value)`):
	`BindAction`에 등록된 실제 멤버 함수입니다. 이 함수는 최종적으로 [[UEnhancedInputComponent]]에 의해 호출되며, [[FInputActionValue]] 구조체를 통해 처리된 입력 값을 받아 실제 게임플레이 로직을 수행합니다.