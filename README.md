## Ruby on Rails 기반 애플리케이션 아키텍처 및 컨벤션 표준 프롬프트 문서

이 문서는 Ruby on Rails(RoR) 애플리케이션 개발에 필요한 **핵심 아키텍처 구조, 컨벤션, 그리고 품질 기준**을 표준화하여, 새로운 팀원 온보딩, 코드 리뷰, 또는 외부 파트너에게 **개발 프롬프트(Prompt)로 제공**하기 위한 지침입니다.

---

### 1. 📂 아키텍처 구조 (Architecture Structure)

RoR의 기본 MVC 구조를 따르되, 대규모 애플리케이션의 복잡성을 관리하기 위해 서비스 계층(Service Layer)과 폼 객체(Form Object) 패턴을 도입합니다.

#### 1.1. 핵심 계층 및 역할

| 디렉토리 | RoR 기본 역할 | 확장 역할 및 패턴 |
| :--- | :--- | :--- |
| `app/models` | 데이터 및 비즈니스 로직 정의 | **유효성 검증(Validations)**, **콜백(Callbacks)**, 관계(Associations)만 포함. |
| `app/controllers` | HTTP 요청 처리 및 응답 반환 | 데이터 처리 로직은 **Service Layer**로 위임. 파라미터 유효성 검증 및 인증/인가 처리. RESTful 원칙 준수. |
| `app/services` | **핵심 비즈니스 로직** | 복잡한 트랜잭션, 외부 API 호출, 다중 모델 조작 등. `ServiceObjectName.call(params)` 형태로 호출. |
| `app/forms` | **입력 데이터 처리** | 다중 모델에 걸친 데이터 처리(Nested Attributes), 복잡한 폼 입력 유효성 검증 및 정규화. |
| `app/presenters` (선택) | **뷰 로직** | 뷰에 전달되는 데이터의 형식 지정 및 표시 로직을 담당. (헬퍼 남용 방지) |
| `app/views` | HTML/JSON/응답 템플릿 | 가능한 한 로직 없이 데이터 표시만 전담. **Partial**을 적극 활용하여 재사용성 확보. |

---

### 2. 📝 코드 컨벤션 및 스타일 가이드 (Code Conventions & Style Guide)

모든 코드는 Ruby/Rails 커뮤니티의 모범 사례와 **RuboCop**을 기준으로 합니다.

#### 2.1. Ruby 스타일

| 항목 | 표준 컨벤션 | 예시 |
| :--- | :--- | :--- |
| **메서드 정의** | `def` 사용 시 괄호 생략 (인수가 없을 때). | `def user_count` |
| **조건문** | 짧은 조건문은 `if`를 문장 뒤에 배치. | `return unless admin?` |
| **문자열** | 내삽(Interpolation)이 필요 없는 경우 **작은따옴표(' ')** 사용. | `'Hello World'` |
| **블록** | 한 줄 블록은 `{}`, 여러 줄 블록은 `do/end` 사용. | `list.each { |i| ... }` vs. `list.each do |i| ... end` |

#### 2.2. Rails 컨벤션

* **RESTful 라우팅:** `resources :model_name`을 기본으로 사용하고, 필요한 경우에만 `member`나 `collection`을 활용합니다.
* **파일 이름:** 클래스 이름과 파일 이름은 **스네이크 케이스**를 따릅니다. (예: `UserSerializer` -> `user_serializer.rb`)
* **Migration:** 컬럼 추가/변경 시 명확한 이름을 사용합니다. (예: `AddIndexToUsersEmail`, `ChangeStatusToEnumInOrders`)
* **Controller 액션:** 기본 7가지 액션(`index`, `show`, `new`, `create`, `edit`, `update`, `destroy`) 외에는 `member` 또는 `collection` 블록 내에서 명확한 이름을 부여합니다.

---

### 3. 🛡️ 품질 및 테스트 (Quality & Testing)

모든 코드 변경사항은 테스트를 동반해야 하며, CI/CD 파이프라인을 통과해야 배포될 수 있습니다.

#### 3.1. 테스트 전략

