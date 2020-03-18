# Movie App Project 를 통한
## MVVM, Pagination, RxJava, Retrofit 학습


### Index
0. [MVVM Architecture](#mvvm-architecture)
1. [Dependency 추가하기](#dependency-추가하기)
2. [SingleMovieActivity UI 작업](#singlemovieactivity-ui-작업)
3. [네트워크 연결 상태 확인을 위한 NetworkState 클래스 추가](#네트워크-연결-상태-확인을-위한-networkstate-클래스-추가)
4. [MovieDetails data class 추가](#moviedetails-data-class-추가)
5. [Retrofit을 이용한 네트워크 통신](#retrofit을-이용한-네트워크-통신)
6. [RxJava를 이용해 API로 부터 data를 받아온 뒤 Repository에 fetch하기](#rxjava를-이용해-api로-부터-data를-받아온-뒤-repository에-fetch하기)

### MVVM Architecture
![mvvm_architecture](/images/mvvm_architecture.png)

- **<span style="color:LightSlateGrey">View</span>**
  + 사용자가 보고 터치하는 화면. Activity, Fragment 등.
  + 네트워크나 Local에서 데이터를 요청하는 행위 등과 같은 Business Logic에 대한 구현은 하지 않는다.
  + 오직 <span style="color:DarkSeaGreen">ViewModel</span> 로 부터 얻은 것 만을 화면에 보여준다.
<br>
- **<span style="color:DarkSeaGreen">ViewModel</span>**
  + <span style="color:LightSlateGrey">View</span> 를 위한 data를, repository를 통해 제공한다.
  + <span style="color:DarkSeaGreen">ViewModel</span> 은 <span style="color:LightSlateGrey">View</span> 에 대한 어떠한 정보도 가지지 않는다.
  + 따라서 <span style="color:DarkSeaGreen">ViewModel</span> 에서는 적절한 data를 observable하도록 한다.
  + <span style="color:DarkSeaGreen">ViewModel</span> 에 LiveData 들이 있으면 <span style="color:LightSlateGrey">View</span> 는 그것들을 관찰한다.
  + Data가 바뀌면 변경된 data를 관찰하고 있던 <span style="color:LightSlateGrey">View</span> 는 그 변화를 인지한다.
<br>
- **<span style="color:Plum">Model</span>** :
  + <span style="color:DarkSeaGreen">ViewModel</span> 과 <span style="color:Plum">Model</span> 사이에 Repository가 존재한다.
  + Repository에 Local 또는 네트워크에 있는 data를 fetch한다.
  + Repository는 Local과 네트워크 사이의 중재 역할을 한다.
  + <span style="color:DarkSeaGreen">ViewModel</span> 에서 Repository에 fetch된 데이터를 받아온다.



### Dependency 추가하기

```kotlin
// ViewModel and LiveData
implementation 'androidx.lifecycle:lifecycle-extensions:2.2.0'

// Recyclerview and CardView
implementation 'com.google.android.material:material:1.1.0'

// Retrofit2
implementation 'com.squareup.retrofit2:retrofit:2.6.2'
implementation 'com.squareup.retrofit2:converter-gson:2.6.2'
implementation 'com.squareup.retrofit2:adapter-rxjava2:2.5.0'

// Gson
implementation 'com.google.code.gson:gson:2.8.6'

// Glide
implementation 'com.github.bumptech.glide:glide:4.10.0'

// Paging
implementation 'androidx.paging:paging-runtime:2.1.1'

// RxJava2
implementation 'io.reactivex.rxjava2:rxjava:2.2.7'
implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
```

### SingleMovieActivity UI 작업

1. single_movie_details 패키지 생성 후, New - Empty Activity 생성.

2. xml 코드 작성
    <details><summary> activity_single_movie.xml </summary>

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".ui.single_movie_details.SingleMovieActivity">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">

            <ProgressBar
                android:id="@+id/progress_bar"
                android:layout_width="wrap_content"
                android:layout_height="match_parent"
                android:layout_gravity="center"
                android:visibility="gone" />

            <TextView
                android:id="@+id/tv_error"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center"
                android:text="@string/network_error_msg"
                android:visibility="gone" />

            <ScrollView
                android:layout_width="match_parent"
                android:layout_height="match_parent">

                <LinearLayout
                    android:id="@+id/linear_layout"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:orientation="vertical">

                    <ImageView
                        android:id="@+id/iv_movie_poster"
                        android:layout_width="match_parent"
                        android:layout_height="500dp"
                        android:layout_gravity="center"
                        android:scaleType="centerCrop"
                        android:src="@drawable/movie_poster_placeholder" />

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="match_parent"
                        android:layout_margin="8dp"
                        android:orientation="vertical">

                        <TextView
                            android:id="@+id/tv_title"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:text="movie"
                            android:textSize="18sp"
                            android:textStyle="bold" />

                        <TextView
                            android:id="@+id/tv_tagline"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:text="sub title"
                            android:textSize="14sp"
                            android:textStyle="bold" />

                        <TextView
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:text="Movie Info"
                            android:layout_marginTop="15dp"
                            android:textSize="14sp"
                            android:textStyle="bold"/>

                        <LinearLayout
                            android:layout_width="match_parent"
                            android:layout_height="match_parent"
                            android:orientation="horizontal">

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="2dp"
                                android:text="ReleaseDate: "
                                android:textSize="12sp"
                                android:textStyle="bold"/>

                            <TextView
                                android:id="@+id/tv_release_date"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="2dp"
                                android:text="2019"
                                android:textSize="12sp"/>

                        </LinearLayout>

                        <LinearLayout
                            android:layout_width="match_parent"
                            android:layout_height="match_parent"
                            android:orientation="horizontal">

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="2dp"
                                android:text="Rating: "
                                android:textSize="12sp"
                                android:textStyle="bold"/>

                            <TextView
                                android:id="@+id/tv_rating"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="2dp"
                                android:text="8"
                                android:textSize="12sp"/>

                        </LinearLayout>

                        <LinearLayout
                            android:layout_width="match_parent"
                            android:layout_height="match_parent"
                            android:orientation="horizontal">

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="Runtime: "
                                android:textSize="12sp"
                                android:textStyle="bold"/>

                            <TextView
                                android:id="@+id/tv_runtime"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="2dp"
                                android:text="2.3 hr"
                                android:textSize="12sp"/>

                        </LinearLayout>

                        <LinearLayout
                            android:layout_width="match_parent"
                            android:layout_height="match_parent"
                            android:orientation="horizontal">

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="2dp"
                                android:text="Budget: "
                                android:textSize="12sp"
                                android:textStyle="bold"/>

                            <TextView
                                android:id="@+id/tv_budget"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="2dp"
                                android:text="130,000"
                                android:textSize="12sp"/>

                        </LinearLayout>

                        <LinearLayout
                            android:layout_width="match_parent"
                            android:layout_height="match_parent"
                            android:orientation="horizontal">

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="2dp"
                                android:text="Revenue: "
                                android:textSize="12sp"
                                android:textStyle="bold"/>

                            <TextView
                                android:id="@+id/tv_revenue"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="2dp"
                                android:text="130,000"
                                android:textSize="12sp"/>

                        </LinearLayout>

                        <TextView
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:layout_marginTop="2dp"
                            android:text="Overview"
                            android:textSize="12sp"
                            android:textStyle="bold"/>

                        <TextView
                            android:id="@+id/tv_overview"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:layout_marginTop="2dp"
                            android:text="overview ........"
                            android:textSize="12sp"/>

                    </LinearLayout>
                </LinearLayout>

            </ScrollView>

        </LinearLayout>

    </androidx.constraintlayout.widget.ConstraintLayout>
    ```

    </details>


### 네트워크 연결 상태 확인을 위한 NetworkState 클래스 추가

<details><summary> NetworkState.kt </summary>

```kotlin
package com.whalez.movieappmvvm.data.repository

enum class Status {
    RUNNING,
    SUCCESS,
    FAILED
}

class NetworkState(val status: Status, val msg: String) {

    companion object {
        val LOADED: NetworkState = NetworkState(Status.SUCCESS, "Success")
        val LOADING: NetworkState = NetworkState(Status.RUNNING, "Running")
        val ERROR: NetworkState = NetworkState(Status.FAILED, "문제가 발생했습니다!")
        val ENDOFLIST: NetworkState = NetworkState(Status.FAILED, "페이지의 끝에 도달했습니다.")
    }
}
```

</details>


### MovieDetails data class 추가

1. 안드로이드 스튜디오에 'Kotlin data class File from JSON' 플러그인 추가.
2. [themoviedb_details](https://developers.themoviedb.org/3/movies/get-movie-details)에서 json 파일을 복사해서 추가한 플러그인을 이용해 간단하게 data class 파일을 추가.
  - 필요없는 data는 제거하고 변수 타입을 확인하여 필요에 따라서 타입을 수정한다.
  - 그 외 클래스 명, 변수 명 등에 대한 수정이 필요하면 수정한다.

  **주의!**
  변수 명을 수정할 땐 json 객체의 이름과 다른 경우 원래의 이름을 `@SerializedName("{name in json}")` 어노테이션을 추가해줘야 한다.

  <details><summary> MovieDetails.kt </summary>

  ```kotlin
  package com.whalez.movieappmvvm.data.vo

  import com.google.gson.annotations.SerializedName

  data class MovieDetails(
      val budget: Int,
      val id: Int,
      val overview: String,
      val popularity: Double,
      @SerializedName("poster_path")
      val posterPath: String,
      @SerializedName("release_date")
      val releaseDate: String,
      val revenue: Int,
      val runtime: Int,
      val status: String,
      val tagline: String,
      val title: String,
      val video: Boolean,
      @SerializedName("vote_average")
      val rating: Double
  )
  ```

  </details>



### Retrofit을 이용한 네트워크 통신

#### 1. 네트워크 통신을 위한 interface 추가

  <details><summary> TheMovieDBInterface.kt </summary>

  ```kotlin
  package com.whalez.movieappmvvm.data.api

  import com.whalez.movieappmvvm.data.vo.MovieDetails
  import com.whalez.movieappmvvm.data.vo.MovieResponse
  import io.reactivex.Single
  import retrofit2.http.GET
  import retrofit2.http.Path
  import retrofit2.http.Query

  interface TheMovieDBInterface {

      // https://api.themoviedb.org/3/movie/popular?api_key=3a1721be25cfab49572c0fa487fa4258
      // https://api.themoviedb.org/3/movie/454626?api_key=3a1721be25cfab49572c0fa487fa4258
      // https://api.themoviedb.org/3/

      @GET("movie/{movie_id}")
      fun getMovieDetails(@Path("movie_id") id: Int): Single<MovieDetails>

  }
  ```

  </details>

  - REST API를 사용하기 위해서는 themoviedb.org에 가입할 때 얻은 API_KEY를 추가해야 하는데 이건 뒤에서 다룰 `Intercepter{}`를 통해 추가할 수 있다.

  - `getMovieDetails()`가 반환하는 `Single`클래스는 **ReactiveX** 또는 **RxJava** 에서 제공하는 **Observable** 타입의 한 종류이다.

  - **RxJava** 에는 두 개의 주요 구성요소가 있다. **Observable**, **Observer** 이다. **Observable** 은 어떤 작업을 하거나 데이터를 뽑아내는 데이터 스트림이다. 그리고 **Observer** 는 **Observable** 의 대상으로, **Observerable**이 보내는 데이터를 받는다.
  여기서는 일련의 값들을 뽑아내는 것이 아닌 하나의 값(영화 하나) 또는 에러만 뽑아내면 되기 때문에 **Single Observable** 을 사용한 것이다.


#### 2. REST API에 request를 보내기 위한 class 추가

  <details><summary> TheMovieDBClient.kt </summary>

  ```kotlin
  package com.whalez.movieappmvvm.data.api

  import okhttp3.Interceptor
  import okhttp3.OkHttpClient
  import retrofit2.Retrofit
  import retrofit2.adapter.rxjava2.RxJava2CallAdapterFactory
  import retrofit2.converter.gson.GsonConverterFactory
  import java.util.concurrent.TimeUnit

  const val API_KEY = "{MY_API_KEY}"
  const val BASE_URL = "https://api.themoviedb.org/3/"

  const val POSTER_BASE_URL = "https://image.tmdb.org/t/p/w342"

  const val FIRST_PAGE = 1
  const val POST_PER_PAGE = 20

  // https://api.themoviedb.org/3/movie/popular?api_key=3a1721be25cfab49572c0fa487fa4258&page=1
  // https://api.themoviedb.org/3/movie/454626?api_key=3a1721be25cfab49572c0fa487fa4258
  // https://image.tmdb.org/t/p/w342//aQvJ5WPzZgYVDrxLX4R6cLJCEaQ.jpg

  object TheMovieDBClient {

      fun getClient(): TheMovieDBInterface {
          val requestInterceptor = Interceptor {
              val url = it.request()
                  .url()
                  .newBuilder()
                  .addQueryParameter("api_key", API_KEY)
                  .build()

              val request = it.request()
                  .newBuilder()
                  .url(url)
                  .build()

              return@Interceptor it.proceed(request)
          }

          val okHttpClient = OkHttpClient.Builder()
              .addInterceptor(requestInterceptor)
              .connectTimeout(60, TimeUnit.SECONDS)
              .build()

          return Retrofit.Builder()
              .client(okHttpClient)
              .baseUrl(BASE_URL)
              .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
              .addConverterFactory(GsonConverterFactory.create())
              .build()
              .create(TheMovieDBInterface::class.java)
      }
  }
  ```

  </details>

- `POSTER_BASE_URL`은 Json 객체를 보면 "/aQvJ5WPzZgYVDrxLX4R6cLJCEaQ.jpg" 같은 형식으로 저장되어 있기 때문에 나중에 사진을 불러오기 위해서 필요하다.

- **Interceptor** 를 이용해 REST API를 위한 `API_KEY`를 추가한다.

- `okHttpClient`에서의 `connectTimeout()`을 이용해 요청을 시작한 후 서버와의 TCP handshake가 완료되기까지 지속되는 시간을 설정할 수 있다. 제한 시간 내에 서버에 연결할 수 없는 경우 해당 요청을 실패한 것으로 처리한다.


### RxJava를 이용해 API로 부터 data를 받아온 뒤 Repository에 fetch하기

#### 1. data 받아오기

  <details><summary> MovieDetailsNetworkDataSource.kt </summary>

  ```kotlin
  package com.whalez.movieappmvvm.data.repository

  import android.util.Log
  import androidx.lifecycle.LiveData
  import androidx.lifecycle.MutableLiveData
  import com.whalez.movieappmvvm.data.api.TheMovieDBInterface
  import com.whalez.movieappmvvm.data.vo.MovieDetails
  import io.reactivex.disposables.CompositeDisposable
  import io.reactivex.schedulers.Schedulers

  class MovieDetailsNetworkDataSource(
      private val apiService: TheMovieDBInterface,
      private val compositeDisposable: CompositeDisposable
  ) {
      private val _networkState = MutableLiveData<NetworkState>()
      val networkState: LiveData<NetworkState>
          get() = _networkState

      private val _downloadedMovieDetailsResponse = MutableLiveData<MovieDetails>()
      val downloadedMovieResponse: LiveData<MovieDetails>
          get() = _downloadedMovieDetailsResponse

      fun fetchMovieDetails(movieId: Int) {
          _networkState.postValue(NetworkState.LOADING)

          try {
              compositeDisposable.add(
                  apiService.getMovieDetails(movieId)
                  .subscribeOn(Schedulers.io())
                      .subscribe(
                          {
                              _downloadedMovieDetailsResponse.postValue(it)
                              _networkState.postValue(NetworkState.LOADED)
                          },
                          {
                              _networkState.postValue(NetworkState.ERROR)
                              Log.e("MovieDetailDataSource", it.message.toString())
                          }
                      ))

          } catch (e: Exception){
              Log.e("MovieDetailDataSource", e.message.toString())
          }
      }
  }
  ```

  </details>

- RxJava를 이용해 API를 호출한다. 그러면 API는 MovieDetails를 반환하고 이 MovieDetails를 LiveData에 할당한다.
- `CompositeDisposable`은 API 호출을 dispose(배치?)하기 위해 사용할 수 있는 **RxJava** 의 구성 요소이다. 따라서 **RxJava Thread** 를 dispose하기 위해서 이를 사용할 수 있다.
- `_networkState`는 mutable(변경가능)한 `MutableLiveData`를 사용한다.
- `getMovieDetails()`를 통한 네트워크 호출에 성공하면 `MovieDetails`를 얻을 수 있으며, 이때 얻은 `MovieDetails`는 `_downloadedMovieDetailsResponse`에 할당해준다.


#### 2. 받아 온 data를 Repository에 fetch하기

<details><summary> MovieDetailsRepository.kt </summary>

```kotlin
package com.whalez.movieappmvvm.ui.single_movie_details

import androidx.lifecycle.LiveData
import com.whalez.movieappmvvm.data.api.TheMovieDBInterface
import com.whalez.movieappmvvm.data.repository.MovieDetailsNetworkDataSource
import com.whalez.movieappmvvm.data.repository.NetworkState
import com.whalez.movieappmvvm.data.vo.MovieDetails
import io.reactivex.disposables.CompositeDisposable

class MovieDetailsRepository(private val apiService: TheMovieDBInterface) {

    private lateinit var movieDetailsNetworkDataSource: MovieDetailsNetworkDataSource

    fun fetchSingleMovieDetails(
        compositeDisposable: CompositeDisposable,
        movieId: Int
    ): LiveData<MovieDetails> {
        movieDetailsNetworkDataSource =
            MovieDetailsNetworkDataSource(apiService, compositeDisposable)
        movieDetailsNetworkDataSource.fetchMovieDetails(movieId)

        return movieDetailsNetworkDataSource.downloadedMovieResponse
    }

    fun getMovieDetailsNetworkState(): LiveData<NetworkState> {
        return movieDetailsNetworkDataSource.networkState
    }
}
```

</details>


#### 3. ViewModel에서 SingleMovie Repository 관찰하기

<details><summary> SingleMovieViewModel.kt </summary>

```kotlin
package com.whalez.movieappmvvm.ui.single_movie_details

import androidx.lifecycle.LiveData
import androidx.lifecycle.ViewModel
import com.whalez.movieappmvvm.data.repository.NetworkState
import com.whalez.movieappmvvm.data.vo.MovieDetails
import io.reactivex.disposables.CompositeDisposable

class SingleMovieViewModel(private val movieRepository: MovieDetailsRepository, movieId: Int) :
    ViewModel() {

    private val compositeDisposable = CompositeDisposable()

    val movieDetails: LiveData<MovieDetails> by lazy {
        movieRepository.fetchSingleMovieDetails(compositeDisposable, movieId)
    }

    val networkState: LiveData<NetworkState> by lazy {
        movieRepository.getMovieDetailsNetworkState()
    }

    override fun onCleared() {
        super.onCleared()
        compositeDisposable.dispose()
    }
}
```

</details>

- `movieDetails`와 `networkState`는 `by lazy`를 통해 `SingleMovieViewModel`이 초기화 될 때 얻어오는 것이 아닌 필요할 때 얻어오도록 한다. <u>(성능 향상을 위한 구현)</u>

- Activity나 Fragment가 destroy될 때, **memory leak** 을 방지하기 위해 `onCleared()` 메서드를 오버라이드 하여 `compositeDisposable`을 dispose(처분) 해준다.



<br>

---

#### Reference
[YouTube] OX Coding - [Android Movie App MVVM | Paging Library, RxJava, Retrofit](https://www.youtube.com/playlist?list=PLRRNzqzbPLd906bPH-xFz9Oy2IcjqVWCH)
