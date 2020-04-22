# Toolbar 내부에 View를 넣을 때 원치 않는 패딩이 왼쪽에 있을 때?

다음과 같이 해주면 왼쪽 패딩이 없어진다.

```xml
<androidx.appcompat.widget.Toolbar
        android:id="@+id/tool_bar"

        ...

        android:contentInsetStart="0dp"
        android:contentInsetLeft="0dp"
        android:contentInsetEnd="0dp"
        android:contentInsetRight="0dp"
        app:contentInsetEnd="0dp"
        app:contentInsetLeft="0dp"
        app:contentInsetRight="0dp"
        app:contentInsetStart="0dp">
```
