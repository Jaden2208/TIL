# 안드로이드 개발 직무 기술 면접을 위한 정리 노트

## :notebook_with_decorative_cover:Index

- [안드로이드 4대 컴포넌트](#안드로이드-4대-컴포넌트)

<br>

---

## 안드로이드 4대 컴포넌트

[Android Developer(Components)](https://developer.android.com/guide/components/fundamentals#Components)

- Activity [:arrow_heading_down:](#activity란)
- Service [:arrow_heading_down:](#service란)
- Broadcast Receiver [:arrow_heading_down:](#broadcast-receiver란)
- Content Provider [:arrow_heading_down:](#content-provider란)

### Activity란?

사용자가 어플리케이션과 상호작용하는 단일 화면으로, 즉, 사용자와 상호작용을 담당하는 인터페이스를 의미한다.

> 특징

- 2개 이상의 Activity를 동시에 Display할 수 없다.
- 1개 이상의 View 또는 ViewGroup을 포함한다.
- 어플리케이션에는 하나 이상의 Activity가 있어야 한다.
- Activity 내에 1개 이상의 Fragment를 추가할 수 있다.

[:arrow_heading_up:](#안드로이드-4대-컴포넌트)

### Service란?

여러 이유로 백그라운드에서 앱을 계속 실행하기 위한 컴포넌트이다.
예를 들어 사용자가 다른 앱에 있는 동안에 백그라운드에서 음악을 재생하거나, 사용자와 액티비티 간의 상호작용을 차단하지 않으면서 네트워크를 통해 데이터를 가져오거나 할 때 사용된다.

> 특징

- 보이지 않는 곳에서 작업을 처리한다고 해서 Worker Thread에서 동작하는 것이 아니라 Main Thread에서 동작하며,
서비스 내에서 별도의 Thread를 생성하여 작업을 처리한다.
- 네트워크와 연동이 가능하다.
- 앱이 종료돼도 이미 시작된 Service는 백그라운드에서 계속 동작한다.

[:arrow_heading_up:](#안드로이드-4대-컴포넌트)

### Broadcast Receiver란?

Android OS로 부터 발생하는 각종 이벤트와 정보를 받아와 핸들링하는 컴포넌트이다.
사용자 안드로이드 디바이스 시스템 부팅 시 앱 초기화, 네트워크 끊김 등 특수한 이벤트에 대한 처리나 배터리 부족 알림, 문자 수신과 같은 정보를 받아 처리를 해야할 때 동작한다.
즉, 안드로이드 OS에서 메신저앱 또는 문자 메시지가 오면 모든 앱에 "메시지가 왔다"라는 하나의 정보를 방송(Broadcast)한다.
이 메시지를 받기 위해 Broadcast Receiver를 구현하면 되며 해당 정보가 오면 특정 이벤트를 처리할 수 있다.

> 특징

- UI를 가지지 않는다.
- 안드로이드 디바이스의 특수한 상황에 대응하기 위해 사용된다.
- 특정한 상황을 제외하고는 브로드캐스트는 시스템에서 시작합니다.

[:arrow_heading_up:](#안드로이드-4대-컴포넌트)

### Content Provider란?

데이터를 관리하고 다른 어플리케이션의 데이터를 제공해주는 컴포넌트이다.
특정한 어플리케이션이 사용하고 있는 데이터베이스를 공유하기 위해 사용하며 어플리케이션 간의 데이터 공유를 위해 표준화된 인터페이스를 제공한다.

> 특징

- SQLite / Web / 파일 입출력 등을 통해서 데이터를 관리한다.
- 외부 어플리케이션이 현재 실행중인 어플리케이션 내에 있는 DB에 함부로 접근하지 못하게 할 수 있고, 공유하고 싶은 데이터만 공유할 수 있도록 해준다.
- 작은 데이터들은 Intent로 어플리케이션끼리 서로 데이터 공유가 가능하지만 ContentProvider는 음악, 사진파일 등과 같이 용량이 비교적 큰 데이터들을 공유하는데 적합하다.
- 데이터를 읽기 및 쓰기 에 대한 Permission이 있어야 어플리케이션 접근이 가능하다.
- 데이터베이스에서 흔히 사용되는 CRUD 원칙을 준수한다.

[:arrow_heading_up:](#안드로이드-4대-컴포넌트)
