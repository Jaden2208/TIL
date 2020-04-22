# 로그로 해시 키 값 추출하기

카카오 지도 api를 적용하던 중 블로그에 설명되어 있던 방식으로 디버깅 해시 키 값을 설정해주었는데 앱 실행시 지도 화면이 뜨지 않았다.

[Developers.Kakao](https://developers.kakao.com/) 사이트에서 알아보니 해시 키를 구하는 가장 정확한 방법은 **로그를 찍어서 구하는 것** 이라고 나와있었다.

해시 키를 구하는 코드는 아래와 같다.

```kotlin
...
import android.content.pm.PackageManager
import android.util.Base64
import android.util.Log
import java.security.MessageDigest
...

class GetHash : AppCompatActivity() {

  ...

  private fun printHashKey(){
    try {
      val info = packageManager.getPackageInfo(packageName, PackageManager.GET_SIGNING_CERTIFICATES)
      val signatures = info.signingInfo.apkContentsSigners
      val md = MessageDigest.getInstance("SHA")
      for (signature in signatures) {
        val md: MessageDigest
        md = MessageDigest.getInstance("SHA")
        md.update(signature.toByteArray())
        val key = String(Base64.encode(md.digest(), 0))
        Log.d("ㅏㅏㅏHash key:", "ㅋㅋㅋ${key}ㅋㅋㅋ")

      }
    } catch(e: Exception) {
        Log.e("ㅏㅏㅏname not found", e.toString())
    }
  }
}

```
