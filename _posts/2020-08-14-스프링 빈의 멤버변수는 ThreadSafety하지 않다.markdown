---
layout: post
title:  "스프링 빈의 멤버변수는 ThreadSafety하지 않다."
date:   2020-08-14 03:18:23 +0700
categories: [spring]

---

# 스프링 빈의 멤버변수는 ThreadSafety하지 않다.

스프링을 사용하면 많은 부분을 스프링 프레임워크과 라이브러리에 의존하고 강하게 믿는다. 또한 왜인지 스프링이 모두 해결해줄 것 같은 느낌을 받는 듯하다.  

특히 빈으로 생성된 객체는 ThreadSafety하다는 착각을 하게되다보니 가끔 실수로 이런 코드를 생성한다.

> 파일을 관리하는 클래스

```java
@Component
public class FileManager {
	private String fileName; 
	private String filePath;
	
	... getter/setter 생략
}
```

```java
@Service
public class FileService {
	
	@Autowired FileManger fileManager;
	
	... 파일 관련 메소드

}
```

운영서버에서는 클라이언트에서 파일을 저장했음에도 간헐적으로 파일이 저장이 되지않았으며 다른 폴더로 파일이 발견되는 이슈가 발생하였다.

결국 문제를 해결 못한 듯한 이전 개발자는파일 경로를 수정하는 메소드를 삭제하였고 문제를 해결된 듯했다.



### 문제 발생 원인

- 빈이 ThreadSafety 하다는 믿음에서 문제가 발생한다.
- 테스트 환경이 단일 로컬 환경에서만 진행되어 문제를 발견하기 힘들었다.



#### 해결방안

- 멤버 변수 대신 메소드 내에서 변수로 생성한다. 다음과 같이 fileName, filePath 같은 경우에는 클래스를 새로 생성하여 관리하는 것이 좋다고 판단
- 스프링 빈의 scope를 **prototype** 으로 지정하는 방법이 있으나 매번 새로운 객체를 생성하기 때문에 성능이슈가 발생할 수 있다.





참고 블로그 

[https://beyondj2ee.wordpress.com/2013/02/28/%EB%A9%80%ED%8B%B0-%EC%93%B0%EB%A0%88%EB%93%9C-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B9%88-%EC%A3%BC%EC%9D%98%EC%82%AC%ED%95%AD/](https://beyondj2ee.wordpress.com/2013/02/28/멀티-쓰레드-환경에서-스프링빈-주의사항/)

