# RxCocoa
> rxswift-reactive-programming 책을 번역 및 재가공하였습니다


## bind(to:)
- 단방향 데이트흐름
- observable -> observer(subject)
- subscribe 의 특별한 버전이다

### flatMapLatest
- single data source -> multi Observable
- 각각의 원소를 obseravble로 만들어서 각각 감시할 수 있도록
- MVVM과 함께 사용하면 강력하다

## Traits
- 기존의 observable, subject 를 사용할 때에 .observeOn(MainScheduler.instance)를 붙여야 하지만 Traits를 사용하면 이거 안써도 메인스케줄러에서 감시함
- UI 관련 특화된 observable
- 에러가 안난다
- 메인 스케줖러 상에 감시된다
- ControlPropery : UI관련 subject
- Driver: UI 관련 observable
- public typealias Driver<E> = SharedSequence<DriverSharingStrategy, E>
- drive: driver -> observer 단방향 데이터 흐름
- bind -> drive