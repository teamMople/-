# 채팅 고민기록기 - redis sentinel mode 적용해보기
### #redis  #sentinel  #replica
---

#### 처음 문제의식/요구사항 "채팅에 관한 정보는 변동이 너무 잦은걸?"
+ 채팅방에 관한 정보들(참여 인원, 실시간 찬반투표 집계 등)은 변동이 너무 잦다.
+ 이 정보들을 매번 db(mySql)에 접근해서 update 하는 것은 too many connections 일 것이다.
+ 그렇다면? 채팅방 참여인원과 실시간 찬반투표 집계 정보는 redis 서버에서 보유하고 있다가 채팅방 종료시에 mySql에 넣어주자.
+ write-back 방식에 대한 그림.  (이미지 출처: https://rlawls1991.tistory.com/entry/Redis%EB%9E%80)
+ <img width="327" alt="image" src="https://user-images.githubusercontent.com/99469096/161481897-2ff2d491-0adb-4c3b-9da9-f90200c8ca5a.png">

#### 추가 문제의식과 시도 "어, 그런데 write-back 좀 불안한데..?"
+ 인메모리 서버에 문제가 생기면 채팅방 정보가 다 유실되어 버릴 것 같은데.. 어떻게 데이터를 좀 더 안전하게 관리하지..?
+ redis를 사용하는 방식(노드 구성 방식)에는 크게 3가지가 있다는 것 발견.
+ 기존에는 단 하나의 primary 노드만을 사용하고 있었음. (standalone mode라고 함)
+ 그러나 안전하게 관리하려면, 두 가지 옵션이 더 있었다.
  + (1)sentinel mode: replica와 sentinel 사용
  + (2)cluster mode: (1)에 더해서 샤딩(데이터 분산)까지 하기!
---
#### sentinel 적용해보기
+ sentinel mode란? replica 노드를 두고 혹시 primary 노드에 문제가 발생하면 이를 홀수개의 sentinel 노드들이 감지하고 문제라고 판단하여 replica와 primary를 교체!
+ sentinel 방식을 적용해보았다. 위에가 구조 그림, 아래는 sentinel 에서 info를 찍어보았을 때 나오는 정보. (1개 primary node, 2개의 replica node, 3개의 sentinel node) 
 <img width="767" alt="스크린샷 2022-04-04 오후 2 18 52" src="https://user-images.githubusercontent.com/99469096/161478754-21b439e7-0b41-4fae-9a56-a798ed31f274.png">
 <img width="661" alt="ubuntuDio-172-31-11-112 redis-cli -p 26379" src="https://user-images.githubusercontent.com/99469096/161479250-bb36f9da-ea55-4381-93a4-fdcb117fb105.png">

+ 그럼 **failover가 정말로 잘 되는지** 확인!
+ 일부러 primary 노드였던 친구를 ("debug sleep 60") 이렇게 재워놓기. 
+ 그러면 연결 대기해보는 시간 30초(로 설정해두었음)가 지나면 자동으로 sentienl들에 의해 replica 중 하나가 primary로 승격이 되어 있어야 함.
+ 그림1은 구조, 그림2는 replica노드에서 자신의 primary(옛 용어로 master) 노드 정보 조회한 화면, 그림3은 redis에 연결된 WAS에서 자신과 연결된 primary를 재인식한 로그화면.
 <img width="580" alt="스크린샷 2022-04-04 오후 2 19 24" src="https://user-images.githubusercontent.com/99469096/161479025-5143394c-4ef5-4456-96ad-826463a323b7.png">
 <img width="700" alt="Pasted Graphic 5" src="https://user-images.githubusercontent.com/99469096/161479826-f395e116-992c-4390-8fbc-fc557913c216.png">
<img width="1279" alt="Pasted Graphic 7" src="https://user-images.githubusercontent.com/99469096/161481082-a2df32a6-0f18-40ec-9318-e55668ad9b16.png">

##### 작은 추가 정보
> Q. sentinel 노드는 왜 3개이상의 홀수개여야 하나? 
> 
> A. 각 노드들이 primary에 대해 진단한 판단이 오진일 수도 있으니 3개 노드의 판단을 투표(?)해서 과반수 결정이 가능하도록! 이 투표수도 conf 파일에서 설정 가능.

> Q. replica node 는 primary와 같이 안 두는 이유?
> 
> A. primary의 문제 상황에 대비하기 위한 것이다보니 replica는 다른 물리서버에 두는 것이 적절하다.
---
##### 추가로 cluster mode 에 대한 공부기록 (적용해보지는 않음)
<img width="327" alt="image" src="https://user-images.githubusercontent.com/99469096/161481284-7b74a9b8-64d7-4c3f-8ae8-636f63f3941f.png">
(이미지 출처: https://brunch.co.kr/@springboot/218)
+ 데이터셋을 여러 노드에 분산(샤딩)하는 방식! 
+ 데이터가 어떤 노드에 저장될 것인지를 결정하는 알고리즘이 여기선 중요한 개념이라고 함!
  + 각 노드에는 데이터 저장이 될 슬롯이 할당
  + 해시함수/알고리즘에 의해 특정 문자열의 키를 갖는 데이터는 항상 특정 슬롯에 저장되도록 되며, 이 해시 알고리즘을 모든 노드가 공유하므로 어떤 정보가 어디에 저장되어 있는지 모두 알고 있다.