| 테스트 유형 | 담당 범위 | 권장 도구 | 필수 요건 |
| :--- | :--- | :--- | :--- |
| **단위 테스트 (Unit)** | 모델의 유효성 검증, 서비스 객체의 핵심 로직. | RSpec | 최소 80% 커버리지 목표. |
| **통합 테스트 (Integration)** | 컨트롤러 액션, 폼 객체 처리, 전체 워크플로우. | RSpec Request Specs | 컨트롤러당 최소 하나의 성공/실패 시나리오 테스트. |
| **피쳐 테스트 (Feature)** | 사용자 인터페이스 관점에서의 기능 테스트. | RSpec + Capybara | 핵심 사용자 여정(Critical User Journeys) 커버. |

#### 3.2. 코드 품질 자동화

1.  **RuboCop:** 모든 코드에 적용하여 스타일 및 잠재적 버그를 검사합니다.
2.  **Rails Best Practices:** 아키텍처 및 디자인 패턴 위반을 검사합니다.
3.  **Security Advisories:** `bundler-audit` 또는 유사 도구를 사용하여 취약점이 있는 Gem 사용을 방지합니다.

---

### 4. 🌐 데이터베이스 접근 및 최적화

#### 4.1. N+1 문제 방지

* 데이터를 조회할 때는 항상 **`includes`** 또는 **`eager_load`**를 사용하여 N+1 쿼리 문제를 방지합니다.
    * *예시:* `User.includes(:posts)`
* Scope는 명확한 이름으로 정의하고, 복잡한 로직은 **Arel**을 활용하여 작성합니다.

#### 4.2. 트랜잭션 관리

* 여러 개의 데이터베이스 조작이 필요한 비즈니스 로직은 **Service Layer** 내에서 **`ActiveRecord::Base.transaction do ... end`** 블록을 사용하여 원자성(Atomicity)을 보장해야 합니다.

---

### 🚀 개발 프롬프트 핵심 요약

**[모든 개발자/파트너에게 요구되는 사항]**

1.  **MVC+S+F 아키텍처**를 엄격히 준수할 것. (복잡한 로직은 Service, 다중 입력 처리는 Form)
2.  **RESTful 원칙**에 따라 Controller를 구성할 것.
3.  **RuboCop**을 통과하고, **N+1 문제를 방지**할 것.
4.  새로 작성된 기능에는 반드시 **RSpec 테스트 코드**를 포함하고, CI를 통과하도록 할 것.



## MVP 방법론

**목표:** **Ruby on Rails(RoR)의 견고함**과 **MVP (Minimum Viable Product) 방법론의 신속한 시장 검증 원칙**을 결합하여, 불필요한 기능 개발을 최소화하고 핵심 가치 제공에 집중하는 개발 표준을 확립합니다.

---

### 1. 🎯 MVP 정의 및 개발 원칙 (Methodology Integration)

#### 1.1. MVP 핵심 정의

* **최소 기능:** 사용자가 문제를 해결하는 데 **반드시 필요한** 단 하나의 핵심 기능 셋을 정의합니다. (The "One Thing" Principle)
* **측정 가능성:** 개발된 기능은 가설 검증을 위해 반드시 측정 가능해야 합니다. (예: 전환율, 사용자 유지율 등)
* **신속한 피드백:** 릴리스 주기를 극도로 단축하여 시장으로부터 빠른 피드백을 확보하고 다음 이터레이션을 결정합니다.

#### 1.2. RoR 기반 MVP 개발 원칙

1.  **Convention Over Configuration (CoC) 극대화:** RoR의 기본 컨벤션(Scaffolding, RESTful)을 최대한 활용하여 초기 개발 시간을 절약합니다. 커스텀 코드(Service Layer 등)는 **핵심 비즈니스 로직**에만 한정합니다.
2.  **데이터 모델 우선:** MVP 가설을 검증하는 데 필요한 **핵심 데이터 모델(Model)**만 먼저 정의합니다. 향후 확장을 고려한 불필요한 복잡한 모델이나 관계는 MVP 범위에서 제외합니다.
3.  **Hotwire (Turbo/Stimulus) 선호:** 프론트엔드 복잡성을 낮추고, RoR의 렌더링 파이프라인을 활용하여 SPA(Single Page Application) 수준의 반응성을 저비용으로 구현합니다. (외부 복잡한 JS 프레임워크 사용 최소화)

---

### 2. 📂 아키텍처 구조 (Architecture Structure) - MVP 조정

기존 아키텍처는 유지하되, MVP 단계에서는 특정 디렉토리에 대한 접근을 제한하거나 단순화합니다.

