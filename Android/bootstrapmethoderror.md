# BootstrapMethodError: Exception from call site #0 bootstrap method 해결 방법

앱에 KAlertDialog 라이브러리를 적용하던 중 다음과 같은 에러를 발견했다.
다이얼로그를 띄우고 확인버튼을 눌러서 종료할 때 앱이 종료되며 발생한 에러이다.

`BootstrapMethodError: Exception from call site #0 bootstrap method`

**자바 8** 이 필요하다는 의미로 앱 수준의 Gradle 파일의 `android { ... }` 안에 아래의 코드를 넣어주면 된다.

```java
android {
  ...

  compileOptions {
    targetCompatibility = "8"
    sourceCompatibility = "8"
  }

  ...
}
```
