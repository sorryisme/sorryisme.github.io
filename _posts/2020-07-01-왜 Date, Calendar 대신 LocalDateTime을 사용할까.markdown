---
layout: post
title:  "왜 Date, Calendar 대신 LocalDateTime을 사용할까"
date:   2020-07-01 00:18:23 +0700
categories: [java]
---

# 왜 Date, Calendar 대신 LocalDateTime을 사용할까

자바 개발 초기부터 자주 사용해왔던 것이 Date와 Calendar인데 사실 이 클래스는 굉장히 위험한 문제를 가지고 있다고 한다. (실무에서 많이 경험하는 사이드 이펙트)너무 당연하게 Date와 SimpleDate Format을 사용해왔던 터라 이런 문제를 가지고 있는지 인지하지 못했다.

많은 개발자들이 Date,Calendar 대신 LocalDateTime, LocalDate를 사용하고 있어서 알아보았다.



## not imuutable 문제

네이버 기술블로그 [Java의 날짜와 시간 API](https://d2.naver.com/helloworld/645609)에서 다음과 같이 기술한다.

> 불행히도 Java의 기본 날짜, 시간 클래스는 불변 객체가 아니다. 앞의 코드에서 Calendar 클래스에 set 메서드를 호출해서 날짜를 지정하고, 다시 같은 객체에 set(int,int) 메서드를 호출해서 수행한 날짜 연산 결과는 같은 인스턴스에 저장되었다. Date 클래스에도 값을 바꿀 수 있는 set 메서드가 존재한다. 이 때문에 Calendar 객체나 Date 객체가 여러 객체에서 공유되면 **한 곳에서 바꾼 값이 다른 곳에 영향**을 미치는 부작용이 생길 수 있다. 『Effective Java 2nd Edition』(2008)의 저자 Joshua Bloch도 Date 클래스는 불변 객체여야 했다고 지적했다.[18]

Date의 setter은 deprecated 되었지만 (API 문서에서는 not immutable 문제때문에 deprecated 된 것이 아니라 calendar.set으로 replaced되었다고 나와있다.)  실제로 다음과 같은 코드에서 처럼 setter를 사용 시 실제 객체 값이 변동되는 것을 확인할 수 있었다.

{% highlight java %}
Date date = new Date();
Member member = new Member();
member.setName("1번째 회원");
member.setJoinDate(date);

printMember(member);
Date date2 = new Date();
Member member2 = new Member();
member2.setName("2번째 회원");
member2.setJoinDate(date2);

printMember(member2);        
date.setDate(1);
System.out.println("----------------------------------------");
printMember(member);
printMember(member2);   
{% endhighlight %}


	~~~
	<<결과값>>
	가입이름 : 1번째 회원가입 일자 : Sat Jan 18 04:08:47 KST 2020
	가입이름 : 2번째 회원가입 일자 : Sat Jan 18 04:08:47 KST 2020
	---------------------------------------------
	가입이름 : 1번째 회원가입 일자 : Wed Jan 01 04:08:47 KST 2020
	가입이름 : 2번째 회원가입 일자 : Sat Jan 18 04:08:47 KST 2020
	~~~



date에서 set을 사용할 경우 member 객체에 저장된 가입일자가 변했다는 사실을 확인할 수 있다. 이런 문제 외에도 헷갈리는 월지정, int 상수필드에 대한 내용들을 문제점으로 지적하는데 이를 해결하기 위한 라이브러리가 JodaTime이다. 

## Joda-Time

```java
LocalDateTime localDateTime = new LocalDateTime();
Member member = new Member();
member.setName("1번째 회원");
member.setJoinDate(localDateTime.now());

printMember(member);
localDateTime.plusDays(1);

System.out.println("-----------------------------------------");
printMember(member);      
```



- 결과

```
가입이름 : 1번째 회원가입 일자 : 2020-01-18T04:21:55.072
---------------------------------------------
가입이름 : 1번째 회원가입 일자 : 2020-01-18T04:21:55.072
```


- 기존 Date와 비슷한 코드이지만 메소드를 호출해도 실제 member 객체 내 가입일자는 변동되지 않는다.



### 실제 내부 plusDays 메소드를 

```java
public LocalDateTime plusDays(int days) {
	if (days == 0) {
    	return this;
	}
	long instant = getChronology().days().add(getLocalMillis(), days);
    return withLocalMillis(instant);
}

LocalDateTime withLocalMillis(long newMillis) {
	return (newMillis == getLocalMillis() ? this : 
            			new LocalDateTime(newMillis, getChronology()));
}
```

실제 값을 비교해서 현재 값과 동일하거나 인자값이 0일 경우 현재 객체를 리턴하고 그 외 모든 경우 새로운 객체를 생성해서 전달하는 것을 알 수 있다. 라이브러리를 받아서 사용하면 되지만 java8 부터는 joda-Time이 아닌 java.time 에서 제공해주는 LocalDateTime을 사용하면 된다. 

참고 :

네이버 기술 블로그 : https://d2.naver.com/helloworld/645609 

자바 8 API Documentation : https://docs.oracle.com/javase/8/docs/api/

jodaTime : https://www.joda.org/joda-time/