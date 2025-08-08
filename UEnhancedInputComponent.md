> **[[UInputAction]]이라는 '추상적인 행동'과, 그 행동이 발동되었을 때 실제로 실행될 'C++ 함수'를 연결하는 '전기 배선판'입니다.** 플레이어의 입력이 최종적으로 어떤 코드를 실행할지를 결정하는 바인딩(Binding) 로직을 담당하는 핵심 [[UActorComponent]]입니다.

### **1. 주요 역할 및 책임**
> `UEnhancedInputComponent`는 [[APawn]]이나 [[APlayerController]]에 추가되어, [[Enhanced Input System]]의 [[Event|이벤트]]와 실제 게임플레이 로직을 연결하는 다리 역할을 합니다.
* **액션과 함수의 바인딩 (Binding Actions to Functions):**
      `BindAction()` 함수를 통해, 특정 [[UInputAction]] 에셋이 특정 [[ETriggerEvent]](예: `Triggered`, `Started`, `Completed`) 상태가 되었을 때, 어떤 객체의 어떤 함수를 호출할지를 명확하게 지정합니다.
* **입력 처리의 종착점 (Destination of Input Processing):**
      [[UEnhancedInputLocalPlayerSubsystem]]이 플레이어의 입력을 받아 [[UInputMappingContext]]와 [[UInputTrigger]]를 거쳐 최종적으로 [[UInputAction]]을 발동시키면, 그 결과가 이 컴포넌트에 도달하여 바인딩된 함수를 실행시킵니다.
* **입력 값 전달 (Passing Input Values):**
      바인딩된 함수에 [[UInputAction]]이 생성한 값(예: `bool`, `float`, `FVector2D`)을 파라미터로 전달합니다. 이를 통해 개발자는 이동 방향, 마우스 움직임 등의 값을 함수 내에서 직접 사용할 수 있습니다.

### **2. 핵심 함수**
> `SetupPlayerInputComponent` 함수 내에서 입력을 바인딩하기 위해 사용하는 가장 중요한 함수입니다.
* `BindAction(const UInputAction* Action, ETriggerEvent TriggerEvent, UObject* Object, FName FunctionName, ...)`:
	지정된 [[UInputAction]]과 C++ 함수를 바인딩하는 핵심 함수입니다.
	* `Action`:
	바인딩할 [[UInputAction]] 에셋을 지정합니다.
	* `TriggerEvent`:
	함수가 호출될 [[ETriggerEvent]]의 종류를 지정합니다. (`Triggered`가 가장 일반적으로 사용됩니다.)
	* `Object`: 
	함수를 소유하고 있는 객체(`this`)를 지정합니다.
	* `FunctionName`: 
	호출될 함수의 이름을 지정합니다.

### **3. 사용 예시**
> [[APawn]]이나 [[ACharacter]]의 `SetupPlayerInputComponent` 함수 내에서 `UEnhancedInputComponent`를 사용하는 전형적인 코드 패턴입니다.

```cpp
void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	// PlayerInputComponent를 UEnhancedInputComponent로 캐스팅합니다.
	if (UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent))
	{
		// JumpAction이 Triggered 상태일 때, ACharacter 클래스에 내장된 Jump 함수를 호출하도록 바인딩합니다.
		EnhancedInputComponent->BindAction(JumpAction, ETriggerEvent::Triggered, this, &ACharacter::Jump);
		// MoveAction이 Triggered 상태일 때, 이 클래스의 Move 함수를 호출하도록 바인딩합니다.
		// Move 함수는 FInputActionValue를 파라미터로 받도록 작성되어야 합니다.
		EnhancedInputComponent->BindAction(MoveAction, ETriggerEvent::Triggered, this, &AMyCharacter::Move);
	}
}

// MoveAction에 바인딩될 함수의 예시
void AMyCharacter::Move(const FInputActionValue& Value)
{
	// FInputActionValue로부터 FVector2D 값을 가져옵니다.
	FVector2D MovementVector = Value.Get<FVector2D>();
	// ... 이동 로직 구현 ...
}
```