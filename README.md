# Data_Layer

이 포스터는 [앱 아키텍쳐 가이드](https://developer.android.com/jetpack/guide?hl=ko)으로 학습후 정리한 내용입니다.
이 포스터에서는 [ToDo](https://github.com/tnvnfdla1214/ToDo)을 참고해주시 바랍니다.


데이터 영역은 앱에서 처리하는 다양한 유형의 데이터별로 저장소 클래스를 만들어야 합니다. 예를 들어 영화 관련 데이터에는 MoviesRepository 클래스를 만들거나 결제 관련 데이터에는 PaymentsRepository 클래스를 만들 수 있습니다.

<div align="center">
<img src = "https://user-images.githubusercontent.com/48902047/150826357-8c47b433-cb62-4d48-af74-b4674ae7937f.png" width="50%" height="50%">
</div>

계층 구조의 다른 레이어는 데이터 소스에 직접 액세스해서는 안 됩니다. 데이터 영역의 진입점은 **항상 저장소 클래스(Repository)여야 합니다.** ViewModel(UI레이어)나 UseCase(Damain레이어)에서 데이터 소스가 직접 종속 항목으로 있어서는 안 됩니다. 저장소 클래스(Repository)를 진입점으로 사용하면 아키텍처의 다양한 레이어를 독립적으로 확장할 수 있습니다.

 ```Kotlin
class ExampleRepository(
    private val exampleRemoteDataSource: ExampleRemoteDataSource, // network
    private val exampleLocalDataSource: ExampleLocalDataSource // database
) { /* ... */ }
```

## API 노출
데이터 영역의 클래스는 일반적으로 원샷 생성, 조회, 업데이트 및 삭제(CRUD) 호출을 실행하거나 시간 경과에 따른 데이터 변경사항에 관해 알림을 받는 함수를 노출합니다. 데이터 영역은 다음과 같은 경우에 각 항목을 노출해야 합니다.

원샷 작업: 데이터 영역에서 Kotlin의 정지 함수(Flow,상태 등)를 노출해야 합니다. 자바 프로그래밍 언어의 경우 데이터 영역에서 작업 결과 또는 RxJava Single, Maybe 또는 Completable 유형에 대한 콜백을 제공하는 함수를 노출해야 합니다.
시간 경과에 따른 데이터 변경사항에 관해 알림을 받으려면: 데이터 영역에서 Kotlin의 흐름을 노출해야 합니다. 자바 프로그래밍 언어의 경우 데이터 영역에서 새 데이터 또는 RxJava Observable 또는 Flowable 유형을 내보내는 콜백을 노출해야 합니다.

 ```Kotlin
class ExampleRepository(
    private val exampleRemoteDataSource: ExampleRemoteDataSource, // network
    private val exampleLocalDataSource: ExampleLocalDataSource // database
) {

    val data: Flow<Example> = ...

    suspend fun modifyData(example: Example) { ... }
}
```
## 여러 수준의 저장소
더 복잡한 비즈니스 요구사항이 포함된 일부 경우에는 저장소가 다른 저장소에 종속되어야 할 수 있습니다. 관련된 데이터가 여러 데이터 소스의 집계이거나 책임이 다른 저장소 클래스에 캡슐화되어야 하기 때문일 수 있습니다.

예를 들어 사용자 인증 데이터를 처리하는 저장소인 UserRepository는 요구사항을 충족하기 위해 LoginRepository 및 RegistrationRepository와 같은 다른 저장소에 종속될 수 있습니다.

<div align="center">
<img src = "https://user-images.githubusercontent.com/48902047/150829548-6b9d3e57-7301-42be-bdce-510fd607185c.png" width="50%" height="50%">
</div>

## 스레딩
데이터 소스와 저장소 호출은 기본 스레드에서 호출하기에 안전하도록 기본 안전성이 보장되어야 합니다. 이러한 클래스는 장기 실행 차단 작업을 실행할 때 로직 실행을 적절한 스레드로 이동합니다. 예를 들어 데이터 소스가 파일에서 읽거나 저장소가 큰 목록에서 비용이 많이 드는 필터링을 수행할 때 기본 안전성이 보장되어야 합니다.

대부분의 데이터 소스는 이미 Room 또는 Retrofit에서 제공하는 정지 메서드 호출과 같은 기본 안전성을 갖춘 API를 제공합니다. API를 사용할 수 있게 되면 저장소에서 API를 활용할 수 있습니다.

Kotlin 사용자의 경우 **코루틴**을 사용하는 것이 좋습니다.

## 수명 주기
클래스에 메모리 내 데이터가 포함된 경우(예: 캐시) 특정 기간 동안 해당 클래스의 동일한 인스턴스를 재사용하고자 할 수 있습니다. 이를 클래스 인스턴스의 수명 주기라고도 합니다.

클래스의 책임이 전체 애플리케이션에 중요한 경우 해당 클래스의 인스턴스 범위를 Application 클래스로 지정할 수 있습니다. 이렇게 하면 인스턴스가 애플리케이션의 수명 주기를 따르게 됩니다. 또는 앱의 특정 흐름(예: 등록 또는 로그인 흐름)에서만 동일한 인스턴스를 재사용해야 하는 경우 흐름의 수명 주기를 소유한 클래스로 인스턴스 범위를 지정해야 합니다. 예를 들어 메모리 내 데이터가 포함된 RegistrationRepository 범위를 RegistrationActivity 또는 등록 흐름의 탐색 그래프로 지정할 수 있습니다.

## 대표 비즈니스 모델
이 포스터에서는 데이터 모델은 다양한 데이터 소스에서 가져오는 정보의 하위집합이라 합니다.

네트워크나 로컬에서 가져오는 정보는 정부 사용하지 않으므로 모델 클래스를 분리하고 저장소에서 계층 구조의 다른 레이어에 필요한 데이터만 노출하도록 하는 것을 추천합니다.

예를 들어 아래와 같이 기사 정보뿐만 아니라 수정 기록, 사용자 댓글, 일부 메타데이터도 반환하는 News API 서버가 있을 경우 필요한 정보로 가공하여 model을 만들어 주는것이 정보를 다듬는 방법이라 설명합니다.
 ```Kotlin
data class ArticleApiModel(
    val id: Long,
    val title: String,
    val content: String,
    val publicationDate: Date,
    val modifications: Array<ArticleApiModel>,
    val comments: Array<CommentApiModel>,
    val lastModificationDate: Date,
    val authorId: Long,
    val authorName: String,
    val authorDateOfBirth: Date,
    val readTimeMin: Int
)
```
 ```Kotlin
data class Article(
    val id: Long,
    val title: String,
    val content: String,
    val publicationDate: Date,
    val authorName: String,
    val readTimeMin: Int
)
```

이 방식을 확장하고 앱 아키텍처의 다른 부분(예: 데이터 소스 클래스 및 ViewModel)에서도 별도의 모델 클래스를 정의할 수 있습니다.

그러나 이를 위해서는 적절하게 문서화하고 테스트해야 하는 추가 클래스 및 로직을 정의해야 합니다.

**최소한 데이터 소스가 앱의 나머지 부분에서 예상하는 데이터와 일치하지 않는 데이터를 수신하는 경우에는 새 모델을 만드는 것이 좋다고 설명합니다.**



