# Process

## process?
- 메모리에 로드된 실행가능한 프로그램
- text, data(global variable, heap -> <- stack(local variable, func)

## scheduling, creation, termination
- 프로세스의 상태
- new - ready, waiting - running - terminated
- Process Control Block = process 정보를 저장하는 repository 역할
- scheduling = new process -> ready queue -> dispatched to cpu ->  execute
- context switch = 실행중인 프로세스를 다른 프로그램으로 바꿀 때 진행중이던 상태를 저장

## iOS 멀티태스킹
- forground = 1개 실행, display
- background = 여러개 실행 but not displayed, 배터리수명과 메모리 문제로 제한된 갯수의 프로그램만 실행하고 나머지는 suspend

## interprocess communication - shared memory, massage passing
- shared memory = 변수를 공유한다. 프로그래머가 구현해야함. 프로세스간에 공유된 메모리 영역에서 많은 양의 정보를 빨리 교환. shared memory가 있기 때문에 굳이 커널을 거치지 않아도 되어서 빠르다
- message passing = 메시지를 통해 프로세스간 적은 양의 정보를 느리게 교환, 구현이 쉽다. system call을 사용하므로 커널을 거쳐야 하므로 느리다 동기 또는 비동기적으로 동작

## communication in client server system
- socket = communication의 양쪽 말단. IP주소로 구별
- remote procedure calls = 일반적. process가 remote application 상에서 procedure를 호출할 때 사용
- pipes