# 안드로이드 API 29부터 강화된 네트워크 보안으로 인한 HTTP 사용 불가, 해결 방법은?

서울 열린데이터 광장의 open api를 사용해서 앱 개발을 하던 중 다음과 같은 에러가 발생했다.

`CLEARTEXT communication to {링크} not permitted by network security policy`

검색해보니 **안드로이드 9(API 29) 부터 강화된 네트워크 보안으로 인해서 HTTP 프로토콜 접속을 제한** 해서 발생한 오류였다.

<br>

약 세가지 정도의 해결 방법이 있었는데 그 중 아래의 방법을 사용해서 간단하게 문제를 해결했다.

### AndroidManifest.xml 파일의 <application> 속성에 android:usesCleartextTraffic="true"로 설정

cleartext HTTP와 같은 cleartext 네트워크 트래픽을 사용할지 여부를 나타내는 flag로 이것이 false로 되어있으면, 플랫폼 구성 요소 (예: HTTP 및 FTP 스택, DownloadMangager, MediaPlayer)는 일반 텍스트 트래픽 사용에 대한 앱의 요청을 거부하게 된다. 이 flag를 true로 설정하면 모든 cleartext 트래픽은 허용처리가 된다.

<br>

---

<br>

###### Reference

https://developside.tistory.com/85