| 디렉토리 | MVP 단계에서의 역할 조정 | MVP 원칙 적용 |
| :--- | :--- | :--- |
| `app/models` | **핵심 비즈니스 로직만 포함.** 복잡한 관계(Poly-morphic, Deep Nested)는 MVP 이후로 연기. | **Simple Schema.** |
| `app/controllers` | **RESTful 표준 철저 준수.** `respond_to :html` 또는 `respond_to :json` 중 하나에만 집중. | **Minimizing Endpoints.** |
| `app/services` | **MVP의 핵심 트랜잭션** (예: 가입, 첫 번째 구매) 로직만 정의. 불필요한 자동화/연동 로직 제외. | **Focused Logic.** |
| `app/views` | **UX/UI는 기능 검증에 충분한 수준만.** 디자인 시스템/고도화된 CSS는 MVP 이후로 연기. Hotwire (Turbo Frames/Streams) 적극 활용. | **Function over Form.** |

---

### 3. 📝 코드 컨벤션 및 스타일 가이드 - 신속성 강조

RoR 기본 컨벤션(`2. Code Conventions` 참조)은 따르되, **개발 속도**를 저해하는 지나친 추상화는 MVP 단계에서는 지양합니다.

| 항목 | MVP 단계 컨벤션 | 목표 |
| :--- | :--- | :--- |
| **재사용성** | **필요할 때만 리팩토링.** 당장 재사용되지 않는 코드를 미리 추상화하지 않습니다. (YAGNI 원칙 준수) | 신속한 기능 완료 |
| **헬퍼 (Helper)** | **단순한 뷰 로직**에 한해 헬퍼를 일시적으로 허용. Presenter/Decorator 패턴 도입은 MVP 이후 고려. | 뷰 로직 단순화 |
| **Gem 사용** | **핵심 기능에 필수적인 Gem**만 사용하고, 의존성 관리(Gemfile)를 간결하게 유지합니다. | 의존성 경량화 |

---

### 4. 🛡️ 품질 및 테스트 - 핵심 기능 검증

테스트는 MVP의 **핵심 기능이 작동하는지** 검증하는 데 초점을 맞춥니다.

#### 4.1. 테스트 전략 (MVP Focusing)

* **단위 테스트 (Model/Service):** **100%** 커버리지를 목표로 합니다. (MVP의 핵심 비즈니스 로직이 안전함을 보장)
* **통합 테스트 (Controller/Request):** 사용자에게 **가치를 제공하는 핵심 여정(Critical Path)**에 대해서만 테스트를 작성합니다. 엣지 케이스 및 오류 처리에 대한 상세 테스트는 MVP 이후로 연기합니다.
* **Feature/UI 테스트:** **최소화.** 수동 QA를 통해 핵심 사용자 여정만 검증하고, 복잡한 자동화 테스트는 MVP 이후 기능 안정화 단계에서 도입합니다.

---

### 5. 🛠️ MVP 이터레이션 관리 및 확장

#### 5.1. MVP 배포 및 검증

* **배포 (Deployment):** Docker와 **Kamal**을 사용하여 빠르고 안정적인 단일 서버 배포 환경을 구축합니다. (배포 복잡성 최소화)
* **측정:** **ActiveRecord Callback** 또는 **Controller Hook**을 활용하여 핵심 액션(예: `after_create :track_user_signup`)에 대한 데이터 추적(Analytics)을 통합합니다.

#### 5.2. 확장 및 리팩토링

* MVP가 성공적으로 검증되면, **기술 부채(Technical Debt)**를 해결하는 리팩토링 스프린트를 반드시 진행합니다.
* 이후 기능 확장 시, `app/services`, `app/forms` 등 계층화된 아키텍처를 점진적으로 강화하고 적용하여 확장성을 확보합니다.

---

### 🚀 최종 개발 프롬프트 요약 (MVP Focus)

**[모든 개발자/파트너에게 요구되는 사항]**

1.  **MVP의 핵심 기능 목록 외의 모든 코드 추가는 금지.** (Do the simplest thing that could possibly work)
2.  **Hotwire (Turbo/Stimulus) 기반으로 프론트엔드를 구축**하여 렌더링 비용을 최소화할 것.
3.  **Service Layer는 핵심 트랜잭션**에만 사용하고, **Model/Service 단위 테스트 100%**를 확보할 것.
4.  **불필요한 Gem 및 복잡한 DB 관계는 MVP 이후로 연기**할 것.
5.  모든 기능은 **빠른 피드백 측정**을 위해 간단한 분석 로깅을 포함해야 함.
