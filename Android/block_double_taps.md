# 더블 탭으로 인해서 같은 화면이 두번 켜지는 경우?

아래의 방법으로 더블 탭을 방지할 수 있다.

```kotlin
companion object{
            const val CLICK_TIME_INTERVAL = 500
}
private var mLastClickTime = System.currentTimeMillis()

...

view.setOnClickListener {
  val clickedTime = System.currentTimeMillis()
  if(clickedTime - mLastClickTime < CLICK_TIME_INTERVAL) return@setOnClickListener
  mLastClickTime = clickedTime

  // 더블 탭이 아닌 경우의 동작 구현
  ...

}
```
