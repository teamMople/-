# 효율적인 자원관리에 도전!!  
서버를 증설하는건 너무나 쉽다.그래서 주어진 자원을 최대한 효율적으로 사용하는 방법들을 습득하고싶다.  


## nignx의 process 수를 auto-> 2개 
  nginx에서 process를 auto로 설정했다.  
  현재 ec2에서 사용중인 모델은 t2.m 이다(cpu 2개).   
  모든 요청을 1개의 프로세스로 처리하다보니 요청 처리가 늦겠구나! 라고 생각했지만 이는 틀렸다!!.    
### 왜 틀린거지??
  auto는 cpu수 만큼 자동으로 설정되는듯...! 😂
  
## linux는 모든 것을 file로 관리한다!
  linux는 네트워크 호출시 TCP 커넥션 Socket수도 파일로 취급된다.
  따라서 최대 오픈 가능한 파일 수를 늘려주자.   
![image](https://user-images.githubusercontent.com/67067346/161112315-ce682e0f-988f-434e-b5e6-98e11277648e.png)
### 결과
  초반에 한번에 처리 가능한 요청들의 수가 늘었다 빨간색이 file open을 변경해준  결과이다.   
  
## events블럭을 볼까??
  worker_connections을 2배인 2048로 설정했다.성능을 살펴보고 계속 바꿔주자 !(동시에 2048개의 요청을 받을수 있다)  
  multi_accppt 를 on하라는 얘기도 있지만 공식문서에 (https://www.nginx.com/blog/performance-tuning-tips-tricks/) off를 권장하고 있어서 설정하지 않았다.   
  ![image](https://user-images.githubusercontent.com/67067346/161117726-b4894c5c-b006-491b-b6c4-bfe0a06061ed.png)   
   결과는 성공적이다 !! 처음에 처리 가능한 요청들이 늘어나서 그래프가 굵직하게 뭉쳐서 나타나고,중반에도 처리시간이 확연히 줄어든게 보인다!!😁😁😁  
   이번엔 4096으로 설정했다   
   ![image](https://user-images.githubusercontent.com/67067346/161124336-a4fed6b7-2ef7-44c7-96ec-0b8a1ae799b6.png)  
   나중에 성능이 더 나오지 않아서 2048로 교체했다.  
   
 ## tcp설정
### 1.tcp_nopush 
이 설정은 클라이언트로 패킷이 전송되기 전에 버퍼가 가득찼는지 확인하고 만약 가득 찼다면   패킷을 전송하게 해서 네트워크의 오버헤드를 막아준다.  만약 이 설정이 없다면 무지막지하게 큰 패킷이 갈것!    
### 2.tcp_nodelay 
소켓이 패킷의 크기에 상관없이 버퍼에 데이터를 보내도록한다.  이 설정을 사용하는 이유는 전송해야될 데이터의 크기가 상대방의 윈도우 크기가 그보다 더 작은경우 작은 패킷이 만들어지고 전송된다.따라서 보낼 수 있는 데이터를 바로    패킷으로 만들지 않고 버퍼에 저장해서 더 큰 패킷으로 만들어서 보내는 설정이다.  (네이글 알고리즘 설명 https://ozt88.tistory.com/18)   
### 3.rest_timedout_connection 
서버가 응답하지 않는 클라이언트에서 연결을 닫을 수 있게한다.  
응답하지 않는 클라이언트와의 연결을 유지할 이유가 없다!  
### 4.keepalive_timeout
 설정한 시간이 지나면 서버가 연결을 종료하는 설정이다.  
 http는 무상태 특성을 가진다.여기서 keepalive를 주면 연결된 소켓을 계속 유지한다.
 
### 5.client_body_timeout
  요청 시간 초과를 설정해준다.테스트시  getConnection으로 인한 요청 지연이 발생해서 20으로 설정했다.
  
### 6.reset_timedout_connection
  닫힌 소켓이(서버의 응답은 받는게 가능한 상태이다)Fin_wait1상태로 오랫동안 유지되는 상태를 방지해준다.현재 채팅프로그램에서 socket이 닫힌경우 서버에서 Fin_wait1 상태가 발생하는듯 했다 ... !
  
### 6.resolver_timeout 
  😣😅😣😅건들지말자
  
## 1~6의 과정을 적용하고 얻은 결과
  모두 1000명의 Vuser로 테스트를 진행했다   
  응답까지의 시간이 많이 줄었다.
  여전히 시간이 오래 걸리는 이유은 db가 t2.micro여서 connection이 부족하다보니 발생하는듯...?   
![image](https://user-images.githubusercontent.com/67067346/161141817-0dc49a23-0717-41a0-8575-059c7b4c4e99.png)

## 7.reuseport
    listen        80 default_server reuseport;  
    이렇게 설정해서 socket을 재사용😁
    
    
    
    
## 매우 많은 time wait socket
  ![image](https://user-images.githubusercontent.com/67067346/161150435-94463d25-516f-497b-bd8d-3003537b94c7.png)
  엄청나게 많은 Time_Wait socket들이 있다   
  ![image](https://user-images.githubusercontent.com/67067346/161150631-c8b9971b-b3ea-4f94-9e1f-0e1e51ec78a8.png)
  아마 소켓을 이용한 채팅에서 발생한 Time_Wait이지 않을까?? 생각중이다.   
### Time_Wait이 뭐가 문제??
  Time_Wait이 발생하면 소켓이 종료되어 커널로 돌아갈 때까지는 사용할 수 없다.이렇게 되면 커널은 새로운 포트를 할당하고 그러다 모든 포트가 고갈되는 상황이 발생할 수 있다.
### 해결 
  1.default로 할당된 32768 ~ 60999의 포트 범위를 10240 ~ 65535로 늘려준다.  
   하지만 이는 임시방편이다.만약 또 많은 포트들이 Time_Wait에 빠진다면??   
  2.Time_Wait인 소켓을 재사용한다.그러면 Time_Wait에 빠진 수많은 소켓들을 효율적으로 사용 가능하다.


## 결과 😎😎 
  ![image](https://user-images.githubusercontent.com/67067346/161159732-67151696-bcf0-4817-9f73-9bcb867fefa2.png)     
  빨간 동그라미에 잘 뭉쳐져있다.즉 응답속도가 빠르다!! 그리고 응답 속도 그래프를 보다가 최대 60개 까지 올라가는 응답그래프를 보았다  
  아래 사진은 캡쳐타이밍을 못맞췄다😆  현재 로드 밸런서를 이용해서 2대의 ec2를 관리하고 1대의 db를 t2.micro를 이용해서 관리한다.  
  현재 db의 최대 Connection수는 66이다.모든 Connection이 소비되면 다른 요청들은 getConnection을 호출하고 기다린다.만약 여기서  TimeOut이 되면 예외가 던져진다(Noooooooooooooooooooooo...)
  ![image](https://user-images.githubusercontent.com/67067346/161159380-6ddc58fd-df2a-4a54-81b5-8a5e6e235230.png)
  






    
   



  
