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
   worker_connections은 2048로 결정했다.
   



  
