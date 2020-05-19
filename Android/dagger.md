# Dagger2 란?

## DI(Dependency Injection) 이란?

> Dependency Injection : 의존성 주입

## 의존성이란?

- 코드에서 두 모듈간의 연결
- 두 클래스 간의 관계
- 의존성이 크다는 것은 결합도가 높다는 것.

## 의존성이 미치는 영향은?

- 하나의 모듈이 변경됨에 따라 결합된 다른 모듈이 영향을 받게 된다.
- 두 개의 모듈일 때는 괜찮지만 최악의 경우 모듈이 100개, 1000개 ... 일 때 하나의 모듈변경으로 인해 n-1개의 모듈이 영향을 받는다고 생각해보자
- 나머지 모듈이 제대로 동작하는 지에 대한 검증이 필요할 수도 있다. 그럼 시간과 비용도 n만큼?
- 결합도가 높으면 독립성이 떨어진다. 반대로 결합도가 낮으면 독립성이 높아진다.

## 의존성 주입의 목적(테스트, 유지보수, 재사용성)

- 가장 큰 목적은 모듈을 Testable하게 만들 수 있다는 점이다. 독립된 모듈에 대한 테스트 코드 작성
- 하나의 모듈이 변경되어도 다른 모듈들이 영향을 받지 않으므로 유지보수가 용이
- new를 이용한 생성자를 없애자. 모듈 내에서 다른 모듈을 초기화하면 결합도가 높아지므로 객체생성은 다른 곳에서 하고 생성된 객체를 참조하자.
- 객체생성을 외부에서 하면 클래스의 독립성이 증가되며 이에따라 클래스를 테스트 가능하며, 재사용을 할 가능성도 높아진다.

## Dagger를 이용한 의존성 주입

안드로이드 프레임워크의 4대 컴포넌트인 Activity, Service, ContentProvider, BroadcastReceiver 들이 모여 하나의 어플리케이션을 이룬다. 각 컴포넌트들은 고유의 생명주기를 가지고 있고, Application은 이런 컴포넌트보다 더 상위 개념이기 때문에 컴포넌트의 생명주기에 맞춰 의존성을 주입하기가 좋다.

그러므로, Application은 싱글톤으로서의 역할을 하고, `@Component` 어노테이션을 이용해 Component 단위로 구성하고, 그 하위에 `@Subcomponent` 어노테이션을 이용해서 객체 주입을 하게한다.

## Dagger의 기본 개념

1. Inject
2. Component
3. Subcomponent
4. Module
5. Scope

**Inject**

의존성 주입을 요청한다. `@Inject` 어노테이션으로 주입을 요청하면 연결된 Component가 Module로 부터 객체를 생성하여 넘겨준다.

**Component**

연결된 Module을 이용하여 의존성 객체를 생성하고, Inject로 요청받은 인스턴스에 생성한 객체를 주입한다. 의존성 요청을 받고 주입하는 Dagger의 주된 역할을 수행한다.

**Subcomponent**

Component는 계층관계를 만들 수 있다. Subcomponent는 Inner Class 방식의 하위 계층 Component 이다. Sub의 Sub도 가능하다. Subcomponent는 Dagger의 중요한 컨셉인 그래프를 형성한다. Inject로 주입을 요청받으면 Subcomponent에서 먼저 의존성을 검색하고, 없으면 부모로 올라가면서 검색한다.

**Module**

Component에 연결되어 의존성 객체를 생성한다. 생성 후 Scope에 따라 관리도 한다.

**Scope**

생성된 객체의 Lifecycle 범위이다. 안드로이드에서는 주로 PerActivity, PerFragment 등으로 화면의 생명ㅈ기와 맞추어 사용한다. Module에서 Scope를 보고 객체를 관리한다.


@inject -> Subcomponent -> Module -> Scope에 있으면 return, 없으면 생성.  

Subcomponent Module에서 맞는 타입을 못찾으면 상위 Component -> Module -> Scope에 있으면 return, 없으면 생성.  
