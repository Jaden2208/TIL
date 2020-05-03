# 안드로이드 개발 직무 기술 면접을 위한 정리 노트

## :notebook_with_decorative_cover:Index

- [안드로이드 4대 컴포넌트](#안드로이드-4대-컴포넌트)  
- [안드로이드의 실행환경(구조)](#안드로이드의-실행환경구조)  
- [안드로이드 프로젝트 구성요소](#안드로이드-프로젝트-구성요소)  
- [안드로이드에서 다국어 지원을 위해 해야할 작업](#안드로이드에서-다국어-지원을-위해-해야할-작업)  
- [Manifest 파일](#manifest-파일)  
- [안드로이드 액티비티의 생명주기](#안드로이드-액티비티의-생명주기)  
- [MVC 아키텍쳐 패턴](#mvc-아키텍쳐-패턴)  
- [MVP 아키텍쳐 패턴](#mvp-아키텍쳐-패턴)  
- [MVVM 아키텍쳐 패턴](#mvvm-아키텍쳐-패턴)


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

---

## 안드로이드의 실행환경(구조)

![android_system_architecture](/images/android_system_architecture.png)
> By Smieh - Anatomy Physiology of an Android, CC BY-SA 3.0, https://commons.wikimedia.org/w/index.php?curid=20067152

안드로이드는 크게 4가지 실행환경으로 구성되어있다. 가장 하단부터 **리눅스 커널**, **라이브러리**, **어플리케이션 프레임워크**, **어플리케이션** 순서이다. **리눅스 커널** 은 OS로 안드로이드 스마트폰의 다양한 하드웨어(화면, 카메라, 블루투스, GPS, 메모리 등)를 관리한다. **라이브러리** 는 안드로이드에 있는 다양한 기능(UI 처리, 미디어 프레임워크, 데이터베이스 등)을 소프트웨어적으로 구현해 놓은 환경 뿐만 아니라 안드로이드 앱을 구동해주는 dalvik 가상머신과 코어 라이브러리까지 포함하는 영역이다. **어플리케이션 프레임워크** 는 사용자의 입력(액티비티, 윈도우, 컨텐츠, 뷰, 노티피케이션 등) 또는 특정한 이벤트에 따라 출력을 담당하는 환경을 말한다. 마지막으로 **어플리케이션** 은 실제로 동작하는 앱이 설치되는 환경을 의미한다.

[:arrow_heading_up:](#notebook_with_decorative_coverindex)

---

## 안드로이드 프로젝트 구성요소

- **libs**: 프로젝트에서 사용하는 다양한 라이브러리 소스가 저장되는 공간  
- **androidTest**: 앱의 일부 코드를 테스트하기 위한 소스를 저장하는 공간  
- **java**: 자바 또는 코틀린 코드를 저장하는 공간. 표준 자바와 동일하게 패키지를 이용한 하위 디렉토리 생성 방식을 사용한다.  
- **res**: 리소스를 저장하는 공간  
- **AndroidManifest.xml**: 앱에 대한 전체적인 정보를 담고있는 파일이다. 앱의 구성요소와 실행 권한 정보가 정의되어있다.  
- **project > build.gradle**: 개발자가 직접 작성한 Gradle Build Script 파일  
- **gradle > build.gradle**: 앱에 대한 컴파일 버전정보, 의존성 프로젝트에 대한 정의가 되어있는 파일

[:arrow_heading_up:](#notebook_with_decorative_coverindex)

---

## 안드로이드에서 다국어 지원을 위해 해야할 작업

다국어 지원을 위해서는 value resource file을 따로 생성해줘야 한다.

```
1. 'values' 에 'string.xml' 추가
2. 이 때 `available qualifiers` 탭에서 locale 선택
3. `language` 탭에서 언어 선택
4. `specific region only`에서 세부 국가 선택
```

[:arrow_heading_up:](#notebook_with_decorative_coverindex)

---

## Manifest 파일

앱의 이름, 버전, 구성요소, 권한 등 앱의 실행에 있어서 필요한 각종 정보가 저장되어있는 파일이다.
반드시 존재해야하는 xml 형식의 파일로 안드로이드 프로젝트의 최상위에 위치하고 있다.

- `<manifest>`: 패키지명, 앱 버전 코드, 앱 버전 이름을 정의한다.  
- `<uses-permission>`: 앱에서 필요한 권한을 정의한다.  
- `<application>`: 앱 아이콘, 앱 이름을 정의한다.  
- `<activity>`: 액티비티의 클래스명, 이름을 정의한다. 하위에 `<intent-filter>` 태그를 이용해 액티비티에 대한 인텐트 작업 시 필요한 action 과 category를 정의한다.  
- `<service>`, `<receiver>`, `<provider>`: 각각 서비스, 리시버, 프로바이더에 대한 내용을 정의한다.  
- 그 밖에 최소 안드로이드 SDK 버전을 지정하는 `<uses-sdk>`와 다른 패키지를 등록할 수 있는 `<uses-library>` 태그 등이 존재한다.

[:arrow_heading_up:](#notebook_with_decorative_coverindex)

---

## 안드로이드 액티비티의 생명주기

액티비티에는 특정 시점에 호출되는 여러 메서드가 있다. 예를 들어 `onCreate()`는 생성 시점에 호출된다. 이렇게 특정한 타이밍에 호출되는 메서드를 '콜백 메서드'라고 한다.

![activity_lifecycle](/images/activity_lifecycle.png)

### 액티비티 시작

1) `onCreate()`: 액티비티가 시작되면 호출  
2) `onStart()`  
3) `onResume()`  

### 액티비티 종료

1) `onPause()`: 액티비티가 화면에서 보이지 않게 되는 순간 호출  
2) `onStop()`: 완전히 보이지 않게 되면 호출  
3) `onDestroy()`  

### 액티비티 재개

앱을 완전히 종료하지 않고 홈으로 가거나 전원버튼을 통해 화면을 끄는 경우에는 `onPause()`, `onStop()` 까지만 호출되고 대기하게 된다. 이 때 다시 앱이 전면으로 나오는 경우에는 `onRestart()`, `onStart()`, `onResume()` 순으로 호출된다.

### 프로세스 강제 종료

안드로이드의 모든 앱은 백그라운드 실행 중에 메모리 부족 등으로 강제 종료될 수 있다. 이 때 앱을 다시 실행하면 `onCreate()` 메서드부터 호출된다.

### 질문 예시

**Q1) 메인 액티비티에서 하위 액티비티를 호출했을 때 라이프사이클이 호출되는 순서는?**

**A)**

```
1. <MainActivity> onPause()
2. <SubActivity> onCreate()
3. <SubActivity> onStart()
4. <SubActivity> onResume()
5. <MainActivity> onStop()
```

**Q2) 하위 액티비티에서 메인 액티비티로 돌아갈 때 하위 액티비티 종료 후 라이프사이클이 호출되는 순서는?**

**A)**

```
1. <SubActivity> onPause()
2. <MainActivity> onRestart()
3. <MainActivity> onStart()
4. <MainActivity> onResume()
5. <SubActivity> onStop()
6. <SubActivity> onDestroy()
```

[:arrow_heading_up:](#notebook_with_decorative_coverindex)

---

## MVC 아키텍쳐 패턴

안드로이드와 관계없이 프로그래밍 시 가장 널리 사용되는 구조로, **MVC** 란 **Model, View, Control** 의 약자이다.

![mvc_architecture](/images/mvc_architecture.png)

**Model**

- 데이터를 가지며 어플리케이션에서 사용되는 데이터와 그 데이터를 처리한다.  
- **View** 또는 **Control** 에 묶이지 않아 재사용 가능하다.

**View**

- 사용자에게 보일 화면을 표현한다.  
- 앱 및 UI와의 상호작용에서 컨트롤러와 통신한다.  
- 유저가 어떤 입력을 하든 **View** 는 무엇을 해야할지 모른다.

**Control**

- 사용자로부터 입력을 받고 이 입력을 모델에 의해 View 정의를 하게 된다.  
- 모델의 데이터 변화에 따라 뷰를 선택한다.


![mvc_architecture_easy](/images/mvc_architecture_easy.png)

### MVC의 장점

- **Model** 과 **View** 가 서로 분리된다. (완전히는 x)  
- 구현하기 가장 쉽고 단순하다.
- 유닛테스트에서 **View** 는 테스트할 부분이 없기 때문에 쉽게 **Model** 만 테스트 가능하다.

### MVC의 단점

- **Model** 과 **View** 사이의 의존성을 아예 없앨 수 없다. 즉, **View** 의 UI 갱신을 위해 **Model** 을 직/간접적으로 참조하므로 앱 자체가 커지고 로직이 복잡해질수록 유지보수가 힘들어진다.  
- 시간이 지날수록 **Control** 에 많은 코드가 쌓여 코드가 비대화하여 문제가 발생할 가능성이 있다.  

[:arrow_heading_up:](#notebook_with_decorative_coverindex)

---

## MVP 아키텍쳐 패턴

MVP란 **Model, View, Presenter** 의 약자로 MVC와는 다르게 UI를 담당하는 **View** 와 비즈니스 로직을 담당하는 **Model** 을 분리하고, 서로 간의 상호작용을 다른 객체인 **Presenter** 에 그 역할을 줌으로써 <u>서로의 영향(의존성)을 최소화</u>하는 것에 있다.

![mvp_architecture](/images/mvp_architecture.png)

**Model**

- 프로그램 내부적으로 쓰이는 데이터를 저장하고, 처리하는 역할을 한다. → 비즈니스 로직
- **View** 또는 **Presenter** 등 다른 어떤 요소에도 의존적이지 않은 독립적인 영역이다.

**View**

- UI를 담당하며 안드로이드에서는 Activity, Fragment가 대표적인 예이다.
- **Model** 에서 처리된 데이터를 **Presenter** 를 통해 받아서 유저에게 보여준다.
- 유저 액션 및 액티비티 라이프사이클 상태 변경을 주시하며 **Presenter** 에게 보내는 역할이다.
- **Presenter** 를 이용해 데이터를 주고 받기 때문에 **Presenter** 에 매우 의존적이다.

**Presenter**

- **Model** 과 **View** 사이의 매개체이다.
- 모델과 뷰의 매개체라는 점에서 MVC의 Controller와 유사하지만, **View** 에 직접 연결되는 것이 아니라 인터페이스를 통해 상호작용 한다는 점이 다르다.
- 인터페이스를 통해 상호작용하므로 MVC가 가진 테스트 문제와 함께 모듈화/유연성 문제 역시 해결할 수 있다.
- **View** 에게 표시할 내용(Data)만 전달하며 어떻게 보여줄 지는 **View** 가 담당한다.

### MVP의 장점

MVC와는 다르게 코드가 매우 깔끔해지며 MVP를 이용해서 이와 같이 **Model** 과 **View** 간의 결합도를 낮추면, 새로운 기능 추가 및 변경을 해야할 때 관련된 해당 부분만 코드를 수정하면 되기 때문에 확장성이 좋아짐과 동시에 유닛 테스트 시 테스트 코드를 작성하기 편리해지기 때문에 더 쉽게 안전한 코딩이 가능해진다.

또한 UI, Data 각각 파트를 나누기 때문에 해야할 일이 명확해지고 그 결과로 쉽고 빠르게 코딩이 가능하다.

### MVP의 단점

가장 큰 단점은 어플리케이션이 복잡해질수록 **View** 와 **Presenter** 사이의 의존성이 생기는 단점이 있다.  

그리고 MVC의 Controller처럼 **Presenter** 도 어느정도 시간이 지남에 따라 추가 비즈니스 로직이 집중되는 경향이 있다.  

개발자는 시간이 지난 어느 순간 거대해지며 동시에 다루기도 어렵고, 문제가 발생하기 쉽고, 서로 간의 분리를 하기도 어려운 **Presenter** 를 발견하게 된다.  

물론 초기에 설계/기획을 잘함과 동시에 유능한 개발자라면 시간의 흐름에 따른 앱의 다양한 변화에 맞춰서 이 문제를 해결해나갈 수 있겠지만 실제로는 그것도 쉽지만은 않다.

[:arrow_heading_up:](#notebook_with_decorative_coverindex)

---

## MVVM 아키텍쳐 패턴

MVVM이란 **Model, View, ViewModel** 로, 하나의 소프트웨어를 최대한 기능적으로 작은 단위로 나눠 테스트가 쉽고 큰 프로젝트도 상대적으로 관리하기가 좋은 구조이다.  

모든 입력(Input)들은 **View** 로 전달되며 **ViewModel** 은 입력에 해당하는 Presentaion Logic 처리하여 **View** 에 데이터를 전달한다. **ViewModel** 은 **View** 를 따로 참조하지 않기 때문에 독립적이며 **ViewModel** 과 **View** 는 1:n의 관계이다.  

따라서 **View** 는 자신이 이용할 **ViewModel** 을 선택해 데이터를 바인딩하여 업데이트를 받게 된다.  

그 후 **Model** 이 상태 및 데이터가 변경되면 해당하는 **ViewModel** 을 이용하는 **View** 가 자동으로 업데이트 된다.  

마지막으로 **ViewModel** 은 **View** 를 나타내기 위한 **Model** 이자, **View** 의 Presentation Logic을 처리한다.  

MVP와 마찬가지로 **M-V** 사이의 의존성이 없고, **V-VM** 이 1:1 관계가 아닌 독립적 관계인기 때문에 이 둘 사이의 의존성도 없다.

![mvvm_architecture2](/images/mvvm_architecture2.png)

**View**

- Activity나 Fragment 같은 화면에 표현되는 레리아웃을 정의한다.  
- 기본적으로 데이터를 보여주기만 해야하기 때문에 비즈니스 로직을 포함하지 않지만 UI 변경과 관련된 일부 로직은 포함될 수 있다.  
- **ViewModel** 을 관찰하고 있다가 상태 변화가 전달되면 화면을 갱신한다.

**ViewModel**

- **View** 와 **Model** 사이의 매개체 역할을 한다.  
- 모든 **View** 와 관련된 비즈니스 로직은 이 곳에 들어가게 되며 데이터를 잘 가공해서 **View** 에서 뿌리기 쉬운 **Model** 로 바꾸는 역할을 한다.  
- **View** 와 **ViewModel** 은 MVP와는 다르게 1:n의 관계를 가질 수 있으며 여러 개의 Fragment가 하나의 **ViewModel** 을 가질 수 있다.  
- **ViewModel** 은 **View** 가 Databinding할 수 있는 속성과 명령으로 구성되어 있다.

**Model**

- MVC의 Model과 역할은 동일한다.  
- *DataModel* 이라고도 하며 DB, Network, SharedPreference 등 다양한 데이터 소스로부터 필요한 데이터를 준비한다.  
- **ViewModel** 에서 데이터를 가져갈 수 있게 데이터를 준비하고 그에 대한 이벤트를 보낸다.

### MVVM의 장점

**Model** 과 **View** 사이, **ViewModel** 과 **View** 사이의 의존성이 없으므로 유닛 테스트가 더 쉬워지며 MVP 패턴에서 처럼 테스트를 위한 가상 뷰를 만들 필요 없이, 테스트할 때 **Model** 이 변경되는 시점에 Observable 변수가 제대로 설정됐는지 확인하면 된다.  

Databinding 라이브러리를 이용함으로써 서로 간의 의존성을 낮추고, 유닛 테스트를 더욱 쉽게 작성할 수 있고 UI 코드는 네티브 코드에 관여하지 않아도 된다. 또한 중복되는 코드를 모듈화할 수 있다.  

### MVVM의 단점

**View** 가 변수와 표현식 모두에 binding될 수 있으므로 시간이 지남에 따라 관계없는 Presentation Logic이 늘어나고 이를 보완하기 위해 XML에 코드를 추가하게 된다. 이 때 난해하게 코드가 증가된다면 유지보수 단계에서 어려움을 겪을 수 있다.  

"또 **ViewModel** 은 설계하기가 어려운 문제도 있다. 다행히 요즘은 잘 설계된 구조가 샘플 코드로 많이 나와 있기 때문에 해결가능하다" 고 한다.

[:arrow_heading_up:](#notebook_with_decorative_coverindex)

---

<br>

> ###### Reference
> - https://velog.io/@jojo_devstory
> - https://developer.android.com/guide/components/fundamentals#Components  
> - https://github.com/devetude/Android-Interview-QnA  
> - <오준석의 안드로이드 생존코딩 - 코틀린 편>
> - https://medium.com/nspoons/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-architecture-%ED%8C%A8%ED%84%B4-part-1-%EB%AA%A8%EB%8D%B8-%EB%B7%B0-%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC-model-view-controller-881c6fda24d9
