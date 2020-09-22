---
layout: post
title: "왜 int 대신 Integer를 사용했을까"
date:   2020-09-21 00:18:23 +0700
categories: [spring]
---

# 왜 int 대신 Integer를 사용했을까?

실무 코드를 보면(마이바티스를 사용한 프로젝트) int 대신 Integer를 사용한 경우가 많다. 실제로 관련된 책에서는 int 대신 Integer 사용을 권고했는데 실상 이유에 대해서는 자세히 밝히지 않아 내용을 찾아보았다.



- 테이블

  컬럼과 데이터는 다음과 같이 구성하였다. 

  ```java
  CREATE TABLE `tb_test` (
  	`SEQ` INT(11) NOT NULL AUTO_INCREMENT 
  	`TITLE` INT(11) NULL DEFAULT NULL 
  	`CONTENTS` VARCHAR(6) NULL DEFAULT NULL COMMENT
  	PRIMARY KEY (`SEQ`) USING BTREE
  )
  COLLATE='utf8_general_ci'
  ENGINE=InnoDB
  AUTO_INCREMENT=1;
  
  INSERT INTO `tb_test` (`SEQ`, `TITLE`, `CONTENTS`) VALUES (1, NULL, NULL);
  ```

- XML은 다음과 같이 세팅하였다.

  ```xml
  <select id="countTestBySeq" resultType="int">
  	SELECT TITLE FROM TB_TEST WHERE SEQ = #{seq}
  </select>
  ```

  - 넘겨주는 인자 값은 1로 고정하였고 TITLE은 NULL을 허용하여 마이바티스 프레임워크를 통해 NULL을 리턴받았다

- 테스트 코드

  ```java
  @Test
  public void mybatisIntegerNull() {
      Integer IntegerCnt = testService.getCountTest();
      log.info("Integer count {}", IntegerCnt);
      assertNull(IntegerCnt);
  }
  
  @Test(expected = NullPointerException.class)
  public void mybatisIntNull() {
  	int intCnt = testService.getCountTest();
  }
  ```

  - Integer로 받을 경우 null을 확인할 수 있다.
  - int로 받을 경우 NullPointerException이 발생한다( int는 null을 받을 수 있다.)



### 결론

- 테이블 컬럼이 null을 허용한 숫자 타입의 경우 int로 받을 때 null Point Exception이 발생
  - 이런 문제를 해결하기 위해 primitive type의 Wrapper 클래스인 Integer를 사용한다.
  - 당장 null Point Exception은 해결하더라도 null 체크 코드가 들어가야한다.
  - 이런 케이스의 경우 DB 상에서 NVL이나 IFNULL을 통해 해결하는 것이 좋을 것으로 판단
- count는 값이 없을 경우 null이 아닌 0을 리턴함으로 이 경우는 제외

