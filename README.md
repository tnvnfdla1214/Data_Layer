# Data_Layer

이 포스터는 [앱 아키텍쳐 가이드](https://developer.android.com/jetpack/guide?hl=ko)으로 학습후 정리한 내용입니다.



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

원샷 작업: 데이터 영역에서 Kotlin의 정지 함수를 노출해야 합니다. 자바 프로그래밍 언어의 경우 데이터 영역에서 작업 결과 또는 RxJava Single, Maybe 또는 Completable 유형에 대한 콜백을 제공하는 함수를 노출해야 합니다.
시간 경과에 따른 데이터 변경사항에 관해 알림을 받으려면: 데이터 영역에서 Kotlin의 흐름을 노출해야 합니다. 자바 프로그래밍 언어의 경우 데이터 영역에서 새 데이터 또는 RxJava Observable 또는 Flowable 유형을 내보내는 콜백을 노출해야 합니다.
