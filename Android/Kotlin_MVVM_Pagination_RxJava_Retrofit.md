# Movie App Project 를 통한 <br> MVVM, Pagination, RxJava, Retrofit 학습

### 완성된 프로젝트 링크
 👉 [MovieAppMVVM](https://github.com/Jaden2208/MovieAppMVVM)


### Index

>**Step 0**
>  - [0. MVVM Architecture](#0-mvvm-architecture)
>
>**Step 1**
>  - [1. Dependency와 Permission 추가하기](#1-dependency와-permission-추가하기)
>
>**Step 2** : SingleMovieDetails
>  - [2. SingleMovieActivity UI 작업](#2-singlemovieactivity-ui-작업)
>  - [3. 네트워크 연결 상태 확인을 위한 NetworkState 클래스 추가](#3-네트워크-연결-상태-확인을-위한-networkstate-클래스-추가)
>  - [4. MovieDetails data class 추가](#4-moviedetails-data-class-추가)
>  - [5. Retrofit을 이용한 네트워크 통신](#5-retrofit을-이용한-네트워크-통신)
>  - [6. RxJava를 이용해 API로 부터 data를 받아온 뒤 Repository에 fetch하기](#6-rxjava를-이용해-api로-부터-data를-받아온-뒤-repository에-fetch하기)
>  - [7. 받아온 데이터들을 SingleMovieActivity에서 보여주기(관찰하기)](#7-받아온-데이터들을-singlemovieactivity에서-보여주기관찰하기)
>
>**Step 3** : PopularMovies
>  - [8. Popular Movie UI 작업](#8-popular-movie-ui-작업)
>  - [9. Pagination 구현을 위한 PageKeyedDataSource 만들기](#9-pagination-구현을-위한-pagekeyeddatasource-만들기)
>  - [10. MovieDataSourceFactory와 Repository 추가하기 + ViewModel](#10-moviedatasourcefactory와-repository-추가하기-viewmodel)
>  - [11. RecyclerView에 보여주기 위한 PagedListAdapter 구현](#11-recyclerview에-보여주기-위한-pagedlistadapter-구현)
>  - [12. 받아온 데이터들을 MainActivity에서 보여주기(관찰하기)](#12-받아온-데이터들을-mainactivity에서-보여주기관찰하기)

<br>



### 0. MVVM Architecture

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

- **<span style="color:Plum">Model</span>**
  + <span style="color:DarkSeaGreen">ViewModel</span> 과 <span style="color:Plum">Model</span> 사이에 Repository가 존재한다.
  + Repository에 Local 또는 네트워크에 있는 data를 fetch한다.
  + Repository는 Local과 네트워크 사이의 중재 역할을 한다.
  + <span style="color:DarkSeaGreen">ViewModel</span> 에서 Repository에 fetch된 데이터를 받아온다.

<br>

### 1. Dependency와 Permission 추가하기

#### 1) Dependency 추가
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

<br>

#### 2) Manifest에 Permission 추가
```kotlin
<uses-permission android:name="android.permission.INTERNET"/>
```

<br>

### 2. SingleMovieActivity UI 작업

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

<br>

### 3. 네트워크 연결 상태 확인을 위한 NetworkState 클래스 추가

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

<br>

### 4. MovieDetails data class 추가

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

<br>

### 5. Retrofit을 이용한 네트워크 통신

#### 1) 네트워크 통신을 위한 interface 추가

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


#### 2) REST API에 request를 보내기 위한 class 추가

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

<br>

### 6. RxJava를 이용해 API로 부터 data를 받아온 뒤 Repository에 fetch하기

#### 1) data 받아오기

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


#### 2) 받아 온 data를 Repository에 fetch하기

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


#### 3) ViewModel에서 SingleMovie Repository 관찰하기

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

### 7. 받아온 데이터들을 SingleMovieActivity에서 보여주기(관찰하기)

<details><summary> SingleMovieActivity.kt </summary>

```kotlin
package com.whalez.movieappmvvm.ui.single_movie_details

import android.annotation.SuppressLint
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.view.View
import androidx.lifecycle.Observer
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import com.bumptech.glide.Glide
import com.whalez.movieappmvvm.R
import com.whalez.movieappmvvm.data.api.POSTER_BASE_URL
import com.whalez.movieappmvvm.data.api.TheMovieDBClient
import com.whalez.movieappmvvm.data.api.TheMovieDBInterface
import com.whalez.movieappmvvm.data.repository.NetworkState
import com.whalez.movieappmvvm.data.vo.MovieDetails
import kotlinx.android.synthetic.main.activity_single_movie.*
import java.text.NumberFormat
import java.util.*

class SingleMovieActivity : AppCompatActivity() {

    private lateinit var viewModel: SingleMovieViewModel
    private lateinit var movieRepository: MovieDetailsRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_single_movie)

        val movieId: Int = intent.getIntExtra("id", 1)

        val apiService: TheMovieDBInterface = TheMovieDBClient.getClient()
        movieRepository = MovieDetailsRepository(apiService)

        viewModel = getViewModel(movieId)

        viewModel.movieDetails.observe(this, Observer {
            bindUI(it)
        })

        viewModel.networkState.observe(this, Observer {
            progress_bar.visibility = if(it == NetworkState.LOADING) View.VISIBLE else View.GONE
            tv_error.visibility = if(it == NetworkState.ERROR) View.VISIBLE else View.GONE
        })
    }

    @SuppressLint("SetTextI18n")
    fun bindUI(it: MovieDetails) {
        tv_title.text = it.title
        tv_tagline.text = it.tagline
        tv_release_date.text = it.releaseDate
        tv_rating.text = it.rating.toString()
        tv_runtime.text =it.runtime.toString() + " minutes"
        tv_overview.text = it.overview

        val formatCurrency = NumberFormat.getCurrencyInstance(Locale.US)
        tv_budget.text = formatCurrency.format(it.budget)
        tv_revenue.text = formatCurrency.format(it.revenue)

        val moviePosterURL = POSTER_BASE_URL + it.posterPath
        Glide.with(this)
            .load(moviePosterURL)
            .into(iv_movie_poster)
    }

    private fun getViewModel(movieId: Int): SingleMovieViewModel {
        return ViewModelProvider(this, object: ViewModelProvider.Factory {
            override fun <T : ViewModel?> create(modelClass: Class<T>): T {
                return SingleMovieViewModel(movieRepository, movieId) as T
            }
        })[SingleMovieViewModel::class.java]
    }
}
```

</details>

- 이전 Activity에서 intent를 통해 전달받은 id 값을 `movieId`로 저장한다.
- Retrofit을 이용한 `TheMovieDBClient`를 `MovieDetailsRepository`의 인자로 하여 **Repository** 를 받아온다.
- `movieId`와 앞서 받아온 **Repository** 를 `SingleMovieViewModel`의 인자로 하는 **ViewModel** 을 만든다.
- 그리고 그 **ViewModel** 의 `movieDetails`와 `networkState`를 관찰해 UI를 업데이트 하도록 한다.

<br>

### 8. Popular Movie UI 작업

#### 1) MovieResponse, Moive data class 추가

<details><summary> MovieRsponse.kt </summary>

```kotlin
package com.whalez.movieappmvvm.data.vo

import com.google.gson.annotations.SerializedName

data class MovieResponse(
    val page: Int,
    @SerializedName("results")
    val movieList: List<Movie>,
    @SerializedName("total_pages")
    val totalPages: Int,
    @SerializedName("total_results")
    val totalResults: Int
)
```

</details>

<br>

<details><summary> Movie.kt </summary>

```kotlin
package com.whalez.movieappmvvm.data.vo

import com.google.gson.annotations.SerializedName

data class Movie(
    val id: Int,
    @SerializedName("poster_path")
    val posterPath: String,
    @SerializedName("release_date")
    val releaseDate: String,
    val title: String
)
```

</details>

<br>

- [[4. MovieDetails data class 추가](#4-moviedetails-data-class-추가)] 과정과 동일하지만 이번에는 details가 아닌 [MoviePopuar](https://developers.themoviedb.org/3/movies/get-popular-movies) 에서 json 을 받아와서 data class를 만들어야 한다.

<br>

#### 2) TheMovieDBInterface에 popular 리스트를 받아오기 위한 메서드 추가.

```kotlin
interface TheMovieDBInterface {
    ...  

    @GET("movie/popular")
    fun getPopularMovie(@Query("page") page: Int): Single<MovieResponse>

    ...

}
```
[MoviePopuar](https://developers.themoviedb.org/3/movies/get-popular-movies) 의 Query String을 보면 **page** 옵션이 있음을 알 수 있다.

![popular_movie_query_string](/images/popular_movie_query_string.png)

이 프로젝트에서는 Pagination을 저 **page** 쿼리를 이용해 구현할 것이기 때문에 `@Query` 어노테이션을 받는 인자를 메서드에 추가해주었다.

<br>

#### 3) MainActivity UI 작업

<details><summary> activity_main.xml </summary>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ui.popular_movie.MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <ProgressBar
            android:id="@+id/progress_bar"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:layout_gravity="center"
            android:visibility="gone"/>

        <TextView
            android:id="@+id/tv_error"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:gravity="center"
            android:text="@string/network_error_msg"
            android:visibility="gone"/>

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/rv_movie_list"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:scrollbars="vertical"
            android:layout_margin="4dp"/>

    </LinearLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
```

</details>

<br>

<details><summary> movie_list_item.xml </summary>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    android:id="@+id/card_view"
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:foreground="?selectableItemBackground"
    android:layout_marginHorizontal="4dp"
    android:layout_marginTop="4dp"
    android:layout_marginBottom="8dp"
    android:focusable="true"
    android:clickable="true">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <ImageView
            android:id="@+id/cv_iv_movie_poster"
            android:layout_width="match_parent"
            android:layout_height="180dp"
            android:scaleType="centerCrop"
            android:layout_gravity="center"
            android:src="@drawable/movie_poster_placeholder"/>

        <TextView
            android:id="@+id/cv_tv_title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="2dp"
            android:layout_marginStart="4dp"
            android:text="movie"
            android:textSize="12sp"
            android:maxLines="1"
            android:ellipsize="end"/>

        <TextView
            android:id="@+id/cv_tv_release_date"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="4dp"
            android:layout_marginBottom="4dp"
            android:text="2019"
            android:maxLength="4"
            android:textSize="12sp"/>

    </LinearLayout>

</androidx.cardview.widget.CardView>
```

</details>

<br>

### 9. Pagination 구현을 위한 PageKeyedDataSource 만들기
- `PageKeyedDataSource` 클래스를 구현하기 전에 이전에 구현했던 `TheMovieDBClient.kt`에 아래와 같이 두 `const val`을 추가해준다.
  ```kotlin
  ...

  const val FIRST_PAGE = 1
  const val POST_PER_PAGE = 20

  ...

  object TheMovieDBClient { ... }

  ```
- 그리고 `PageKeyedDataSource`를 참조하는 `MovieDataSource.kt`를 구현한다.

  <details><summary> MovieDataSource.kt </summary>

  ```kotlin
  package com.whalez.movieappmvvm.data.repository

  import android.util.Log
  import androidx.lifecycle.MutableLiveData
  import androidx.paging.PageKeyedDataSource
  import com.whalez.movieappmvvm.data.api.FIRST_PAGE
  import com.whalez.movieappmvvm.data.api.TheMovieDBInterface
  import com.whalez.movieappmvvm.data.vo.Movie
  import io.reactivex.disposables.CompositeDisposable
  import io.reactivex.schedulers.Schedulers

  class MovieDataSource(
      private val apiService: TheMovieDBInterface,
      private val compositeDisposable: CompositeDisposable
  ) : PageKeyedDataSource<Int, Movie>() {

      private var page = FIRST_PAGE

      val networkState: MutableLiveData<NetworkState> = MutableLiveData()

      override fun loadInitial(
          params: LoadInitialParams<Int>,
          callback: LoadInitialCallback<Int, Movie>
      ) {
          networkState.postValue(NetworkState.LOADING)

          compositeDisposable.add(
              apiService.getPopularMovie(page)
                  .subscribeOn(Schedulers.io())
                  .subscribe(
                      {
                          callback.onResult(it.movieList, null, page+1)
                          networkState.postValue(NetworkState.LOADED)
                      },
                      {
                          networkState.postValue(NetworkState.ERROR)
                          Log.e("MovieDataSource", it.message.toString())
                      }
                  )
          )
      }

      override fun loadAfter(params: LoadParams<Int>, callback: LoadCallback<Int, Movie>) {
          networkState.postValue(NetworkState.LOADING)

          compositeDisposable.add(
              apiService.getPopularMovie(params.key)
                  .subscribeOn(Schedulers.io())
                  .subscribe(
                      {
                          if(it.totalPages >= params.key){
                              callback.onResult(it.movieList, params.key+1)
                              networkState.postValue(NetworkState.LOADED)
                          } else {
                              networkState.postValue(NetworkState.ENDOFLIST)
                          }
                      },
                      {
                          networkState.postValue(NetworkState.ERROR)
                          Log.e("MovieDataSource", it.message.toString())
                      }
                  )
          )
      }

      override fun loadBefore(params: LoadParams<Int>, callback: LoadCallback<Int, Movie>) {

      }
  }
  ```

  </details>

- Page Number 에 따라서 데이터를 load 해야 되기 때문에 `PageKeyedDataSource` 를 참조해야한다.
- `loadInitial()`는 첫 페이지의 데이터를 load하기 위한 오버라이드 메서드이다.
- `loadAfter()`는 사용자가 Scroll Down 할 때 다음 페이지의 데이터를 load하기 위한 오버라이드 메서드이다.
- `loadBefore()`는 사용자가 Scroll Up 할 때 이전 페이지의 데이터를 load하기 위한 오버라이드 메서드 이다. 이 프로젝트에서는 RecyclerView에 이전 페이지의 데이터들을 load하기 때문에 따로 구현하지 않는다.
- `loadInitial()` 메서드에서 `networkState`를 `LOADING`으로 셋팅한 뒤, `apiService`에 `page`를 인자로 전달해준다.
- 그리고 `callback.onReuslt()` 메서드에 `movieList`와 다음 페이지인 `page+1`을 인자로 넘겨준다.
- `loadAfter()` 메서드에도 마찬가지로 구현하지만, 다음 페이지를 의미하는 `key`가 `totalPages` 보다 작거나 같은 경우에만 데이터를 load하도록 구현한다. 그리고 다음 페이지가 없는 경우(totalpages < key)인 경우에는 `networkState`를 `ENDOFLIST`로 셋팅해 준다.

<br>

### 10. MovieDataSourceFactory와 Repository 추가하기 + ViewModel

#### 1) MovieDataSourceFactory 구현하기

<details><summary> MovieDataSourceFactory.kt </summary>

```kotlin
package com.whalez.movieappmvvm.data.repository

import androidx.lifecycle.MutableLiveData
import androidx.paging.DataSource
import com.whalez.movieappmvvm.data.api.TheMovieDBInterface
import com.whalez.movieappmvvm.data.vo.Movie
import io.reactivex.disposables.CompositeDisposable

class MovieDataSourceFactory(
    private val apiService: TheMovieDBInterface,
    private val compositeDisposable: CompositeDisposable
) : DataSource.Factory<Int, Movie>() {

    val moviesLiveDataSource = MutableLiveData<MovieDataSource>()

    override fun create(): DataSource<Int, Movie> {
        return MovieDataSource(apiService, compositeDisposable)
    }
}
```

</details>

- 설명 링크 : [Android Developer - Paging - Custom Data Source](https://developer.android.com/topic/libraries/architecture/paging/data?hl=ko#custom-data-source)

<br>

#### 2) Repository 구현하기 : MoviePagedListRepository

<details><summary> MoviePagedListRepository.kt </summary>

```kotlin
package com.whalez.movieappmvvm.ui.popular_movie

import androidx.lifecycle.LiveData
import androidx.lifecycle.Transformations
import androidx.paging.LivePagedListBuilder
import androidx.paging.PagedList
import com.whalez.movieappmvvm.data.api.POST_PER_PAGE
import com.whalez.movieappmvvm.data.api.TheMovieDBInterface
import com.whalez.movieappmvvm.data.repository.MovieDataSource
import com.whalez.movieappmvvm.data.repository.MovieDataSourceFactory
import com.whalez.movieappmvvm.data.repository.NetworkState
import com.whalez.movieappmvvm.data.vo.Movie
import io.reactivex.disposables.CompositeDisposable

class MoviePagedListRepository(private val apiService: TheMovieDBInterface) {

    private lateinit var moviePagedList: LiveData<PagedList<Movie>>
    private lateinit var moviesDataSourceFactory: MovieDataSourceFactory

    fun fetchLiveMoviePagedList(compositeDisposable: CompositeDisposable): LiveData<PagedList<Movie>> {
        moviesDataSourceFactory = MovieDataSourceFactory(apiService, compositeDisposable)

        val config = PagedList.Config.Builder()
            .setEnablePlaceholders(false)
            .setPageSize(POST_PER_PAGE)
            .build()

        moviePagedList = LivePagedListBuilder(moviesDataSourceFactory, config).build()

        return moviePagedList
    }

    fun getNetworkState(): LiveData<NetworkState> {
        return Transformations.switchMap<MovieDataSource, NetworkState>(
            moviesDataSourceFactory.moviesLiveDataSource, MovieDataSource::networkState
        )
    }
}
```

</details>

- PagedList를 구성하기 위한 `val config`를 만든다.
- `MovieDataSource.kt`에 있는 `networkState`를 받아오기 위해 `getNetworkState()` 메서드를 추가해준다.
- 이 때, `Transformations.switchMap()`메서드를 통해 `MutableLiveData`인 `networkState`를 `LiveData`인 `moviewLiveDataSource`로 변환해서 반환해준다.

<br>

#### 3) MainActivity ViewModel에서 MoviePagedListRepository 관찰하기

<details><summary> MainActivityViewModel.kt </summary>

```kotlin
package com.whalez.movieappmvvm.ui.popular_movie

import androidx.lifecycle.LiveData
import androidx.lifecycle.ViewModel
import androidx.paging.PagedList
import com.whalez.movieappmvvm.data.repository.NetworkState
import com.whalez.movieappmvvm.data.vo.Movie
import io.reactivex.disposables.CompositeDisposable

class MainActivityViewModel(private val movieRepository: MoviePagedListRepository) : ViewModel() {

    private val compositeDisposable = CompositeDisposable()

    val moviePagedList: LiveData<PagedList<Movie>> by lazy {
        movieRepository.fetchLiveMoviePagedList(compositeDisposable)
    }

    val networkState: LiveData<NetworkState> by lazy {
        movieRepository.getNetworkState()
    }

    fun listIsEmpty(): Boolean = moviePagedList.value?.isEmpty() ?: true

    override fun onCleared() {
        super.onCleared()
        compositeDisposable.dispose()
    }
}
```

</details>

- 설명이 필요한 경우 [[6. 3) ViewModel에서 SingleMovie Repository 관찰하기](#3-viewmodel에서-singlemovie-repository-관찰하기)]를 참고할 것.

<br>

### 11. RecyclerView에 보여주기 위한 PagedListAdapter 구현

<details><summary> PopularMoviePagedListAdapter.kt </summary>

```kotlin
package com.whalez.movieappmvvm.ui.popular_movie

import android.content.Context
import android.content.Intent
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.paging.PagedListAdapter
import androidx.recyclerview.widget.DiffUtil
import androidx.recyclerview.widget.RecyclerView
import com.bumptech.glide.Glide
import com.whalez.movieappmvvm.R
import com.whalez.movieappmvvm.data.api.POSTER_BASE_URL
import com.whalez.movieappmvvm.data.repository.NetworkState
import com.whalez.movieappmvvm.data.vo.Movie
import com.whalez.movieappmvvm.ui.single_movie_details.SingleMovieActivity
import kotlinx.android.synthetic.main.movie_list_item.view.*
import kotlinx.android.synthetic.main.network_state_item.view.*

class PopularMoviePagedListAdapter(private val context: Context) :
    PagedListAdapter<Movie, RecyclerView.ViewHolder>(MovieDiffCallback()) {

    val MOVIE_VIEW_TYPE = 1
    val NETWORK_VIEW_TYPE = 2

    private var networkState: NetworkState? = null

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        val layoutInflater = LayoutInflater.from(parent.context)
        val view: View

        return if (viewType == MOVIE_VIEW_TYPE) {
            view = layoutInflater.inflate(R.layout.movie_list_item, parent, false)
            MovieItemViewHolder(view)
        } else {
            view = layoutInflater.inflate(R.layout.network_state_item, parent, false)
            NetworkStateItemViewHolder(view)
        }
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        if (getItemViewType(position) == MOVIE_VIEW_TYPE) {
            (holder as MovieItemViewHolder).bind(getItem(position), context)
        } else {
            (holder as NetworkStateItemViewHolder).bind(networkState)
        }
    }

    private fun hasExtraRow(): Boolean {
        return networkState != null && networkState != NetworkState.LOADED
    }

    override fun getItemCount(): Int {
        return super.getItemCount() + if (hasExtraRow()) 1 else 0
    }

    override fun getItemViewType(position: Int): Int {
        return if (hasExtraRow() && position == itemCount - 1) {
            NETWORK_VIEW_TYPE
        } else {
            MOVIE_VIEW_TYPE
        }
    }

    class MovieDiffCallback : DiffUtil.ItemCallback<Movie>() {
        override fun areItemsTheSame(oldItem: Movie, newItem: Movie): Boolean {
            return oldItem.id == newItem.id
        }

        override fun areContentsTheSame(oldItem: Movie, newItem: Movie): Boolean {
            return oldItem == newItem
        }

    }

    class MovieItemViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        fun bind(movie: Movie?, context: Context) {
            itemView.cv_tv_title.text = movie?.title
            itemView.cv_tv_release_date.text = movie?.releaseDate

            val moviePosterURL = POSTER_BASE_URL + movie?.posterPath
            Glide.with(itemView.context)
                .load(moviePosterURL)
                .into(itemView.cv_iv_movie_poster)

            itemView.setOnClickListener {
                val intent = Intent(context, SingleMovieActivity::class.java)
                intent.putExtra("id", movie?.id)
                context.startActivity(intent)
            }
        }
    }

    class NetworkStateItemViewHolder(view: View) : RecyclerView.ViewHolder(view) {

        fun bind(networkState: NetworkState?) {
            if (networkState != null && networkState == NetworkState.LOADING) {
                itemView.progress_bar_item.visibility = View.VISIBLE
            } else {
                itemView.progress_bar_item.visibility = View.GONE
            }

            if (networkState != null && networkState == NetworkState.ERROR) {
                itemView.error_msg_item.visibility = View.VISIBLE
                itemView.error_msg_item.text = networkState.msg
            } else if (networkState != null && networkState == NetworkState.ENDOFLIST) {
                itemView.error_msg_item.visibility = View.VISIBLE
                itemView.error_msg_item.text = networkState.msg
            } else {
                itemView.error_msg_item.visibility = View.GONE
            }
        }
    }

    fun setNetworkState(newNetworkState: NetworkState) {
        val previousState = this.networkState
        val hadExtraRow = hasExtraRow()
        this.networkState = newNetworkState
        val hasExtraRow = hasExtraRow()

        if (hadExtraRow != hasExtraRow) {
            if (hadExtraRow) {
                notifyItemRemoved(super.getItemCount())
            } else {
                notifyItemInserted(super.getItemCount())
            }
        } else if (hasExtraRow && previousState != newNetworkState) {
            notifyItemChanged(itemCount - 1)
        }
    }
}
```

</details>

- `RecyclerView.Adapter`의 성능을 높이기 위해서 또는 `Paging Component`를 쓰는 경우에 위와 같이 `DiffUtil.ItemCallback`을 반드시 구현해야 한다. `arItemsTheSame()`을 통해 아이템과 새로운 아이템의 ID가 같은지 비교한 뒤 같은 경우 내부에서 `areContentsTheSame()`을 다시 호출해서 객체의 필드가 같은지 비교한다. 만약 `areItemsTheSame()`이 **true** 를 반환하면 `areContentsTheSame()`은 호출되지 않는다. [[참고링크](https://www.charlezz.com/?p=1363)]
- `MovieItemViewHolder` 클래스에는 `setOnClickListener`에서 `intent`로 id 값을 전달하면서 Activity를 실행하도록 한다.
- 네트워크 상태를 보여주기 위해 `NetworkStateItemViewHolder` 클래스를 만들고 만약 `NetworkState` 값에 따라 UI를 수정하도록 구현한다.

<br>

### 12. 받아온 데이터들을 MainActivity에서 보여주기(관찰하기)

<details><summary> MainActivity.kt </summary>

```kotlin
package com.whalez.movieappmvvm.ui.popular_movie

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.view.View
import androidx.lifecycle.Observer
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import androidx.recyclerview.widget.GridLayoutManager
import com.whalez.movieappmvvm.R
import com.whalez.movieappmvvm.data.api.TheMovieDBClient
import com.whalez.movieappmvvm.data.api.TheMovieDBInterface
import com.whalez.movieappmvvm.data.repository.NetworkState
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {

    private lateinit var viewModel: MainActivityViewModel

    lateinit var movieRepository: MoviePagedListRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val apiService: TheMovieDBInterface = TheMovieDBClient.getClient()

        movieRepository = MoviePagedListRepository(apiService)

        viewModel = getViewModel()

        val movieAdapter = PopularMoviePagedListAdapter(this)

        val gridLayoutManager = GridLayoutManager(this, 3)

        gridLayoutManager.spanSizeLookup = object : GridLayoutManager.SpanSizeLookup() {
            override fun getSpanSize(position: Int): Int {
                val viewType = movieAdapter.getItemViewType(position)
                return if (viewType == movieAdapter.MOVIE_VIEW_TYPE) 1
                else 3
            }

        }

        rv_movie_list.apply {
            layoutManager = gridLayoutManager
            setHasFixedSize(true)
            adapter = movieAdapter
        }

        viewModel.moviePagedList.observe(this, Observer {
            movieAdapter.submitList(it)
        })

        viewModel.networkState.observe(this, Observer {
            progress_bar.visibility =
                if (viewModel.listIsEmpty() && it == NetworkState.LOADING) View.VISIBLE
                else View.GONE
            tv_error.visibility =
                if (viewModel.listIsEmpty() && it == NetworkState.ERROR) View.VISIBLE
                else View.GONE

            if(!viewModel.listIsEmpty()) {
                movieAdapter.setNetworkState(it)
            }
        })
    }

    private fun getViewModel(): MainActivityViewModel {
        return ViewModelProvider(this, object : ViewModelProvider.Factory {
            override fun <T : ViewModel?> create(modelClass: Class<T>): T {
                return MainActivityViewModel(movieRepository) as T
            }
        })[MainActivityViewModel::class.java]
    }
}

```

</details>

- `GridLayoutManager()`의 두번째 인자인 `spanCount` 값을 통해 RecyclerView를 몇 열로 배치할지 정할 수 있다.
- 데이터를 로드 중일 때는 `ProgressBar`를 보여줘야 하는데 위 코드와 같이 `spanCount`를 3으로 준다면, `ProgressBar`가 화면의 가로 1/3 크기 만큼 차지 하게 된다. 따라서 `gridLayoutManager.spanSizeLookup` 이 필요하다. `viewType == MOVIE_VIEW_TYPE`이라면 **1** 을 반환하여 3 span 중 **1** 칸을 차지하도록 하고, `viewType == NETWORK_VIEW_TYPE`이라면 **3** 을 반환하여 3 span 중 **3** 칸 전부를 차지하도록 한 것이다.
- 이 외 구현에 대한 설명은 [[7. 받아온 데이터들을 SingleMovieActivity에서 보여주기(관찰하기)](#7-받아온-데이터들을-singlemovieactivity에서-보여주기관찰하기)] 와 거의 같다. 다만, `MainActivity.kt`에서 관찰하는 **ViewModel** 대상이 다를 뿐이다.

<br>




<br>
<br>

[▲ 맨 위로 이동 (Index)](#index)

---

<br>

#### Reference
[YouTube] OX Coding - [Android Movie App MVVM | Paging Library, RxJava, Retrofit](https://www.youtube.com/playlist?list=PLRRNzqzbPLd906bPH-xFz9Oy2IcjqVWCH)
