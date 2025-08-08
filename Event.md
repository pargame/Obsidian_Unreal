> **특정 사건이 발생했음을 다른 객체에게 알리고, 그 사건에 관심 있는 객체들이 자신만의 반응(함수)을 실행하도록 하는 '신호 및 응답' 시스템입니다.** C++의 함수 포인터를 더욱 안전하고 유연하게 확장한 '델리게이트(Delegate)'라는 강력한 기능을 기반으로 하며, 언리얼 엔진에서 객체 간의 결합도를 낮추는 핵심적인 역할을 합니다.

### **1. 주요 역할 및 책임**
> 이벤트/델리게이트 시스템은 코드의 유연성과 재사용성을 극대화합니다.
* **결합도 감소 (Decoupling):**
    이벤트를 보내는 객체(Broadcaster)는 누가 이벤트를 받는지(Listener) 전혀 알 필요가 없습니다. 그저 "내 체력이 0이 되었다!"라고 소리치기만 하면, '캐릭터 사망 처리기', 'UI 업데이트 처리기', '점수 계산 처리기' 등 관심 있는 객체들이 각자 알아서 반응합니다. 이를 통해 각 시스템을 독립적으로 개발하고 수정할 수 있습니다.
* **다대다 통신 (Many-to-Many Communication):**
    하나의 이벤트에 여러 개의 함수를 연결(Bind)할 수 있으며(멀티캐스트 델리게이트), 반대로 여러 개의 이벤트가 하나의 함수를 호출할 수도 있습니다.
* **블루프린트 연동 (Blueprint Integration):**
    C++ 코드에서 선언한 델리게이트를 `BlueprintAssignable`로 지정하면, 블루프린트의 이벤트 그래프에서 해당 이벤트를 노드로 받아 커스텀 로직을 연결할 수 있습니다.

### **2. 델리게이트(Delegate)의 종류**
> 델리게이트는 목적에 따라 다양한 종류가 있으며, 선언 매크로를 통해 만듭니다.
* **`DECLARE_DELEGATE`:**
    가장 기본적인 델리게이트입니다. 단 하나의 함수만 바인딩할 수 있습니다.
* **`DECLARE_MULTICAST_DELEGATE`:**
    여러 개의 함수를 바인딩할 수 있는 멀티캐스트 델리게이트입니다. 가장 널리 사용되는 형태입니다. (예: `OnClicked` 이벤트)
* **`DECLARE_DYNAMIC_DELEGATE`:**
    블루프린트에서 접근하고, 직렬화(저장)가 가능한 동적 델리게이트입니다. `BlueprintAssignable` 이벤트는 반드시 동적 델리게이트여야 합니다.
* **`DECLARE_DYNAMIC_MULTICAST_DELEGATE`:**
    동적 멀티캐스트 델리게이트입니다. 블루프린트에 노출되는 이벤트의 대부분이 이 형태로 선언됩니다.

### **3. 사용 흐름**
1.  **선언 (Declaration):**
    `.h` 헤더 파일에서 매크로를 사용하여 델리게이트 타입을 선언합니다.
    `DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnHealthChanged);`
2.  **인스턴스 생성 (Instantiation):**
    클래스 내부에 선언한 델리게이트 타입의 인스턴스를 `UPROPERTY(BlueprintAssignable)`과 함께 선언합니다.
    `UPROPERTY(BlueprintAssignable) FOnHealthChanged OnHealthChanged;`
3.  **바인딩 (Binding):**
    리스너(Listener) 객체에서, 이벤트에 반응할 자신의 함수를 추가합니다.
    `MyHealthComponent->OnHealthChanged.AddDynamic(this, &AMyHUD::UpdateHealthBar);`
4.  **브로드캐스트 (Broadcast):**
    브로드캐스터(Broadcaster) 객체에서, 특정 사건이 발생했을 때 `Broadcast()` 함수를 호출하여 이벤트를 모든 리스너에게 알립니다.
    `OnHealthChanged.Broadcast();`

### **4. 주요 예시**
* **`UPrimitiveComponent`의 충돌 이벤트:**
    `OnComponentBeginOverlap`은 `DECLARE_DYNAMIC_MULTICAST_DELEGATE...`로 선언된 델리게이트이며, 액터가 영역에 들어왔을 때 `Broadcast()`를 호출합니다.
* **UI 버튼의 `OnClicked` 이벤트:**
    버튼이 클릭되었을 때 `Broadcast()`를 호출하여, 바인딩된 함수들이 실행되도록 합니다.
* **`FTimerManager`의 타이머:**
    `SetTimer` 함수는 지정된 시간이 지난 후 바인딩된 함수를 호출해주는 델리게이트를 인자로 받습니다.
