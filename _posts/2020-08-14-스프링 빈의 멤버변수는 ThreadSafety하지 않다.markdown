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

결국 문제를 해결 못한 듯한 이전 개발자는(나는 아니다..)파일 경로를 수정하는 메소드를 삭제하였고 문제를 해결된 듯했다.