# html 코드에서 텍스트만 뽑아내기

```kotlin
private fun String.htmlToString() : String {
    return if(Build.VERSION.SDK_INT >= 24)
        Html.fromHtml(this, Html.FROM_HTML_MODE_LEGACY).toString()
    else Html.fromHtml(this).toString()
}
```
