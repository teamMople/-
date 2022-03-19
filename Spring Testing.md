# Test Code를 작성하며 알게된것😥
  
/gradlew test입력하여 테스트를 실행하면  
![image](https://user-images.githubusercontent.com/67067346/159112138-1cf97d3c-c938-431f-9b4c-9014fcfe028d.png)  
이런 화면이 나온다!!   
저 화면은 Applicationcontext 로딩시에 올라온는듯 하다.  
Test가 아닌 환경에서는 단 한번 로딩된다.   
그렇게 평화로운 날들을 보내다가 유투브에서 https://www.youtube.com/watch?v=N06UeRrWFdY를 보았고  허접한 내 코드들을 다시 돌아보았다...    

## 나의 착각들 
나는 무지하게도 Test환경도 Applicationcontext를 하나만 올리고 실행하겠지~ 라고 생각하고 신경도 안 쓰고 있었다...🤦‍♂   
빠르게 나의 잘못된 생각들을 바로잡고자 구글링후  테스트를 했다   
(참고한글 >> https://suhwan.dev/2019/03/27/spring-test-context-management-and-caching/)

```java
@Configuration
public class ConfigA {
    @Bean
    public BeanA beanA(){
        return new BeanA();
    }
}
=======================================
@Configuration
public class ConfigB {

    @Bean
    public BeanB beanB(){
        return  new BeanB();
    }
}
=======================================
@Component
public class ComponentC  {

}
=======================================
@Component
public class ComponentD {

}
```
![image](https://user-images.githubusercontent.com/67067346/159112698-7954e937-a00a-4809-9570-8e942754322b.png)    


일단 테스트 코드 파일을 2개 작성했다 . 

```java 

@ExtendWith(SpringExtension.class)
@SpringBootTest

public abstract class Testing {
    @Autowired
    ApplicationContext applicationContext;
}
```   
```java
@ContextConfiguration(
        classes = {  ConfigA.class }
)
public class SomeIntegrationTest1  extends Testing  {
    @Test
    void someTest1(){
        System.out.println(applicationContext); 
    }
}
```
```java
@ContextConfiguration(
        classes = {  ConfigB.class }
)
public class SomeIntegrationTest2  extends  Testing  {
    @Test
    void someTest2(){
        System.out.println(applicationContext);
    }
}
```   
 <em>"Applicationcontext는 언제 새롭게 로딩되는가 ??"<em>
 
@ContextConfiguration 을 다르게 주면 SomeIntegrationTest1과 SomeIntegrationTest2 각각의 Context가 생성된다  
  ![image](https://user-images.githubusercontent.com/67067346/159113085-6a2ce61d-8893-463d-beaf-e511ded6c1bb.png)
  ![image](https://user-images.githubusercontent.com/67067346/159113103-f77eed0d-f232-41c7-9cb2-445b374702ca.png)

 즉 서로다른 ContextConfiguration을 가지게 되면 Spring은 각각의 Applicationcontext(설계도가 다른 느낌...?)를 생성한다 
 당연히 "오 래 걸 린 다 내 컴퓨터는 더 ~ 오래걸려..."
  😎  왜 오래 걸리면 안되는건가??
  1. 배포를 하기전에 보통 Test로 검증을 수행한다. "Test"를 수행한다 만약 Test에서의 시간이 오래 걸린다면 ??
  2. "Test"가 주는 강점은 빠른 피드백과 빠른 검증이라고 생각한다.  
     만약 내가 작성한 테스트 코드가 오래걸린다면 그 장점을 하나 죽이게 되고,테스트가 완료될 때까지 나는 유투브나 보고 있지 않을까...?  
     (실제로 30초 걸리는 테스트도 유투브 봤다...😅)
  3. CI의 목표는 무엇인가 ??   
      버그를 신속하게 찾아 해결하고,  
      소프트웨어의 품질을 개선하고,   
      새로운 업데이트의 검증 및 릴리즈의 시간을 단축시키는 것에 있습니다.  
      출처: https://artist-developer.tistory.com/24 [개발자 김모씨의 성장 일기] (감사합니다 김모씨님)   
      "신속"이다 . 내가 만든 Test가 오래 걸리면 신속은 어렵겠지 ... ? 
  
  그럼 어떻게 조금이라도 더 빠른 Test Code를 작성할 수 있을까??
  ##   Context caching.    
    Spring의 TestContext 프레임 워크는 ApplicationContext가 만들어지면 이를 캐시에 저장한다 
    그리고 다른 테스트를 실행 시에 사용 가능한 ApplicationContext를 꺼내서 사용한다.
    이렇게 되면 다시 ApplicationContext를 만들지 않아도 된다.
    띠용 ...? 코드로 보자... 🙄
  
 ```java
@ExtendWith(SpringExtension.class)
@SpringBootTest
@ContextConfiguration(
        classes = {  ConfigA.class,ConfigB.class }
)
public abstract class Testing {
    @Autowired
    ApplicationContext applicationContext;
}
```
```java
public class SomeIntegrationTest1  extends Testing  {
    @Test
    void someTest1(){
        System.out.println(applicationContext);
   }
}
```
```java
public class SomeIntegrationTest2  extends  Testing  {
    @Test
    void someTest2(){
        System.out.println(applicationContext);
    }
}

```
### 실행!!🏍   
  ![image](https://user-images.githubusercontent.com/67067346/159113804-5dccfa84-87d4-4f19-b564-cd21aaaea976.png)

  ![image](https://user-images.githubusercontent.com/67067346/159113835-0a9caddb-6ee2-474c-a01d-bf812c37a7e4.png)
    

  오 !! Applicationcontext가 Caching된다   
  이런식으로 더 빠른 검증이 가능하다! 
  물론 @ContextConfiguration에 설정이 필요하다 (이에 관한 내용은 위의 링크에 자세히 나와 있다)

  ## 마치면서 ... 
  단순히 Test Code "만"작성하는게 아니라 그 후에 벌어질 상황까지 고려하여 코드를 작성하면 더 좋겠다 ~라고  생각하면서 마친다 !   
  부족한 글을 읽어주셔서 감사합니다. 잘못된 부분은 언제든지 지적바랍니다!!   
  🌝🌝🌝
 
