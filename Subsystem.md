> **언리얼 엔진의 기능을 모듈화하여 필요할 때 쉽게 접근하고 관리할 수 있도록 만든 '서비스 센터'입니다.** 복잡한 시스템(예: 온라인 세션, 리플레이)의 API를 하나의 객체에 깔끔하게 정리하여, 개발자가 엔진의 내부 구조를 깊이 알지 못해도 편리하게 해당 기능들을 사용할 수 있도록 돕습니다.

### **1. 주요 역할 및 책임**
> `Subsystem`은 특정 기능에 대한 접근 창구를 단일화하고, 그 기능의 생명 주기를 엔진의 흐름에 자동으로 맞춰주는 역할을 합니다.
* **기능별 API 제공 (Feature-Specific API Provider):**
      특정 목적(예: 스팀 연동, UI 관리)에 관련된 기능들을 하나의 클래스에 모아 일관된 API를 제공합니다. 개발자는 더 이상 복잡한 관리자 클래스를 직접 찾아다닐 필요 없이, 원하는 `Subsystem`을 요청하기만 하면 됩니다.
* **자동 생명 주기 관리 (Automatic Lifecycle Management):**
      `Subsystem`은 자신이 속한 부모 객체(예: [[UGameInstance]], [[UWorldSubsystem]])와 생명 주기를 함께합니다. 부모 객체가 생성될 때 함께 초기화(`Initialize`)되고, 소멸될 때 함께 정리(`Deinitialize`)되므로, 개발자가 직접 생성과 소멸을 관리할 필요가 없습니다.
* **싱글턴과 유사한 접근성 (Singleton-like Accessibility):**
      대부분의 `Subsystem`은 특정 스코프 내에서 단 하나만 존재합니다. `GetSubsystem` 함수를 통해 어디서든 쉽게 해당 기능의 유일한 인스턴스에 접근할 수 있어, 싱글톤 패턴의 편리함을 안전하게 제공합니다.

### **2. Subsystem의 종류**
> `Subsystem`은 그 기능이 유효한 범위(스코프)에 따라 여러 종류로 나뉩니다.
* **[[UEngineSubsystem]]:**
      엔진이 실행되는 동안 항상 존재하는 최상위 `Subsystem`입니다. 엔진 자체의 핵심 기능이나 플러그인 관련 기능을 제공합니다.     
* **[[UGameInstanceSubsystem]]:**
	게임이 실행되고 종료될 때까지, 즉 [[UGameInstance]]의 생명 주기 동안 유지되는 `Subsystem`입니다. 게임 전체에 걸쳐 필요한 데이터나 기능(예: 저장/불러오기, 유저 설정)을 관리하기에 적합합니다.
* **[[UWorldSubsystem]]:**
      하나의 월드(레벨)가 로드되고 언로드될 때까지 유지되는 `Subsystem`입니다. 특정 레벨에서만 필요한 동적인 시스템(예: AI 관리자, 퀘스트 시스템)을 구현하는 데 사용됩니다.
* **[[UEnhancedInputLocalPlayerSubsystem]]:**
      로컬 플레이어가 존재할 때만 유지되는 `Subsystem`입니다. 플레이어 개개인에게 특화된 기능(예: 개인화된 UI 관리, 입력 처리)을 관리합니다.

### **3. 사용 방법**
> `Subsystem`을 사용하는 방법은 매우 간단하고 일관됩니다.
* **`Subsystem` 가져오기 (Getting a Subsystem):**
	아래와 같이, 원하는 `Subsystem`의 부모 객체(예: `GetGameInstance()`, `GetWorld()`)로부터 `GetSubsystem<T>()` 템플릿 함수를 호출하여 쉽게 접근할 수 있습니다.
```cpp
//게임 인스턴스 Subsystem 가져오기
UMyGameSubsystem* MyGameSubsystem = GetGameInstance()->GetSubsystem<UMyGameSubsystem>();

// 월드 Subsystem 가져오기
AMyWorldSubsystem* MyWorldSubsystem = GetWorld()->GetSubsystem<AMyWorldSubsystem>();
```