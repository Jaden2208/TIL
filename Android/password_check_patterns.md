# 입력 양식에 맞게 입력됐는지 확인하는 법

```kotlin
  private fun isValid(string: String): Boolean {
        val expression = "^[\\w.-]+@([\\w\\-]+\\.)+[A-Z]{2,4}$"
        val pattern = Pattern.compile(expression, Pattern.CASE_INSENSITIVE)
        val matcher = pattern.matcher(string)
        return matcher.matches()
    }
```
위 코드는 인자로 들어온 `string`이 이메일 형식인지 여부를 반환해주는 함수이다.

여기서 `expression` 값을 아래와 같이 상황에 맞게 써주면 된다.

**이메일**
`"^[\w.-]+@([\w\-]+\.)+[A-Z]{2,4}$"`

**문자 1개 이상 && 숫자 1개 이상 && 길이 8이상**
`"^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{8,}$"`
