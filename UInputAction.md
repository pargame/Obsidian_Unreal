> **"점프", "발사", "상호작용"과 같이, 플레이어가 수행할 수 있는 추상적인 '행동' 그 자체를 정의하는 데이터 에셋입니다.** 이 행동이 구체적으로 어떤 키에 의해, 어떤 조건에서 발동될지는 [[UInputMappingContext|IMC]]에서 결정하므로, `UInputAction (IA)`은 순수한 '행동의 의미'에만 집중합니다.

### **1. 주요 역할 및 책임**
> `IA`는 입력을 물리적인 키로부터 분리하여, 코드의 가독성과 재사용성을 높이는 핵심적인 역할을 합니다.

* **입력의 추상화 (Abstraction of Input):**
      개발자는 코드에서 `W` 키가 눌렸는지를 확인하는 대신, `IA_MoveForward`라는 '행동'이 발생했는지를 확인하게 됩니다. 이를 통해 키 바인딩이 변경되더라도 코드를 수정할 필요가 없어집니다.
* **값 타입 정의 (Value Type Definition):**
      이 행동이 전달할 값의 종류를 결정합니다. 예를 들어, '이동' 액션은 `Axis2D` 값을, '점프' 액션은 `Digital` 값을, '줌인/줌아웃' 액션은 `Axis1D` 값을 가질 수 있습니다.
* **[[UInputTrigger|트리거]]와 [[UInputModifier|모디파이어]]의 컨테이너 (Container for Triggers and Modifiers):**
      `IA` 자체에도 기본적인 [[UInputTrigger|트리거]]와 [[UInputModifier|모디파이어]]를 추가할 수 있습니다. 여기에 추가된 규칙은 이 액션이 어떤 [[UInputMappingContext|IMC]]에서 사용되든 항상 적용됩니다. (보통은 컨텍스트 레벨에서 설정하는 것이 더 유연합니다.)
	  
### **2. 핵심 속성**
> `IA` 에셋 에디터에서 설정하는 가장 중요한 속성입니다.

* **Value Type:** 이 `IA`가 생성하는 값의 데이터 타입을 결정합니다.
    * **`Digital (bool)`:** 눌렸는지 안 눌렸는지를 나타내는 `true`/`false` 값을 가집니다. (예: 점프, 발사)
    * **`Axis1D (float)`:** `-1.0`에서 `1.0` 사이의 단일 축 값을 가집니다. (예: 마우스 휠을 이용한 줌, 전진/후진)
    * **`Axis2D (FVector2D)`:** `X`, `Y` 두 축으로 구성된 2D 벡터 값을 가집니다. (예: WASD 이동, 마우스/조이스틱 조준)
    * **`Axis3D (FVector)`:** `X`, `Y`, `Z` 세 축으로 구성된 3D 벡터 값을 가집니다. (6축 모션 컨트롤러 등에 사용)
* **`Triggers`:** 이 `IA`에 항상 적용될 기본 [[UInputTrigger|트리거]] 목록입니다.
* **`Modifiers`:** 이 `IA`에 항상 적용될 기본 [[UInputModifier|모디파이어]] 목록입니다.

### **3. 사용 흐름**
> `IA`는코드 내에서 특정 행동에 대한 콜백 함수를 바인딩하는 데 사용됩니다.
1. **생성 및 정의:**
      콘텐츠 브라우저에서 `IA` 에셋을 생성하고, `Value Type` 등 주요 속성을 설정합니다.
2. **컨텍스트에 매핑:**
      [[UInputMappingContext|IMC]]를 열어, 방금 만든 `IA`를 특정 키를 연결하고, [[UInputTrigger]]와 [[UInputModifier]]를 설정합니다.
3. **코드에 바인딩:**
      [[APlayerController]]나 [[APawn]]의 `SetupPlayerInputComponent` 함수 내에서, [[UEnhancedInputComponent]]의 `BindAction()` 함수를 사용하여 `UInputAction`과 실제 실행될 함수를 연결합니다.
```cpp
// EnhancedInputComponent에 IA_Jump 액션을 바인딩하는 예시
if (UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent))
{
	//IA_Jump 액션이 "Triggered" 상태일 때, this 객체의 Jump 함수를 호출하도록 바인딩합니다.
	EnhancedInputComponent->BindAction(JumpAction, ETriggerEvent::Triggered, this, &ACharacter::Jump);
}
```