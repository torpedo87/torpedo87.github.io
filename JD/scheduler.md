# scheduler
- observable operator가 어디서 작업을 수행할지 지정
- GCD 개념과 비슷한듯
- 멀티스레드 환경에서 스레드간 Observable의 transition을 관리하기 위해 사용

## 스케줄러?
- 프로세스가 발생하는 문맥(스레드, 디스패치큐, 오퍼레이션큐 등)
- background 스케줄러에서는 observable 이 서버에 요청하 데이터를 받아서 cache() 에 의해 가공하고 저장했다가 Main 스케줄러에서 UI를 업데이트한다
- 쓰레드와 비슷하지만 일대일 매칭하지 않는다. 

## subscribeOn 연산자
- Wraps the source sequence in order to run its subscription and unsubscription logic on the specified scheduler.
- This only performs the side-effects of subscription and unsubscription on the specified scheduler.
- public func subscribeOn(_ scheduler: ImmediateSchedulerType) -> Observable<E>
- subscribe 가 호출되는 현재 스레드에 스케줄러가 지정되는 것을 변경 가능
- observable 이 이벤트를 발생시키는 장소 변경
- 어느 시점에서 호출하든지간에 source observable이 이벤트를 방출하는 지점을 지정한다
- 서버로부터 데이터를 받아서 가공하는 것은 일반적으로 background

## observeOn 연산자
- Wraps the source sequence in order to run its observer callbacks on the specified scheduler.
- This only invokes observer callbacks on a scheduler. 
- In case the subscription and/or unsubscription actions have side-effects that require to be run on a scheduler, use subscribeOn.
- public func observeOn(_ scheduler: ImmediateSchedulerType) -> Observable<E>
- source observable이 방출한 아이템 중에 onError 이벤트를 즉시 옵저버에게 전달한다.
- 가공한 데이터를 UI에 뿌려주는 과정은 일반적으로 main
- subscribeOn 연산자와 달리, 호출하는 시점에 따라서 observable이 옵저버에게 전달하는 장소를 지정한다. 

## hot and cold observable problem
- hot = 구독 전에도 발생장소가 있다
- cold = 구독 전까지 발생장소가 없다. 따라서 구독을 시작하면  side effect 발생

## main scheduler
- main thread에서 동작한다. ui 관련 작업
- 데이터를 UI 에 바인딩하는 Driver를 사용하는 작업

## Serial scheduler
- serial dispatch queue 의 작업을 관리
- observeOn과 함께 사용해서 극대화

## Concurrent scheduler
- concurrent dispatch queue를 사용
- combining 작업

## operation queue scheduler
- operationqueue를 사용하는 스케줄러
- concurrent 작업을 좀더 정밀하게 할 때

## test scheduler
- 테스트할 때에만 사용
- cold observable은 200초 후에 구독 시작
- hot observable은 바로 구독 시작

---

# GCD = concurrency를 관리
- 멀티 쓰레드 상황을 전제
- 스레드를 대신 관리해준다

## level
- thread < queue

## race condition
- 복수의 thread가 동시에 접근
- serial queue로 해결

## priority inversion
- task 우선순위에 따라 값에 접근하는 것이 뒤바뀌었을 때
- GCD는 qos 설정을 같게 해서 task의 우선순위를 같게 해서 해결

## deadlock
- 여러 thread 가 서로 기다리다가 잠드는 것
- 현재 큐에서 절대 sync 호출하지 말아야 함

## serial queue, concurrent queue
- 가스불이 여러개 있을 때 시리얼은 1개만 사용 즉 요리 순서가 중요할 때
- 컨커런트는 가스불을 여러개 사용
- 큐가 1개 혹은 복수의 스레드를 사용하는지 결정

## 동기 비동기
- 주어진 가스불을 사용해서 동기는 하던 요리를 마칠때까지 다른 요리를 안하는것. 비동기는 하던 요리를 마치기 전에 다른 요리를 시작할 수 있는것
- 동기 = set get value 할때 사용해서 짧은 시간동안 thread safe 유지
- 현재 큐가 앞의 작업이 완료될때까지 기다려야할지 결정
- 현재 큐에서 task를 동기적으로 할지 비동기적으로 할지 결정 가능
- GCD 효과는 비동기 작업을 다른 큐에서 실행하는 것에 달려있다
