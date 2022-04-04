### 현재 redis 용도
1. Look-aside - 가볍게 조회하기
> 과정
>  1) client -> webApp server로 데이터 조회 요청
>  2) webApp -> redis 캐시에서 먼저 확인하여 있으면 이를 client에게 전달
>  3) redis에 없을 시 db에서 데이터 가져와 전달
> 
> 적용 : 게시글 
2. Write-back - 가볍게 (하지만 조금 불안하게) 쓰기
> 과정
> 1) client -> webApp server로 데이터 전달
> 2) redis에 저장
> 3) 특정 시점에 redis -> db(mysql)로 데이터 전달
>
> 적용 : 채팅방 종료 전까지 찬반투표와 참여인원 집계
> 
3. Message-broker(pub/sub) - 실시간 텍스트 채팅 및 찬반투표의 브로커
> 과정
> 1) 참여자는 특정 채널을 구독(subscribe)
> 2) 채널에 메시지 발행(publish)
> 3) 발행 측에서는 구독자(subscriber)들에 대한 정보를 파악하지 않음(느슨한 결합)
> 
> ( 발행측은 수신 완료 여부는 관계 없이 자신의 작업을 비동기적으로 진행 )
***
### redis 에 대해 고민했던 점들
1. 여러 용도를 하나의 redis로 사용해도 될지 (ex. cache, message broker(pub/sub))
> 멘토링 피드백 및 잠정 결론
> + 용도가 다르면 분리하는 것이 좋다. 하지만 서비스 규모, 개발 단계 등을 고려할 때 우선 하나의 서버에서 진행해도 된다.
> 
> + 다만 용도별 서버분리가 필요하고 좋은 이유에 대해 주지하고 있자.

2. standalone 모드 or sentinel or cluster mode 로 전환해야 하는지 고민
* 고민의 내용
> + 우리 프로젝트에서 redis는 일정 시간동안 데이터를 보유하고 있다가 나중에 db에 저장하는 write-back 역할과, 채팅의 broker 역할을 맡고 있다. 그러다보니 데이터 손실이 걱정되었다.
> + 그래서 찾아보니 redis를 하나의 primary node만으로 사용하는 standalone 보다 안정적인 방식들이 있었다.

>(1) sentinel mode (이에 대한 적용시도과정은 redis 노드 구성에 대한 기록으로 남겨두기)
> * primary node뿐만 아니라 replica node를 사용한다.
> * sentinel node은 primary/replica를 모니터링한다. primary에 문제 발생시 replica를 primary로 승격시켜야 한다.
> * 이를 위해서는 3개 이상 홀수의 sentinel node 가 있어야 하는데, 과반 수 이상의 동의를 얻어 failover를 진행하기 때문이다.

> (2) cluster mode
> * 단지 복제본을 만드는 것이 아니라, 데이터 셋을 여러 node에 분산하여 저장(샤딩)한다.
> * 즉 여러 개의 primary가 존재하며, 보통의 경우 각각에 대한 replica는 서로 다른 물리서버에 두는 것이 안정적이다.
> * 성능 향상의 이점을 가질 수 있다.
#### 멘토링 피드백 및 잠정 결론
> * 현 상황에서 redis를 위한 서버 여럿을 두는 것은 과도. 다만 목적/의도 측면에서는 구조에 대한 고민과 변경은 좋음. 실제 서비스에서 트래픽이 많아지거나 하는 등의 필요가 나타난다면 고려할 수 있음.

***
### redis 사용시 주의점
> 메모리 관리
> (1) 커다란 메모리 용량 하나 보다는 작은 메모리 여러 개 서버를 두는 것이 좋다.
>  
> (2) redis 는 싱글스레드이므로 시간이 오래 걸리는 단일 명령은 피하자. 그보다는 해당 명령을 잘게 쪼개서 요청하도록 하자.
> * (피해야 할 명령의 예 - keys *, flushall, delete collections ... ) -> keys 보다는  scan!


