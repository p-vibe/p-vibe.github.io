---
layout: post
title:  자바 ORM 표준 JPA 프로그래밍 정리
date:   2020-08-23 01:10:00 +0800
categories: 책/강의
tag: JPA
sitemap :
  changefreq : daily
  priority : 1.0
---

이 포스트에서는 김영한님의 [자바 orm 표준 jpa 프로그래밍](https://book.naver.com/bookdb/book_detail.nhn?bid=9252528)을 정리합니다. 정리하는대로 조금씩 업데이트할 예정입니다.

## 1. JPA 소개

JDBC API를 활용하면 애플리케이션의 비즈니스 로직보다 SQL과 JDBC API를 작성하는 데 더 많은 시간을 보내기도 한다. JdbcTemplate 같은 SQL 매퍼를 사용하면서 JDBC API 사용 코드를 많이 줄일 수 있지만, 여전히 등록, 수정, 삭제, 조회용 SQL은 반복해서 작성해야 했다. 객체 지향의 장점을 살리기 위해 객체 모델링을 세밀하게 진행할수록 객체를 데이터베이스에 저장하거나 조회하기는 점점 더 어려워지고 객체와 관계형 데이터베이스의 차이를 메우기 위해 더 많은 SQL을 작성해야 한다.

ORM은 객체와 관계형 데이터베이스 간의 차이를 중간에서 해결해준다. JPA는 지루하고 반복적인 CRUD SQL을 알아서 처리해줄 뿐만 아니라 객체 모델링과 관계형 데이터베이스 사이의 차이점도 해결해준다. 그리고 JPA는 실행 시점에 자동으로 SQL을 만들어서 실행하는데, JPA를 사용하는 개발자는 SQL을 직접 작성하는 것이 아니라 어떤 SQL이 실행될지 생각만 하면 된다.  JPA를 사용하면 CRUD SQL을 작성할 필요가 없고, 조회된 결과를 객체로 매핑하는 작업도 대부분 자동으로 처리해주므로 데이터 저장 계층에 작성해야 할 코드가 많이 준다. 또한 애플리케이션을 SQL이 아닌 객체 중심으로 개발하니 생산성과 유지보수가 확연히 좋아지고 테스트를 작성하기도 편리해진다. mysql에서 오라클로의 db 변경에도 수월하게 대응할 수 있다.

### 1.1 SQL을 직접 다룰 때 발생하는 문제점

- 진정한 의미의 계층 분할이 어렵다.
- 엔티티를 신뢰할 수 없다.
- SQL에 의존적인 개발을 피하기 어렵다.

데이터베이스에 데이터를 관리하려면 SQL을 사용해야 한다. 데이터베이스는 객체 구조와는 다른 데이터 중심의 구조를 가지므로 객체를 데이터베이스에 직접 저장하거나 조회할 수는 없다. 따라서 개발자가 객체지향 애플리케이션과 데이터베이스 중간에서 SQL과 JDBC API를 사용해서 변환 작업을 직접 해주어야 한다. 문제는 객체를 데이터베이스에 CRUD하려면 너무 많은 SQL과 JDBC API를 코드로 작성해야 한다는 점이다.

**SQL에 의존적인 개발**

요구사항이 변경되어 코드를 수정해야할 때, 자바 컬렉션에 객체를 보관할 땐 필드를 추가한다고 해서 많은 코드를 수정할 필요가 없지만 데이터베이스에 저장할 때는 많은 코드를 수정해야할 수도 있다. 또한 버그가 있을 때 SQL을 확인해야 원인을 알 수 있는 경우가 있다. 즉 객체의 기능이 잘 돌아갈지는 전적으로 SQL에 달려있고, 데이터 접근 계층을 사용해서 SQL을 숨겨도 어쩔 수 없이 어떤 SQL이 실행되는지 확인해야 한다. 

비즈니스 요구사항을 모델링한 객체를 엔티티라 하는데, SQL에 모든 것을 의존하는 상황에서는 개발자들이 엔티티를 신뢰하고 사용할 수 없다. 대신 DAO를 열어서 어떤 SQL이 실행되고 어떤 객체들이 함께 조회되는지 일일이 확인해야 한다. 이것은 진정한 의미의 계층 분할이 아니다. 물리적으로 SQL과 JDBC API를 데이터 접근 계층에 숨기는 데 성공했을지는 몰라도 논리적으로는 엔티티와 아주 강한 의존관계를 가지고 있다. 이런 강한 의존관계 때문에 회원을 조회할 때는 물론이고 회원 객체에 필드를 하나 추가할 때도 DAO의 CRUD 코드와 SQL 대부분을 변경해야 하는 문제가 생긴다.

**JPA를 활용한 문제 해결**

JPA를 사용하면 객체를 데이터베이스에 저장하고 관리할 때, 개발자가 직접 SQL을 작성하는 것이 아니라 JPA가 제공하는 API를 사용하면 된다. 그러면 JPA가 개발자 대신 적절한 SQL을 생성해서 데이터베이스에 전달한다. 또한 JPA는 SQL을 개발자 대신 작성해서 실행해주는 것 이상의 기능들을 제공한다.

### 1.2 패러다임 불일치

객체를 메모리가 아닌 어딘가에 영구 보관해야할 때 문제가 발생한다. 객체가 단순하면 객체의 모든 속성 값을 꺼내서 파일이나 데이터베이스에 저장하면 되지만 부모 객체를 상속받았거나 다른 객체를 참조하고 있다면 객체의 상태를 저장하기는 쉽지 않다. 자바는 이런 문제를 고려하여 객체를 파일로 저장하는 직렬화 기능과 저장된 파일을 객체로 복구하는 역직렬화 기능을 지원하지만, 이 방법은 직렬화된 객체를 검색하기 어렵다는 문제가 있으므로 현실성이 없다. 현실적인 대안은 관계형 데이터베이스에 객체를 저장하는 것인데, 관계형 데이터베이스는 데이터 중심으로 구조화되어 있고, 객체지향에서 얘기하는 추상화, 상속, 다형성 같은 개념이 없다.

객체와 관계형 데이터베이스는 지향하는 목적이 서로 다르므로 둘의 기능과 표현 방법도 다르다. 이것을 패러다임 불일치 문제라 한다. 객체 구조를 테이블 구조로 저장하는 데는 한계가 있다. 자바라는 객체지향 언어로 개발하고 데이터는 관계형 데이터베이스에 저장해야 한다면, 패러다임 불일치 문제를 개발자가 중간에서 해결해야 한다.

**상속**

객체는 상속이라는 기능을 가지지만 테이블은 상속 기능이 없다. 상속 구조를 JDBC API를 사용해서 완성하려면 작성해야 할 코드량이 만만치 않다. JPA는 상속과 관련된 패러다임의 불일치 문제를 개발자 대신 해결해준다.

**연관관계**

객체는 참조를 사용해서 다른 객체와 연관관계를 가지고 참조에 접근해서 연관된 객체를 조회한다. 반면 테이블은 외래 키를 사용해서 다른 테이블과 연관관계를 가지고 조인을 사용해서 연관된 테이블을 조회한다. 참조를 사용하는 객체와 외래 키를 사용하는 관계형 데이터베이스 사이의 패러다임 불일치는 객체지향 모델링을 거의 포기하게 만들 정도로 극복하기 어렵다(조회 방식이 다른 점, 객체는 참조가 있는 방향으로만 조회할 수 있지만 테이블은 외래 키 하나로도 가능한 차이 등으로 인해).

그래서 객체를 테이블에 맞춰 모델링하곤 한다. 그러면 객체를 테이블에 저장하거나 조회할 때는 편리하지만, 객체가 참조를 통해 객체를 찾으려면 외래키가 아닌 객체 참조를 해야 한다는 점 등 때문이다. 객체가 객체 참조 대신 외래키를 통해 다른 객체와 관계를 맺다보면 좋은 객체 모델링을 기대하기 어렵고 객체지향의 특징을 잃어버리게 된다. 그런데 객체 참조를 활용한 객체지향 모델링을 사용하면 객체를 테이블에 저장하거나 조회하기가 쉽지 않다. 진퇴양난이다. 결국 개발자가 중간에서 객체 참조와 외래 키를 변환해주는 역할을 해줘야 한다.

JPA를 활용하면 개발자는 객체 간 관계를 설정하고 한 객체만 저장하면 된다. 객체를 조회할 때 외래 키를 참조로 변환하는 일도 JPA가 처리해준다.

위 문제들은 SQL을 직접 다뤄도 열심히 하기만 하면 어느정도 극복할 수 있는 문제들이지만, 아래 내용은 연관관계와 관련해서 극복하기 어려운 패러다임의 불일치 문제들이다. 

**객체 그래프 탐색**

한 객체에서 연관된 객체를 조회할 때는 참조를 사용하는데, 이것을 객체 그래프 탐색이라 한다. 객체는 마음껏 객체 그래프를 탐색할 수 있어야 한다. SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지 정해진다. 비즈니스 로직에 따라 사용하는 객체 그래프가 다른데 언제 끊어질지 모를 객체 그래프를 함부로 탐색할 수 없다는 건 큰 문제다. 연관된 객체를 탐색할 수 있을지 없을지는 자바 코드만 보고서는 전혀 예측할 수 없다. 결국 데이터 접근 계층인 DAO를 열어 SQL을 직접 확인해야 한다. 이 문제는 근본적으로 엔티티가 SQL에 논리적으로 종속되어서 발생하는 문제다. 그렇다고 연관된 모든 객체 그래프를 데이터베이스에서 조회하여 애플리케이션 메모리에 올려두는 것은 현실성이 없다. Dao로 객체를 조회할 때 상황에 따라 여러 조회 함수를 만들어 사용해야 하기 때문이다.

JPA를 활용하면 객체를 사용하는 싲머에 적절한 SQL을 실행하기 때문에 연관된 객체를 신뢰하고 마음껏 조회할 수 있다. 이걸 실제 객체를 사용하는 시점까지 데이터베이스 조회를 미룬다고 해서 지연로딩이라고 한다.

**비교**

데이터베이스는 기본 키의 값으로 각 로우를 구분한다. 반면 객체는 동일성 비교(==)와 동등성 비교(equals())라는 두 가지 비교 방법이 있다. 따라서 테이블의 로우를 구분하는 방법과 객체를 구분하는 방법에는 차이가 있다. SQL을 직접 사용할 때 데이터베이스의 같은 로우를 조회해도 객체의 동일성 비교에는 실패한다. 이런 패러다임 불일치 문제를 해결하기 위해 데이터베이스의 같은 로우를 조회할 때마다 같은 인스턴스를 반환하도록 구현하는 것은 쉽지 않다. 여기에 여러 트랜잭션이 동시에 실행되는 상황까지 고려하면 더 어려워진다.

JPA는 같은 트랜잭션일 때 같은 객체가 조회되는 것을 보장한다(분산 환경이나 트랜잭션이 다른 상황까지 고려하면 더 복잡해지긴 한다).

### 1.3 JPA란 무엇인가?

ORM은 이름 그대로 객체와 관계형 데이터베이스를 매핑해준다는 뜻이다. ORM 프레임워크는 단순히 SQL을 개발자 대신 생성해서 데이터베이스에 전달해주는 것뿐만 아니라 앞서 얘기한 다양한 패러다임 불일치 문제들도 해결해준다. 따라서 객체 측면에서는 정교한 객체 모델링을 할 수 있고 관계형 데이터베이스는 데이터베이스에 맞도록 모델링하면 된다.

jpa의 탄생 과정으로 자바 빈즈라는 기술 표준 내 엔티티 빈이라는 ORM 기술 -> 하이버네이트 -> JPA로 생각하면 된다. JPA는 자바 ORM 기술에 대한 API 표준 명세다. 즉 인터페이스를 모아둔 것이다. JPA라는 표준 덕분에 특정 구현 기술에 대한 의존도를 줄일 수 있고 다른 구현 기술로 손쉽게 이동할 수 있다는 장점이 있다.

**왜 JPA를 사용해야 하는가?**

지루하고 반복적인 코드와 CRUD용 SQL을 개발자가 직접 작성하지 않아도 된다. 나아가 CREATE TABLE 같은 DDL 문을 자동으로 생성해준다. 이런 기능들을 활용하여 데이터베이스 설계 중심의 패러다임을 객체 설계 중심으로 역전시킬 수 있다.

JPA를 사용하면 필드를 추가하거나 삭제해도 수정해야 할 코드가 줄어든다. 즉 유지보수해야 하는 코드 수가 줄어든다. 게다가 객체지향 언어가 가진 장점들을 활용해서 유연하고 유지보수하기 좋은 도메인 모델을 편리하게 설계할 수 있다(유지보수). 또한 JPA를 사용하면 패러다임 불일치 문제를 해결할 수 있다(패러다임 불일치 해결). 그리고 JPA는 애플리케이션과 데이터베이스 사이에서 다양한 성능 최적화 기회를 제공한다(성능). 또한 JPA는 애플리케이션과 데이터베이스 사이에 추상화된 데이터 접근 계층을 사용해서 애플리케이션이 특정 데이터베이스 기술에 종속되지 않도록 한다(데이터베이스 접근 추상화와 벤더 독립성). 끝으로 JPA라는 표준을 사용하면 다른 구현 기술로 손쉽게 변경할 수 있다(표준).



## 2. JPA 시작

### 2.1 객체 매핑

JPA는 매핑 어노테이션을 분석해서 어떤 객체가 어떤 테이블과 관계가 있는지 알아낸다. 

- @Entity: 이 클래스를 테이블과 매핑한다고 JPA에게 알려준다.
- @Table: 엔티티 클래스에 매핑할 테이블 정보를 알려준다. 이 어노테이션을 생략하면 클래스 이름을 테이블 이름으로 매핑한다.
- @Id: 엔티티 클래스의 필드를 테이블의 기본 키에 매핑한다.
- @Column: 필드를 칼럼에 매핑한다. 
- 매핑 정보가 없는 필드: 매핑 어노테이션을 생략하면 필드명을 사용해서 컬럼명으로 매핑한다.

### 2.2 persistence.xml

이름이 javax.persistence로 시작하는 속성은 JPA 표준 속성으로 특정 구현체에 종속되지 않는다. 반면에 hibernate로 시작하는 속성은 하이버네이트 전용 속성이므로 하이버네이트에서만 사용할 수 있다.

**데이터베이스 방언**

SQL 표준을 지키지 않거나 특정 데이터베이스 만의 고유한 기능을 JPA에서는 방언이라고 한다. 특정 데이터베이스에 종속되는 기능을 많이 사용하면 나중에 데이터베이스를 교체하기가 어렵다. 하이버네이트를 포함한 대부분의 JPA 구현체들은 이런 문제를 해결하려고 다양한 데이터베이스 방언 클래스를 제공한다. 개발자는 JPA가 제공하는 표준 문법에 맞춰 JPA를 사용하면 되고 특정 데이터베이스에 의존적인 SQL은 데이터베이스 방언이 처리해준다. 따라서 데이터베이스가 변경되어도 애플리케이션 코드를 변경할 필요 없이 데이터베이스 방언만 교체하면 된다.

### 2.3 애플리케이션 개발

#### 2.3.1 엔티티 매니저 설정

1. 엔티티 매니저 팩토리 생성 

   JPA를 시작하려면 우선 persistence.xml의 설정 정보를 사용해서 엔티티 매니저 팩토리를 생성해야 한다. 엔티티 매니저 팩토리는 애플리케이션 전체에서 딱 한 번만 생성하고 공유해서 사용해야 한다.

2. 엔티티 매니저 생성

   JPA의 기능 대부분은 엔티티 매니저가 제공한다. 엔티티 매니저를 사용해서 엔티티를 데이터베이스에 등록/수정/삭제/조회할 수 있다. 엔티티 매니저는 내부에 데이터소스(데이터베이스 커넥션)를 유지하면서 데이터베이스와 통신한다. 따라서 개발자는 엔티티 매니저를 가상의 데이터베이스로 생각할 수 있다.

   참고로 엔티티 매니저는 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드 간에 공유하거나 재사용하면 안된다.

3. 종료

   사용이 끝난 엔티티 매니저는 반드시 종료해야 한다. 애플리케이션을 종료할 때 엔티티 매니저 팩토리도 종료해야 한다.

### 2.3.2 트랜잭션 관리

JPA를 사용하면 항상 트랜잭션 안에서 데이터를 변경해야 한다. 트랜잭션 없이 데이터를 변경하면 예외가 발생한다. 

### 2.3.3 JPQL

JPA를 사용하면 애플리케이션 개발자는 엔티티 객체를 중심으로 개발하고 데이터베이스에 대한 처리는 JPA에 맡겨야 한다. 등록, 수정, 삭제, 한 건 조회는 문제 없지만 문제는 검색 쿼리다. JPA는 엔티티 객체를 중심으로 개발하므로 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색해야 한다. 그런데 테이블이 아닌 엔티티 객체를 대상으로 검색하려면 데이터베이스의 모든 데이터를 애플리케이션으로 불러와서 엔티티 객체로 변경한 다음 검색해야 하는데, 이는 사실상 불가능하다. 애플리케이션이 필요한 데이터만 데이터베이스에서 불러오려면 결국 검색 조건이 포함된 SQL을 사용해야 한다. JPA는 JPQL이라는 쿼리 언어로 이 문제를 해결한다. JPA는 SQL을 추상화한 JPQL이라는 객체지향 쿼리 언어를 제공한다. JPQL과 SQL의 차이는 다음과 같다.

1. JPQL은 엔티티 객체를 대상으로 쿼리한다. 즉 클래스와 필드를 대상으로 쿼리한다. JPQL은 데이터베이스 테이블을 전혀 알지 못한다.
2. SQL은 데이터베이스 테이블을 대상으로 쿼리한다.

## 3. 영속성 관리

JPA가 제공하는 기능은 크게 엔티티와 테이블을 매핑하는 설계 부분과 매핑한 엔티티를 실제 사용하는 부분으로 나눌 수 있다.

엔티티 매니저는 엔티티를 저장하고, 수정하고, 삭제하고, 조회하는 등 엔티티와 관련된 모든 일을 처리한다.

### 3.1 엔티티 매니저 팩토리와 엔티티 매니저

데이터베이스를 하나만 사용하는 애플리케이션은 일반적으로 엔티티 매니저 팩토리를 하나만 생성한다. 그리고 필요할 때마다 엔티티 매니저 팩토리에서 엔티티 매니저를 생성하면 된다.

엔티티 매니저 ㅍ팩토리는 여러 스레드가 동시에 접근해도 안전하므로 서로 다른 스레드 간에 공유해도 되지만, 엔티티 매니저는 여러 스레드가 동시에 접근하면 동시성 문제가 발생하므로 스레드 간에 절대 공유하면 안 된다.

엔티티 매니저는 데이터베이스 연결이 꼭 필요한 시점까지 커넥션을 얻지 않는다. 예를 들어 트랜잭션을 시작할 때 커넥션을 획득한다. 

### 3.2 영속성 컨텍스트란?

영속성 컨텍스트를 해석하면 엔티티를 영구 저장하는 환경이라는 뜻이다. 엔티티 매니저로 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관하고 관리한다. 영속성 컨텍스트는 엔티티 매니저를 생성할 때 하나 만들어진다. 엔티티 매니저를 통해서 영속성 컨텍스트에 접근할 수 있고, 영속성 컨텍스트를 관리할 수 있다.

### 3.3 엔티티의 생명주기

- 비영속: 영속성 컨텍스트와 전혀 관계가 없는 상태
- 영속: 영속성 컨텍스트에 저장된 상태. 영속 상태라는 것은 결국 영속성 컨텍스트에 의해 관리된다는 뜻이다.
- 준영속: 영속성 컨텍스트에 저장되었다가 분리된 상태. 영속성 컨텍스트가 관리하던 영속 상태의 엔티티를 영속성 컨텍스트가 관리하지 않으면 준영속 상태가 된다.
- 삭제: 삭제된 상태

### 3.4 영속성 컨텍스트의 특징

- 영속성 컨텍스트와 식별자 값: 영속성 컨텍스트는 엔티티를 식별자 값으로 구분한다. 따라서 영속 상태는 식별자 값이 반드시 있어야 한다. 
- 영속성 컨텍스트와 데이터베이스 저장: JPA는 보통 트랜잭션을 커밋하는 순간 영속성 컨텍스트에 새로 저장된 엔티티를 데이터베이스에 반영하는데 이것을 플러시라 한다.
- 영속성 컨텍스트가 엔티티를 관리하면 다음과 같은 장점이 있다.
  - 1차 캐시
  - 동일성 보장
  - 트랜잭션을 지원하는 쓰기 지연
  - 변경 감지
  - 지연 로딩

#### 3.4.1 엔티티 조회

영속성 컨텍스트는 내부에 캐시를 가지고 있는데 이것을 1차 캐시라 한다. 영속성 컨테스트 내부에 Map이 하나 있는데 키는 @Id로 매핑한 식별자고 값은 엔티티 인스턴스다. 영속성 컨텍스트에 데이터를 저장하고 조회하는 모든 기준은 데이터베이스 기본 키 값이다.

**1차 캐시에서 조회**

em.find()를 호출하면 먼저 1차 캐시에서 엔티티를 찾고 만약 찾는 엔티티가 1차 캐시에 없으면 데이터베이스에서 조회한다. 1차 캐시를 통해 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공한다는 장점이 있다.

**데이터베이스에서 조회**

만약 em.find()를 호출했는데 엔티티가 1차 캐시에 없으면 엔티티 매니저는 데이터베이스를 조회해서 엔티티를 생성한다. 그리고 1차 캐시에 저장한 후에 영속 상태의 엔티티를 반환한다.

**영속 엔티티의 동일성 보장**

영속성 컨텍스트는 성능상 이점 뿐 아니라 엔티티의 동일성을 보장한다.

#### 3.4.2 엔티티 등록

엔티티 매니저는 트랜잭션을 커밋하기 직전까지 데이터베이스에 엔티티를 저장하지 않고 내부 쿼리 저장소에 INSERT SQL을 차곡차곡 모아둔다. 그리고 트랜잭션을 커밋할 때 모아둔 쿼리를 데이터베이스에 보낸다. 이를 트랜잭션을 지원하는 쓰기 지연이라 한다.

예시를 보면, 영속성 컨텍스트는 1차 캐시에 회원 엔티티를 저장하면서 동시에 회원 엔티티 정보로 등록 쿼리를 만든다. 그리고 만들어진 등록 쿼리를 쓰기 지연 SQL 저장소에 보관한다. 트랜잭션을 커밋하면 엔티티 매니저는 우선 영속성 컨텍스트를 플러시한다. 플러시는 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 작업인데 이때 등록, 수정, 삭제한 엔티티를 데이터베이스에 반영한다. 좀 더 구체적으로 이야기하면 쓰기 지연 SQL 저장소에 모인 쿼리를 데이터베이스에 보낸다. 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화한 후에 실제 데이터베이스 트랜잭션을 커밋한다.

어떻게든 커밋 직전에만 데이터베이스에 SQL을 전달하면 된다. 이것이 트랜잭션을 지원하는 쓰기 지연이 가능한 이유다. 이 기능을 잘 활용하면 모아둔 등록 쿼리를 데이터베이스에 한 번에 전달해서 성능을 최적화할 수 있다.

#### 3.4.3 엔티티 수정

JPA를 사용하지 않고 SQL을 사용하면 수정 쿼리를 직접 작성해야 한다. 요구사항이 늘어나면서 수정 쿼리도 추가된다. 이런 개발 방식의 문ㄴ제점은 수정 쿼리가 많아지는 것은 물론이고 비즈니스 로직을 분석하기 위해 SQL을 계속 확인해야 한다. 결국 직접적이든 간접적이든 로직이 SQL에 의존하게 된다.

**변경 감지**

JPA로 엔티티를 수정할 때는 단순히 엔티티를 조회해서 데이터만 변경하면 된다. 엔티티의 변경사항을 데이터베이스에 자동으로 반영하는 기능을 변경 감지라 한다.

JPA는 엔티티를 영속성 컨텍스트에 보관할 때, 최초 상태를 복사해서 저장해두는데 이것을 스냅샷이라 한다. 그리고 플러시 시점에 스냅샷과 엔티티를 비교해서 변경된 엔티티를 찾는다. 변경 감지는 영속성 컨텍스트가 관리하는 영속 상태의 엔티티에만 적용된다.

변경 감지로 인해 실행된 UPDATE SQL을 분석해보면, JPA의 기본 전략은 엔티티의 모든 필드를 업데이트하는 것이다. 이렇게 모든 필드를 사용하면 데이터베이스에 보내는 데이터 전송량이 증가하는 단점이 있지만, 다음과 같은 장점이 있다.

- 모든 필드를 사용하면 수정 쿼리가 항상 같다. 즉 애플리케이션 로딩 시점에 수정 쿼리를 미리 생성해두고 재사용할 수 있다.
- 데이터베이스에 동일한 쿼리를 보내면 데이터베이스는 이전에 한 번 파싱된 쿼리를 재사용할 수 있다.
- 필드가 많거나 저장되는 내용이 너무 크면 수정된 데이터만 사용해서 동적으로 UPDATE SQL을 생성하는 전략을 선택하면 된다. 단 이때는 하이버네이트 확장 기능을 사용해야 한다. @org.hibernate.annotations.DynamicUpdate 어노테이션을 사용하면 수정된 데이터만 사용해서 동적으로 UPDATE SQL을 생성한다. @DynamicInsert도 있다.

#### 3.4.4 엔티티 삭제

엔티티를 삭제하려면 먼저 삭제 대상 엔티티를 조회해야 한다. em.remove()에 삭제 대상 엔티티를 넘겨주면 엔티티를 삭제한다. 즉시 삭제하는 것은 아니고 삭제 쿼리를 쓰기 지연 SQL 저장소에 등록한다. 이후 트랜잭션을 커밋해서 플러시를 호출하면 실제 데이터베이스에 삭제 쿼리를 전달한다. em.remove()를 호출하는 순간 해당 엔티티는 영속성 컨텍스트에서 제거된다. 이렇게 삭제된 엔티티는 재사용하지 말고 자연스럽게 가비지 컬렉션의 대상이 되도록 두는 것이 좋다.

### 3.5 플러시

플러시는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다.

1. 변경 감지도 동작해서 영속성 컨텍스트에 있는 모든 엔티티를 스냅샷과 비교해서 수정된 엔티티를 찾는다. 수정된 엔티티는 수정 쿼리를 만들어 쓰기 지연 SQL 저장소에 등록한다.
2. 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송한다.

영속성 컨텍스트를 플러시하는 방법은 3가지다.

1. em.flush()를 직접 호출
2. 트랜잭션 커밋 시 플러시가 자동 호출됨
3. JPQL 쿼리 실행 시 플러시가 자동 호출됨.

### 3.6 준영속

준영속 상태의 엔티티는 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다. 영속 상태의 엔티티를 준영속 상태로 만드는 방법은 다음과 같다.

1. em.detatch(entity): 특정 엔티티만 준영속 상태로 전환. 이 메소드를 호출하는 순간 1차 캐시부터 쓰기 지연 SQL 저장소까지 해당 엔티티를 관리하기 위한 모든 정보가 제거된다.
2. em.clear(): 영속성 컨텍스트를 완전히 초기화해서 해당 영속성 컨텍스트의 모든 엔티티를 준영속 상태로 만든다.
3. em.close(): 영속성 컨텍스트를 종료. 해당 영속성 컨텍스트가 관리하던 영속 상태의 엔티티가 모두 준영속 상태가 된다. 영속 상태의 엔티티는 주로 영속성 컨텍스트가 종료되면서 준영속 상태가 된다.

**준영속 상태의 특징**

1. 거의 비영속에 가깝다. 영속성 컨텍스트가 제공하는 어떤 기능도 동작하지 않는다.
2. 식별자 값을 가지고 있다. 이미 한 번 영속 상태였기 때문이다.
3. 지연 로딩을 할 수 없다.

**병합**

준영속 상태의 엔티티를 다시 영속 상태로 변경하려면 병합을 사용하면 된다. merge() 메소드는 준영속 상태의 엔티티를 받아서 그 정보로 새로운 영속 상태의 엔티티를 반환한다. 파라미터로 넘어온 엔티티는 병합 후에도 준영속 상태로 남아 있다. 병합은 파라미터로 넘어온 엔티티의 식별자 값으로 영속성 컨텍스트를 조회하고 찾는 엔티티가 없으면 데이터베이스에서 조회한다. 만약 데이터베이스에서도 발견하지 못하면 새로운 엔티티를 생성해서 병합한다. 병합은 준영속, 비영속을 신경쓰지 않는다. 식별자 값으로 엔티티를 조회할 수 있으면 불러서 병합하고 조회할 수 없으면 새로 생성해서 병합한다. 즉 병합은 save or update 기능을 수행한다.
