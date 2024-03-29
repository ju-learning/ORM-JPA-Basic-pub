이 글은 내용을 정리하고 개인적인 사견을 첨가한 2차 창작물이며, 강의 및 강의자료(코드 등)에 대한 저작권은 [원본](https://www.inflearn.com/course/ORM-JPA-Basic#) 및 원작자에게 있고 공유하지 않는다.

# 🗂 강의 자료

[수업 자료](https://www.notion.so/273dbd3e3d7e427bb075eb4bcd2ea344)

# 🌈 강의 환경

- H2 Database

# 📝 강의 내용 정리

## **섹션 0. 강좌 소개**

### 강좌 소개

- JPA 의 정확한 매핑
- JPA 내부 동작 방식 이해

### 수업 자료

- (수업자료 다운로드)

## 섹션 1. JPA 소개

### SQL 중심적인 개발의 문제점

- 개발할때는 객체지향 방법론을 사용, 데이터베이스는 관계형 DB 를 사용 → CRUD SQL 을 작성하는 방법론들이 지루하게 반복되고 지루함 (개발자가 SQL 매퍼 역할)
- 객체와 관계형 데이터베이스는 비슷한것같지만 패러다임 자체가 다름
- 객체는 참조를 사용하고, 테이블은 왜래키를 사용한다 (`getTeam()` vs `JOIN`)
- 객체다운 모델링(참조변수)은 SQL 을 통한 INSERT 문을 작성하기 까다로움
- 객체는 자유롭게 객체 그래프를 참조할 수 있어야하는데, SQL 은 조회되지 않으면(JOIN 하지않으면) 탐색할 수 없다
    - 계층형 아키텍쳐 에서는 넘어온 데이터를 믿고 쓸 수 있어야 하는데, 탐색이 안되면 해당 엔티티에 대한 신뢰가 없어짐
- 객체지향적으로 설계할수록 코드만 지저분해짐 → 자바 컬렉션처럼 DB를 사용할 수 없을까? 하는 고민들 → JPA 가 나옴

### JPA 소개

- JPA → Java Persistence API (자바진영의 ORM 표준)
- ORM → Object Relational Mapping (객체와 관계형을 매핑)
- 자바 컬렉션을 사용하듯 객체를 사용하면 JPA 가 여러 문제점들(매핑, 패러다임불일치) 해결해준다
- 예전에 너무 막장이었던 Enterprise JavaBeans; EJB 를 잘 만들기위해 고민하던 개발자 게빈킹이 하이버네이트를 만들어버림
- 우리는? → “JPA 표준 인터페이스에서 하이버네이트 구현체를 쓰는 중”
- Entity = JPA 가 관리하는 객체
- JPA 의 성능 최적화
    - 1차 캐시와 동일성 보장
    - 트랜잭션을 지원하는 쓰기 지연
    - 지연로딩
- JPA 는 ORM 이고 ORM 은 객체와 관계형DB의 매핑을 도와주는것이기 때문에 결국 잘 사용하려면 객체지향과 RDB를 다 잘 다루어야 함

## **섹션 2. JPA 시작하기**

### Hello JPA - 프로젝트 생성

- (H2 데이터베이스 설치)
    - 최초 접속시엔 `jdbc:h2:~/test` 로 접속, 이후에는 `jdbc:h2:tcp://localhost/~/test` 로 사용
- 실무에선 거의 Gradle 을 사용하지만, 실습환경에선 Maven 을 사용
    - 라이브러리에 사용할 hibernate 버전 확인 → [https://hibernate.org/orm/](https://hibernate.org/orm/)
    - 내게 맞는 버전 찾는법: [https://spring.io/projects/spring-boot#learn](https://spring.io/projects/spring-boot#learn) → Reference Doc. → Dependency Versions
- `hibernate.dialect` 설정을 통해 JPA 는 각각의 데이터베이스만의 특정한 “방언”을 맞춰줄 수 있음

### Hello JPA - 애플리케이션 개발

- (만든 Maven 프로젝트에서 JPA를 사용해보는 `JpaMain` 클래스 예제)

```java
public class JpaMain {
	public static void main(String[] args) {
		EntityManagerFactory emf = persistence.createEntityManagerFactory("hello");

		EntityManager em = emf.createEntityManager();

		EntityTransaction tx = em.getTransaction();

		tx.begin(); // 트랜잭션 시작
		try {
			Member findMamber = em.find(Member.class, 1L);
			findMember.setName("HelloJPA");

			tx.commit(); // 트랜잭션 커밋
		} catch (Exception e) { // 에러시
			tx.rollback(); // 트랜잭션 롤백
		} finally { em.close(); }

		emf.close();
	}
}
```

- JPA 의 모든 변경은 트랜잭션 안에서 실행해야 한다.
- 엔티티 매니저는 쓰레드간에 공유 X (사용하고 버려야함)
- [Java Persistence Query Language (JPQL)](https://en.wikibooks.org/wiki/Java_Persistence/JPQL) is the query language defined by JPA. → 객체지향 SQL
    - JPQL 은 테이블이 아닌 객체(Entity)를 대상으로 쿼리를 실행함
    - 실제 테이블에 쿼리를 날려버리면 JPA 의 구조 자체가 무너지므로, 복잡한 find 쿼리같은건 객체 지향 쿼리인 JPQL 을 많이 사용

## **섹션 3. 영속성 관리 - 내부 동작 방식**

### 영속성 컨텍스트 1

- JPA 에서 가장 중요한 2가지를 뽑으라면?
    1. 객체와 관계형 데이터베이스 매핑 (설계)
    2. 영속성 컨텍스트: 실제 데이터베이스가 내부적으로 어떻게 동작하는지, “엔티티를 영구 저장하는 환경”
- 영속성 컨텍스트
    - 영속성 컨텍스트는 논리적인 개념임 (물리x)
    - 눈에 보이지 않으며, EntityManager 를 통해 영속성컨텍스트를 접근
- 엔티티의 생명주기
    - 비영속(new/transient): 객체를 생성만 한 상태
    - 영속(managed): em(EntityManager) 를 가져와서 `em.persist( .. );` 를 해서 영속화 한 상태. 영속상태가 된다고 쿼리가 날아가는게 아님
    - 준영속
    - 삭제

### 영속성 컨텍스트 2

- 영속성 컨텍스트는 내부에 1차 캐시를 들고있음
- 영속 엔티티의 동일성을 보장해줌
- 트랜잭션을 지원하는 쓰기 지연 → 모았다가 한번에 쿼리할 수 있는 기능 (배치가 아니고 실시간에선 크게 이점은 없지만 유리함)
- 엔티티 변경 감지 (더티체킹) → 커밋을 하면 flush 를 호출하는데, 이때 기존의 1차캐시 데이터와 스냅샷을 비교하고 다르면 update query를 작성해줌

### 플러시

- 플러시: 영속성 컨텍스트의 변경내용을 데이터베이스에 반영 (동기화) 하는 작업
- 실제 코드에서 커밋전에 강제로 플러시를 호출하려면 `em.flush();` 를 호출하면 됨
- JPQL 을 사용시 JPQL 은 쿼리를 직접 생성해서 호출하기때문에, 쿼리가 나가기전 자동으로 플러시를 발생한다

### 준영속 상태

- 영속상태가 되기위해선 `em.persist` 나 `em.find` 를 호출하면 된다 (호출하면 1차캐시에 올라가기때문)
- 영속상태의 객체를 `em.detach( .. )` 를 사용하면 준영속 상태로 영속상태가 해제된다

### 정리

- 영속성 컨텍스트에 대해 알아봄
- JPA 에서 매핑과 영속성컨텍스트가 매우 중요
- 엔티티에는 생명 주기가 있음

![https://s3.us-west-2.amazonaws.com/secure.notion-static.com/0aed5f68-1cb7-4253-bad8-304669fb5f41/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230127%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230127T090454Z&X-Amz-Expires=86400&X-Amz-Signature=1584ee1cb85ebea94a9011966daf88132e568b6110674c6bca2cb003011d3102&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/0aed5f68-1cb7-4253-bad8-304669fb5f41/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230127%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230127T090454Z&X-Amz-Expires=86400&X-Amz-Signature=1584ee1cb85ebea94a9011966daf88132e568b6110674c6bca2cb003011d3102&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

- 플러시는 영속성컨텍스트를 비우지않음 (그냥 동기화임)
- 트랜잭션이라는 작업 단위가 매우 중요

## **섹션 4. 엔티티 매핑**

### 객체와 테이블 매핑

- 객체와 테이블매핑, 스키마 자동생성, 필드와 컬럼, 기본키, 연관관계 매핑에 대해 배움
- 객체와 테이블 매핑 - `@Entity`
    - 이 어노테이션이 붙으면 JPA 가 관리하는 엔티티임
    - 기본 생성자 필수

### 데이터베이스 스키마 자동 생성

- 데이터베이스의 매핑정보만 보면 어떤 쿼리를 만들어야 할지 알 수 있으므로, 로딩시점에 테이블 쿼리를 만들어줌
- 데이터베이스 방언을 활용하여 특정 DB엔진에 맞는 DDL 문을 설정해줌
- 이렇게 생성된 DDL 은 반드시 개발 환경에서만 사용

### 필드와 컬럼 매핑

- 필드의 `@Column` 속성중 `unique` 값은 랜덤값으로 배정되기때문에 보통 `@Table` 의 `uniqueConstraints` 를 많이 사용
- `@Enumerated` 를 사용할때 타입은 반드시 `ORDINAL` 이 아닌, `STRING` 을 사용 → 순서에 의존적이게 된다
- 사실상 `@Temporal` 은 옛날 자바의 `Date`, `Time` 클래스를 사용할때만 필요함

### 기본 키 매핑

- 간단히 기본키 매핑은 `@Id` 어노테이션을 사용함 → 직접 ID 를 셋팅
- 또한, `@GeneratedValue` 로 자동생성 할 수 있음
    - `IDENTITY`: 보통 MySQL 에서 사용(MySQL 의 mysql auto_increment)디비에게 자동생성을 완전 위임
    - `SEQUENCE`: 보통 오라클DB 에서 많이 사용하며, 시퀀스테이블을 사용하여 넘버링
- 권장하는 식별자 전략:
    - 키본키 제약 조건: null이면 안됨, 유일해야함, 변하면 안됨
    - 권장: Long 형 + 대체키(UUID 같은) + 키 생성전략 사용
- IDNTITY 전략의 특징
    - 데이터베이스에 실제로 나가봐야 키값을 알 수 있기때문에, 영속성컨텍스트의 1차캐시에 저장하기 위해 `em.persist` 를 호출하면 즉시 `insert` 쿼리가 나감
    - 따라서 모아서 (버퍼링해서) 한번에 쿼리가 나가는 전략을 사용할 수 없다 → 김영한님 피셜: 근데 뭐 큰 의미는 없음
- SEQUENCE 전략의 특징
    - 시퀀스 전략이면 id 값을 가져오기 위해 `em.persist` 를 호출하기 위해 다음 시퀀스를 위한 SELECT(call next value) 가 호출되고 영속성 컨텍스트에 저장됨
    - 이럴때 성능 최적화를 위해 `allocationSize` 를 사용해서 미리 시퀀스값을 미리 가져와서 사용하는 방법 → 동시성 문제도 없음

### 실전 예제 1 - 요구사항 분석과 기본 매핑

- (조금더 복잡한 예제를 통해 어떻게 설계하는지 알아보자)
- (라이브코딩으로 엔티티 설계대로 간단한 주문 Entity 들 설계)
- 데이터 중심으로 설계하면 `Member` 를 직접 호출하지 않고 `memberId` 를 가지고 찾아야하기 때문에 객체지향 스럽지 않다 → 다음시간에 연관관계 매핑을 배워보자

## **섹션 5. 연관관계 매핑 기초**

### 단방향 연관관계

- 단순히 id 를 가지고 호출하는것이 아닌, 객체 자체를 매핑해서 가져오는 방법
- (연관관계의 주인 개념이 어려움)
- 객체를 테이블에 맞춰서 `team_id` 를 직접 맵핑했을때의 문제점
    - 팀을 저장하고 회원을 저장할 때, `teamId` 를 가지고 팀을 가지고 오는게 객체지향스럽지않음
    - `team_id` 만 가지고 팀을 알 수 없으니, Member 에서 Team 을 사용하려면 계속 `em.find` 를 호출해야 함
    - 테이블은 외래키로 조인 vs 객체는 참조를 사용 → 모델링의 협력 관계를 만들 수 없음
- 단방향 연관관계는 `@ManyToOne` 과 `@JoinColumn` 을 사용하여 간단하게 가능

### 방향 연관관계와 연관관계의 주인 1- 기본

- (JPA 계의 포인터;;) 영속성컨텍스트의 메커니즘과 양방향연관관계와 연관관계 주인이 제일 어려움
- 테이블의 연관관계는 사실상 Foreign Key 하나만 가지고 양방향을 다 다룰 수 있음 but 객체는 아님
- `@OneToMany` 에서는 `mappedBy` 를 사용하여 어디에 매핑이 되어있는지 기재해준다
- 연관관계의 주인과 mappedBy
    - 객체에서의 양방향 연관관계는 사실상 단방향 연관관계가 2개 있는거임
    - 객체의 양방향에서는 어느 단방향쪽에서 업데이트를 해줘야 할지 정해야 한다 (테이블은 FK 하나로 되지만) → 둘중 하나로 주인을 정해서 관리를 해야함 = 연관관계 주인
    - 연관관계 주인에서 주인은 CRUD 가 가능, 주인이 아니면 읽기만 가능, 주인은 `mappedBy` 를 사용하지 않음, 주인이 아니면 `mappedBy` 로 주인이 누군지 써야함
    - 누구를 주인으로 해야하지? → mapped ‘By’ 니까 `mappedBy` 가 없는게 주인임, `mappedBy` 는 읽기만 가능하고 값을 넣어봐야 의미없음
    - (강사 추천) 외래키가 있는곳을 주인으로 정하는게 좋음
        - 이걸 반대로 하면 Team 을 업데이트 했는데 Member 쿼리가 나갈 수 있음
        - 성능이슈도 있음 → 뒤에서 설명
        - 1:N 에서 N쪽이 무조껀 연관관계 주인이 되는게 제일 깔끔함

### 양방향 연관관계와 연관관계의 주인 2 - 주의점, 정리

- `mappedBy` 가 붙어있으면 읽기전용이라 JPA 가 insert 할때 아무런 동작을 안씀
- 양방향 매핑시에 연관관계 주인에 반드시 값을 넣어주지 않으면 `null` 이 들어가므로 주의
- 그냥 양방향 매핑을 할때는 양쪽에 다 값을 넣어주는게 맞음
    - JPA 를 통해 불러오기 전에 `em.flush(), em.clear()` 를 해주면 문제가 안되지만, 그전에 호출해버리면 DB 에서 select 가 안나가서 데이터가 없다고 잘못 나옴
    - JPA 입장에서 보면 주인에만 값을 넣어주면 되지만, 객체지향적으로 생각하면 둘다 데이터가 있는게 맞기때문에
- 이럴때를 위해 연관관계 편의 메서드를 작성해주는게 좋다. 더 자세한 편의메서드 작성법은 책을 참고

```java
public void changeTeam(Team team) {
	this.team = team;
	team.getMembers().add(this);
}
```

```java
public void addMemeber(Member member) {
	member.setTeam = team; // setter 필요 
	this.memberList.add(this);
}
```

- member 를 기준으로 team 을 넣을지, team 을 기준으로 member 를 넣을지 편의메서드를 작성할곳을 정해주면 됨 (한쪽에만 작성해주는게 좋음) → 어플리케이션 작성하면서 어디 있으면 되겠다 싶은곳에 넣으면됨
- 양방향 매핑시에 무한루프를 주의 (toString, lombok, JSON 생성 라이브러리 사용시)
    - Lombok 에서 toString 은 지양해라
    - Entity를 JSON 생성할때 쭉 뽑으면서 에러가 남 (Entity를 컨트롤러에서 직접 반환하는 경우) → 컨트롤러에는 그냥 Entity 를 쓰지말고, DTO 로 변환해서 반환해라
- 정리: 단방향 매핑만으로도 이미 연관관계는 매핑이 완료된거임
    - 객체입장에서 “양방향매핑”은 단점이 많아짐
    - 처음에 설계할 때, 먼저 단방향 매핑으로 싹다 설계를 완료해야함
    - (양방향매핑은 그냥 읽기 권한만 추가되는거라서) 필요시에만 양방향을 추가함 → 양방으로 바꾸는건 그냥 List 만 하나 추가해주면 되기때문
- 중요) 연관관계의 주인은 외래키의 위치를 기준으로 정해야함

### 실전 예제 2 - 연관관계 매핑 시작

- (그전엔 DB 설계대로 만들었는데, 이제 객체지향 스럽게 연관관계를 설계해보자)
- (섹션4 마지막에 했던 실전예제 1을 객체연관관계 매핑으로 변경하는 내용)
- 가급적이면 단방향 매핑이 좋은거임 → 실제 개발하다가 필요할때만 양방향 매핑
- “특정 회원의 전체 주문목록” 을 가져오는 기능을 구현하려면 “주문” 에서 특정 멤버의 정보를 가져와야지, 특정 멤버 → 멤버의 주문 으로 가는건 관심사를 끊어내지못하고 더 어렵게 설계한거라고 생각함 (주문이 필요하면 그냥 주문에서부터 시작하면됨)
- 실전에서는 JPQL 을 작성하는 경우가 꽤 많은데 이럴경우 양방향 연관관계가 필요한 경우가 많다

## 섹션 6. 다양한 연관관계 매핑

### 다대일 [N:1]

- 기본적인 연관관계 고려사항을 따져볼것이고, 여러 매핑시 고려사항을 알아봄
- 연관관계 매핑시 고려사항 3가지:
    - 다중성 → n:1, 1:n, 1:1, n:m (다대다는 실무에선 지양하는게 좋음)
    - 단(양)방향 → 테이블은 외래키 하나로 양쪽 조인으로 데이터를 가져오기때문에 “방향”이라는 개념이 없지만, 객체는 참조형 필드가 있어야해서 단/양방향이 있음. 그리고 사실 “양방향”은 아니고 단방향 2개임
    - 연관관계의 주인
- 다대일 단방향: 당연히 테이블에는 다(N) 쪽에 외래키(FK)가 가야하고, 객체는 외래키 있는쪽에 참조객체를 넣으면 됨 (가장 많이 사용하는 연관관계)
    - 여기서 양방향을 하려면 간단하게 반대쪽에 List 만 넣고 `mappedBy` 해주면 해결

### 일대다 [1:N]

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/5859b315-f6b4-45df-8073-9f247df06d95/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230131%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230131T131925Z&X-Amz-Expires=86400&X-Amz-Signature=0ec2503d76e531091e34c471db51b27b4ece34bb8515454101a431e16141d42d&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

- 여기선 1쪽에서 외래키를 관리 (강사님은 권장하지 않는 모델임, 스펙상 스프링이 지원만 할 뿐 실무에서 지양함)
- `@OneToMany` 를 사용한 `List<Mamber> members` 한곳에 `@JoinColumn( .. )` 을 선언해주면 실제로 사용 가능
- 이렇게 매핑할 시 하이버네이트가 옆 테이블 (FK 가 있는 테이블) 의 업데이트 쿼리를 만들어줌
- 가장 큰 단점: 도메인 입장에서 Team Entity 만 수정했는데도, Member Table 의 update 가 나가기때문에 트래킹이 굉장히 힘들거나 사용이 난해해짐
- 이런 구조가 너무 필요하다면 그냥 원래대로 n:1 로 설계하고, 양방향 매핑으로 가져오는걸 권장
- 주의) `@JoinColumn( .. )` 을 안써버리면 테이블과 테이블 사이에 JPA 가 매핑테이블을 만들어서 관리함 → 테이블이 하나 더들어가니까 성능이슈도 있고, 운영에서 너무 복잡해짐
- 결론) 조금 객체지향적으로 참조를 하나 더 넣더라도 “다대일 양방향 매핑”을 사용하는것을 권장
- 일대다 양방향
    - 이런 매핑은 공식적으론 없는거지만, `@ManyToOne` 와 `@JoinColumn(name="..", insertable=false, updatable=false` 를 사용하면 insert 와 update 를 사용하지 않는 읽기전용 (==mappedBy랑 비슷한) 역할을 할 수 있음

### 일대일 [1:1]

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/80d91aa4-23b1-4203-bab4-ef3c829a42c4/Untitled.png)

- 일대일은 대칭관계이기때문에, 주테이블이나 대상테이블에 아무곳에나 외래키 선택이 가능
- 외래키 데이터베이스에 유니크제약조건을 추가해주는게 좋다
- `XXXToOne` 은 사용법이 비슷함
- (위 그림대로 설계하면 굉장히 쉽고 편리함)
- 일대일인데 대상테이블(locker) 에 FK 가 존재하는 단방향은 JPA 에서 아예 지원하지 않음
- 일대일인데 대상테이블(locker) 에 FK 가 존재하는 양방향은 그냥 위 그림에서 locker 쪽에 외래키를 넣고, 반대쪽에 읽기전용 mappedBy 를 넣어주면됨
- 외래키는 Member 에 넣는게 좋을까 Locker 에 넣는게 좋을까? (일단 정답은 없지만)
    - 만약에 1:1이 1:n 으로 바뀔 미래를 생각하면 → Locker 쪽에 FK 가 있다면 그냥 유니크 제약조건만 빼버리면됨
    - 실제 ORM 을 사용해서 개발을할 개발자 입장에선 Member 에 Locker_Id 를 FK 로 가지고있으면 비즈니스 로직에서 멤버는 대부분 가져올때 이미 FK 값이 있으므로 JOIN 을 사용하여 쉽게 Locker 의 상태를 알 수 있음. 단, Locker 가 null 일수도있음
    - 강사님은 그냥 Member 에서 Locker 를 가져가는걸 선호함 (위의 그림 그대로)
    - Locker 에 Member_Id FK 가 걸려있으면 무조껀 양방향으로 걸려있어야됨 (1:1 단방향 반대쪽 FK는 지원을 안하니까)
    - 주테이블에 외래키 → 주테이블만 조회해도 이미 Locker 정보를 알기 쉽지만, null이 들어갈 수 있음
    - 대상 테이블에 외래키 → 나중에 Locker 가 N 으로 변경되어도 수정할게 적음, 단 Entity 구조 자체가 반드시 양방향으로 해야하고 지연로딩을 사용할 수 없음

### 다대다 [N:M]

- (강사님은 실무에서 쓰면 안된다고 생각)
- 관계형 데이터베이스는 정규화된 테이블2개로 다대다 관계를 표현할 수 없음 → 따라서 연결테이블(매핑테이블) 을 사용해서 1:n 두개로 만들어야함
- 근데 객체는 N:M 구조가 그냥 가능함 (컬렉션 2개 넣으면 되니까) `@ManyToMany` 를 사용하고 동일하게 단/양방향 모두 가능
- 실무 테이블에선 연결만 하고 끝나지 않고 복잡한데, 매핑테이블에 다른 필드(칼럼)들이 필요한데 이런것들 추가가 불가능하고, 중간 테이블이 숨겨져있어서 쿼리가 굉장히 복잡하게 나감
- n:m 이 필요한경우 중간에 수동으로 매핑테이블을 만들고 1:n 두개를 만들어서 연결
- 중간 매핑테이블에서 실제로 사용할땐 PK는 의미없는 값을 넣고, FK 로 그냥 매핑하는게 대부분 괜찮았다

### 실전 예제 3 - 다양한 연관관계 매핑

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/22bf2a4a-6254-4170-a451-b060d1b66315/Untitled.png)

- (1:1 관계와 N:N 관계를 추가하고 Entity 를 설계하는거 실습) `@ManyToMany` 지양하라고 했지만 보여줄려고 그냥 사용함
- (`@JoinColumn` `@ManyToOne` 등의 옵션들 한번 찾아볼 것)
    - 다:1 은 `mappedBy` 자체가 없음 (연관관계 주인이 되어야해서)

## 섹션 7. 고급 매핑

### 상속관계 매핑

- 객체는 상속관계가 있지만, DB 테이블은 상속개념이란개 (몇개 엔진 빼곤) 없음
- 슈퍼타입 - 서브타입 관계가 그나마 객체의 상속관계와 비슷함
- DB 는 이런 상속 구조를 3가지 전략으로 구현할 수 있음
    - 조인 전략: 최상위에 item table 을 두고, PK 값과 DTYPE 으로 구분해서 가져옴 → 4개의 테이블
    - 단일테이블 전략: 한개의 테이블에 때려박기 → 1개의 테이블 (단순하고 성능이 좋음(조인이 없으니까;))
    - 개별테이블 전략: 테이블을 각각만들고 똑같은 필드를 그냥 중복시킴 → 3의 테이블
    - 여기서 어떤 방법을 선택하던 JPA 로 구현 가능함
- 상속으로 해결할 시 JPA 의 기본전략은 “단일테이블전략” 임

```java
@Entity 
@Inheritance(strategy = InheritanceType.JOINED) // JOINED, SINGLE_TABLE, TABLE_PER_CLASS
@DiscriminatorColumn
public class Item { // TABLE_PER_CLASS 시에는 abstract class 로 선언해서 Entity 생성이 안되게 
	...
}

//
@Entity 
//@DiscriminatorValue("B")
public class Book extends Item {
	...
}

@Entity
public class Album extends Item {
	...
}
```

- 조인 전략 (사용법은`@Inheritance(strategy=InheritanceType.JOINED)` 로 선언하면 됨)
    - `@DiscriminatorColumn` 을 넣어주는게 좋음 → DTYPE 없이 상위테이블만 보면 얘가 뭐에 관련된 데이턴지 알 수 없기때문에
    - JPA 스펙상 조인전략도 DTYPE 이 필수인데, 하이버네이트 구현체는 필수는 아닌듯 (선언 안하면 안만들어짐)
- 단일테이블 전략
    - 성능이 제일 잘나옴(단일 테이블이라서), join 없고, insert 도 쉽게 나감
    - 단일테이블은 칼럼 구분을 할 수 없기때문에 DTYPE 이 필수로 들어가야함 (생략해버리면 JPA 가 강제로 넣어버림)
- 구현클래스마다 테이블 전략
    - 상위 테이블은 추상 (abstract) 클래스로 만드는게 맞음(상속으로 사용할꺼니까)
    - 테이블이 각자 다 다르고 연관된것도 없어서 DTYPE 을 선언해도 생성안됨
    - 이 전략은 값을 넣을땐 편리한데, 부모클래스 타입으로 조회하면 UNION ALL 로 다찾아와야함 (어디에 뭐가있는지 모르니까)
- 각각의 장단점
    - 조인 전략
        - 테이블이 정규화가 잘 되어있음, 외래키 참조 무결성 제약조건을 지킴, 저장공간이 효율적임
        - 조회할 때 뭘해도 JOIN 해야해서 성능이 저하되고, 쿼리가 복잡
        - 이게 객체랑도 잘 맞고 정규화도 되고 설계도 깔끔해서 거의 정석적인 방법
    - 단일 테이블 전략
        - 조인이 필요없으므로 단순함, 조회도 단순함
        - (치명적인 단점) 관계 없는 테이블은 싹다 null 값이 들어가야됨 → 데이터 무결성 제약조건을 만족하지않음
        - 한테이블에 다들어가므로 테이블에 데이터가 비약적으로 많아질 수 있다
    - 구현클래스마다 테이블 전략
        - (일단 이건 쓰지마셈) DBA 와 객체설계 뭘로봐도 안좋음
        - 서브타입을 완전히 구분해서 처리할 때 쓸만함
        - 쿼리가 UNION 으로 나가야하고 아주 복잡함
        - 시스템이 한번 만들어지면 변경하기 힘들어짐
- 강사님은 보통 조인전략을 사용하고, 애플리케이션이 아~주 단순하면 그냥 단일테이블로 함, “구현클래스마다 테이블전략 은 절때 쓰지마라.”

### Mapped Superclass - 매핑 정보 상속

- 매핑정보만 받는 부모(슈퍼) 클래스
- 이건 엔티티도 아니고, 절때 상속관계 매핑이 아님 → `@MappedSuperclass` 는 직접 조회 (em.find) 가 안됨
- 직접 생성할일이 없으므로 추상클래스로 사용하시길 권장

```java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
public abstract class BaseEntity {

    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
    
}
```

### 실전 예제 4 - 상속관계 매핑

- (기존의 실전예제 4에 Item 상속관계와 MappedSuperclass 로 생성일자 데이터 등을 추가)
- 애플리케이션이 너무 커지면 잘만든 상속관계에서도 복잡해서 다시 또 고민을 해야함 (정답은 없음)

## **섹션 8. 프록시와 연관관계 관리**

### 프록시

- 일단 JPA 에서 프록시 개념이 쉽지가 않음
- 공부를 할때는 왜? 써야하는지를 꼭 알아야됨
- 연관관계가 걸려있다는 이유만으로 항상 모든 데이터를 한번에 (JOIN) 가져오면 비용만 늘어나고 비효율적임 → 이런 방법론을 지연로딩 & 프록시로 해결
- JPA 는 `em.getReference()` 를 지원하고 이는 데이터베이스에 직접 조회하지 않는 프록시 객체를 던져줌
- 프록시는 Entity 의 본체를 상속받아서 만든 더미클래스이며, 실제 엔티티처럼 사용해도 이론상 상관이 없다
- 특징
    - 프록시 객체는 참조(Target Entity) 를 보관
    - 프록시 객체는 `.getName` 을 호출하고 진짜 값이 없으면 영속성 컨텍스트를 건너서 데이터를 받아온 뒤 실제 Entity 의 위치를 저장하고, 상위 Entity 의 메서드를 실행한다 (이 동작 방식 자체는 JPA 스펙이 아니라서 구현체마다 다를 수 있음)
    - 프록시 객체는 처음 사용할 때 1번만 초기화 됨
    - 프록시 객체를 초기화 하면 실제 엔티티로 변경되는게 아니고, 실제 엔티티에 접근해서 사용하는 것 임 → 따라서 “==” 비교를 하게되면 타입이 다를 수 있고, “instance of” 를 사용해야 함
    - 영속성 컨텍스트에 엔티티가 이미 들어있으면 `getReference` 를 호출해도 프록시가 아닌 실제 Entity 를 반환
        - 이미 영속성 컨텍스트에 올려놨는데 프록시를 던져봐야 의미가 없음
        - JPA 는 같은 영속성 컨텍스트내에서 “==” 비교했을때 항상 같은 결과가 나오도록 지원해야 하기때문에 → 재밌는건 프록시를 비교해도 같은 프록시를 가져옴
        - 더 재밌는건 프록시로 먼저 조회를 하고 `em.find` 를 호출하면 find 를 해도 프록시 객체가 나온다 (==을 지원하려고)
    - 준영속 상태일 때 프록시를 초기화하면 초기화되지 않았다는 에러가 발생
- 프록시용 유틸리티 메서드가 잇음
    - `isLoaded` : 프록시 인스턴스 초기화 여부 확인
    - `getClass.getName` 으로 프록시 클래스를 확인 할 수 있음
    - `Hibernate.initialize(ref)` 로 호출하지 않고 강제 초기화 할 수 있음 → JPA 표준스펙에 없어서 Hibernate 임
- 그럼 `getReference` 많이 쓰나요? → 그렇지는 않음 잘 안씀 but 프록시개념을 알고있어야 잘 쓸 수 있음

### 즉시 로딩과 지연 로딩

- 팀와 멤버를 한번에 다가져오면 손해라서 JPA 는 `@ManyToOne(fetch = FetchType.LAZY)` 로 지연로딩을 지원함
- 지연로딩 객체는 처음에 프록시 객체를 반환
- `getTeam` 까지가 아니고 실제 내부에 접근 (e.g. `getTeam.getName`) 을 하면 초기화함
- `@ManyToOne(fetch = FetchType.EAGER)` 로 즉시로딩을 지원함 (한번에 가져오고 초기화까지 다 끝난상태)
- EAGER 로 선언되어있으면 각자의 쿼리가 나가기 vs 조인으로 한번에 가져오기 선택이 가능한데, 대부분의 JPA 구현체들은 조인전략으로 구현함
- 강사님 피셜) 실무에선 가급적 지연로딩만 사용
    - 즉시로딩을 적용하면 전~혀 예상하지 못한 SQL 이 막 나감 → JOIN 이라고 해서 막 느리진 않은데 테이블이 많아지면 성능이 처참해짐
    - 즉시로딩은 JPQL 에서 N+1 문제를 일으킨다 → em.find 는 pk 로 가져오는것이기때문에 JPA 가 JOIN 최적화를 할 수 있지만, JPQL 은 쿼리를 그대로 실행하기때문에, 해당 객체의 select 가 나가고 그 행의 갯수만큼 또 select 문이 나간다
    - 해결법 3가지: 페치조인을 사용 or 엔티티그래프 사용 or 배치사이즈를 조정해서 1:1 로 해결
- (실습은 그냥 이론이고 현업에선 모두 지연로딩을 사용하길 권장)

### 영속성 전이(CASCADE)와 고아 객체

```java
// 예시 
@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval =true)
private List<Child> childList = new ArrayList<>();
```

- 영속성 전이 (CASCADE) 는 연관관계와 전~혀 상관이없음
- 부모를 저장할때 자식도 함께 저장하고 싶거나 할 때 (특정 엔티티를 영속상태로 만들때 연관 엔티티를 함께 영속상태로 만들기위해) 사용 (모두 `em.persist` 가 나감)
- 양방향 편의 메서드 만들때 기존 값이 있나 확인하고 빼주는게 나중에 코드 (복잡하지만 사용)
- 개발할 때 부모코드를 중심으로 할 때 자식클래스의 `em.persist` 를 자동으로 하고싶을 때 사용
- 종류
    - ALL: 모두 적용
    - PERSIST: 영속화만 함께 적용
    - REMOVE: 삭제
- 그래서 언제 써야하나? → 하나의 부모가 자식들을 오롯이 관리할 때 (게시판과 게시물 첨부파일 관계), 다른 Entity 에서 또 관리를 하면 사용했을때 난리날 수 있음 (소유자가 하나일때만 사용) + 라이프사이클이 동일할때
- 고아객체란 “부모엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능” (컬렉션에서 빠진애는 디비에도 없애버림)
    - `orphanRemoval = true` 로 사용 가능
- 참조가 제거된 엔티티는 다른곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능
- 참조하는 곳이 하나이고, 특정엔티티가 개인이 소유할 때 사용
- 개념적으로 부모 Entity 가 지워지면 자식은 자동으로 고아객체기때문에 다 삭제됨 → 이는 삭제가 전파되는 `CascadeType.REMOVE` 처럼 동작한다
- 영속성전이 + 고아객체 = ? : 이 두옵션을 다 활성화하면 부모의 생명주기로 자식의 생명주기를 동일하게 관리할 수 있음
- DDD의 Aggregate Root 에서 “리포지토리는 Aggregate root 만 구현하고 나머지는 Repository 를 만들지 않는것이 낫다” 라는 조건을 구현 할 수 있음

### 실전 예제 5 - 연관관계 관리

- (모든 연관관계를 지연로딩으로 변경하는 실습)

## **섹션 9. 값 타입**

### 기본값 타입

- 임베디드타입과 값타입컬렉션이 중요
- JPA는 최상위 데이터타입을 “엔티티타입”과 “값타입” 으로 구분
    - 엔티티타입: `@Entity` 로 정의하는 객체, 데이터가 변해도 식별자(pk) 로 추적 가능
    - 값타입: int, Integer 처럼 단순히 자바 기본 타입이나 객체, 식별자가 없으므로 값 변경시 추적 불가
- 값타입에는 기본값타입(자바 기본타입, 래퍼클래스, String), 임베디드타입(e.g. x,y 로 만든 좌표클래스), 컬렉션 값타입(자바 컬렉션) 3개가 있음
    - 기본값타입:
        - 생명주기를 엔티티에 의존 → 엔티티를 삭제하면 이름, 나이도 같이 삭제됨
        - 값 타입은 공유되지 않음 → 회원 이름을 바꿨다고 다른 회원 이름도 바뀌면 안되니까 → 자바의 기본 타입은 절때 공유되지않음 (자바 기본 지식)
        - `int a = 10; b = a;` 로 해놓고 `a=20` 으로 변경해도 기존 값은 변경되지 않음 (자바의 기본타입에 대한 기본기임) → primitive type 은 공유되지 않음
        - but Integer 같은 래퍼 클래스나 String 같은 클래스들은 참조로 공유 가능하나, 변경 자체가 불가능하게 해서 사이드이펙트가 없음

### 임베디드 타입

- (내장 타입 으로 번역되기도 함)
- JPA 는 embedded type 이라고 부름
- int 나 String 처럼 임베디드타입도 값타입으로 분류되기때문에 주의해야 한다
- 장점
    - 재사용이 가능 (기간이나 주소 같은거)
    - 응집도가 높다 (따로 관리하지않음)
    - `Period.isWork()` 같은 객체지향적인 의미있는 메소드를 생성 가능
    - 값타입을 가지는 엔티티의 생명주기에 의존함 (엔티티가 사라지면 날짜들도 사라짐)
- 테이블은 테이블대로 생기는게 맞지만, 객체는 공통화하고 묶어주는게 (의미있는 메서드 만들기도 쉽고) 유리함
- `@Embeddable` 이나 `@Embedded` 는 둘중에 하나만 넣고 생략해도 되지만 강사님은 둘다 넣는걸 선호
- 특징
    - 임베디드 타입은 사용하기 전과 후에 매핑하는 테이블은 같다
    - 객체와 테이블을 아주 세밀하게 매핑할 수 있다
    - 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많음
- 만약에 같은 Entity 에서 같은 값 타입을 여러개 사용하면? → 원래는 에러남 이때 `AttributeOverride` 를 사용해서 사용하게 할 수 있음 (자주 사용하진 않음)
- 당연한 말이지만, 임베디드타입이 null 이면 안에 있는 필드들도 다 null 임

### 값 타입과 불변 객체

- 값 타입은 복잡한 객체 세상을 조금더 단순화하려고 만든 개념이라서 값타입은 단순하고 안전하게 쓸 수 있어야함
- 단, 값타입 공유참조(하나의 임베디드 타입을 다른 엔티티에서 볼때) 상황에서 변경하면 문제가 생김
- 만약에 난 “이렇게 같이 변경되는것도 의도했어” 라고 하면 값타입 대신 새로운 Entity 로 사용해야 함
- 한계
    - 항상 값을 복사해서 사용하면 공유참조 문제로 발생하는 부작용을 피할 수 있음
    - 직접 정의한 값타입은 자바의 기본타입이 아니라 객체타입이라서 참조값을 대입하는 방법론을 (개발자가 입력하는것을) 막을 수 없다
    - 객체의 공유참조는 피할수가 없다
    - 객체타입의 한계: primitive type 은 값을 복사해서 대입해서 안전, 객체타입은 참조를 전달하기 때문에 안전하지않음
- 객체 타입을 “불변객체”로 설계하여 부작용을 원천차단 해버리는게 좋음 → 생성자로만 값을 설정하고 setter 를 다막아버림 → 자료들 찾아보면 방법론이 더 있음 찾아보는게 좋음
- 그럼 값을 바꾸고싶을땐? → 새로운 객체를 만들어서 new 로 아예 그 값타입 자체를 통째로 바꾼다 (부분만 바꾸지말고)

### 값 타입의 비교

- 값 타입은 인스턴스가 달라도 그 안의 값이 같으면 같은 개념으로 봐야함
- 자바의 == 비교 자체가 참조값을 비교하기때문에 새로 new 로 생성한 객체가 같다고 나올리가 없음
- 동일성 비교: 인스턴스의 참조를 비교 (`==`)
- 동등성 비교: 인스턴스틔 값을 비교 (`equals()`) → equals 의 기본값도 ==는 비교를 사용하기때문에, 오버라이드 해줘야함 (알겠지만 equals 를 구현해주면 hashCode 도 구현해줘야 함)

### 값 타입 컬렉션

- 값타입을 컬렉션에 넣어서 사용하는걸 “값 타입 컬렉션” 이라고 한다
- 데이터베이스 테이블은 컬렉션을 지원하지 않기때문에 (최근엔 JSON으로 풀어서 하는 방법 등이 존재하지만 원래는 안되므로) 컬렉션을 별도의 테이블로 풀어서 저장해야 한다
- 값타입 컬렉션은 영속성전이 + 고아객체제거 기능을 필수로 가진다
- 값타입 컬렉션도 지연로딩을 사용한다
- 값 타입은 엔티티와 다르게 식별자 (PK) 가 없기때문에 변경하면 추적이 어려움
- 값 타입 컬렉션에 변경사항이 발생하면 그냥 모든데이터를 다 날리고 현재 필요한 값을 다시 넣는식으로 관리함 (wow..) → 결론부터 말하자면 그냥 값 타입 컬렉션 쓰지말고 1:N 으로 엔티티를 만들어서 관리하고 여기에 값 타입 (컬렉션x) 을 사용하고 영속성전이+고아객체제거 를 사용하는게 나음
- 값 타입 컬렉션 매핑은 모든 칼럼을 묶어서 기본키로 사용해야 함 (wow..)
- 그럼 언제써야하나? → 진~짜 단순해서 SelectBox 에 단순한 값 몇개 넣고 추적도 필요없고 수정도 필요없고 그럴때 사용

### 실전 예제 6 - 값 타입 매핑

- (지금까지 배운 값타입들을 실전예제5 코드에 적용하는 실습)
- 임베디드타입의 equals 를 자동 구현할 때, 필드 직접접근보다 getter 를 사용하는게 좋다 → 프록시가 getter 를 사용하기때문

## **섹션 10. 객체지향 쿼리 언어1 - 기본 문법**

### 소개

- JPA 는 다양한 쿼리 작성 방법들을 지원함 → JPQL, JPA Criteria, QueryDSL, Native SQL, JDBCTemplate, MyBatis ..
- JPQL
    - 테이블은 매핑만 하는거지 우리는 엔티티 객체를 중심으로 개발을 해야하고, 검색을 할때도 테이블이 아닌 엔티티 객체를 대상으로 쿼리를 날릴 수 있어야함
    - JPA 는 SQL 을 추상화한 JPQL 이라는 객체 지향 쿼리 언어를 제공하며, SQL 과 유사함
    - JPQL 은 객체를 대상으로 쿼리를 하는것이고, SQL 은 테이블을 대상으로 쿼리를 한다는 점이 다름
    - JPQL 도 단순히 그냥 문자열이라서 “동적쿼리” 작성하기가 너무 힘듦
- Criteria (자바 표준에서 지원)
    - (실무에서 강사님이 써봤는데 너무 복잡해서 잘 안쓰게됨) → QueryDSL 을 추천
    - 문자대신 자바 코드로 JPQL 을 만들 수 있고, JPA 공식임 → (JPQL 빌더 느낌)
- [QueryDSL](http://querydsl.com/)
    - 딱 읽어보기만 해도 SQL 이랑 문법이 똑같음
    - Criteria 처럼 JPQL 빌더 역할이며, 자바코드로 JPQL 작성가능
    - QClass 를 사용하여 컴파일 시점에 문법오류를 찾을 수 있음
    - (강사님 피셜 복잡한쿼리가 나가야하는 대부분의 상황에 그냥 QueryDSL 을 깔고감)
    - 아무리 QueryDSL 을 쓰더라도 JPQL 을 알고 써야 잘 쓸 수 있음
- Native SQL
    - 특정 데이터베이스만 종속적인 기능이나 hint 를 사용할 수 있도록 JPA 에서 지원
    - `em.createNativeQuery( .. )` 로 작성 가능
- SpringJdbcTemplate 지원
    - (강사님은 Native 를 직접쓰기보다 그냥 JDBC Template 를 사용)
    - 주의) JPA 를 사용할 때는 flush 를 관리 해주지만, db connection 을 가져와서 executeQuery 할 때는 강제로 `em.flush()` 를 사용해줘야함 (JPA 를 우회해서 SQL을 실행한다면 그 직전에 수동으로 플러쉬)
    - 플러시는 네이티브쿼리랑 commit 할때 자동으로 날아감 (기존 내용)
- JPQL 만 잘한다면 그냥 대부분 다 잘 할 수 있음 → 95% 정도는 JPQL 과 QueryDSL 로 대부분 해결 가능

### 기본 문법과 쿼리 API

- [JPQL](https://en.wikibooks.org/wiki/Java_Persistence/JPQL) == Java Persistence Query Language
- (JPQL 기본적인 사용법 실습 내용)
- JPQL 작성시에 내용에 대소문자를 구분함 (객체의 내용과 똑같이)
- 반환 타입이 명확하면 `TypeQuery`, 명확하지 않으면 `Query` 를 지원한다
- 결과는 `getResultList()` 를 쓰면 컬렉션(비면 빈 컬렉션이)이, `getSingleResult()` 를 쓰면 그냥 그 객체(결과가 없거나 여러개면 에러남)가 나옴
    - `getSingleResult()` 는 나중에 Spring Data JPA 를 쓰면 에러가 안나고 그냥 null 이나 optional 이 던지도록 확장되어있음
- 파라메터 바인딩시에 “위치”와 “이름” 기준으로 할 수 있는데, “이름”으로 하는게 덜 헷갈리고 휴먼에러 발생 가능성도 적음

### 프로젝션(SELECT)

- 프로젝션 == select 절에 조회할 대상을 지정하는 것을 말함
- 원래 DB 는 스칼라 타입만 넣을 수 있는데 여기선 엔티티, 임베디드타입도 모두 사용 가능
- SELECT 에서 엔티티프로젝션(엔티티를 가져오면) 조회된 모든 엔티티가 영속성컨텍스트에서 관리됨
- (경로표현식에서 설명 하겠지만) JPQL 에선 자동으로 생성된 SQL JOIN 을 쓰기(묵시적으로 사용)보다 명시적으로 코드를 작성해주는게 좋다 (묵시적으로 하면 쿼리가 어떻게 나갈지 예측이 힘들고, 튜닝할때 추측으로 해야하니까)
- 임베디드값타입의 한계) 그냥 임베디드타입으로 호출이 불가능하고 (소속되어있기때문에) 소속을 알려줘야함
- 스칼라 타입 프로젝션은 그냥 필드를 적어주면 됨
- `SELECT DISTINCT ~` 로 중복을 제거할 수 있다
- 여러 타입을 한번에 가져올 경우는?
    - TypeQuery 말고 Query 로 받으면 됨 그럼 내부에 `Object` 배열로 값이 넘어옴
    - 제네릭으로 `List<Ovject[]> resultList = em.createQuery(..).getResultList();` 하면됨
    - (제일 깔끔하게) new 명령어로 DTO 로 바로 조회 가능 (내가 DTO 프로젝션이라 불렀던 것) →  type 을 DTO 로 주고 `em.createQuery("select new jpql.aDTO from ~", aDTO).getResultList();` 로 가져올 수 있음 → 순서와 타입이 일치해야 함

### 페이징

- 오라클 데이터베이스 같은건 페이징이 거지같은데, JPA 는 `setFirstResult` (시작위치) 와 `setMaxResults` (조회 데이터 수) 로 잘 추상화 되어있음
- `order by` 로 sort 까지 넣어야 페이징이 완벽히 잘되는지 확인되는거지 (순차적으로)
- 미세팁) Entity `toString` 만들땐 연관관계 무한루프 주의
- 페이징도 각 데이터베이스의 방언에 맞춰서 만들어줌

```java
String jpal = "select m from Member m order by m.name desc";
List<Member> resultList = em.createQuery(jpql, Member.class)
	.setFirstResult(10)
	.setMaxResults(20)
	.getResultList();
```

### 조인

- 조인은 SQL 조인이랑 실행은 똑같은데 객체 스타일로 조인 문법이 나감

```java
// List<Member> result = em.createQuery("select m from Member m left outer join m.team t", Member.class) // outer 는 생략 가능
List<Member> result = em.createQuery("select m from Member m inner join m.team t", Member.class) // inner 는 생략 가능
	.getResultList();
```

- JPA 2.1 부터 on 절을 활용하여 조인 대상을 필터링하고

```java
String jpql = "select m, t from Member m left join m.team t on t.name = 'a'"
```

- (하이버네이트 5.1부터) 연관관계가 없는 엔티티 외부 조인을 지원한다 (내부조인은 원래 됐음)

```java
String jpql = "select m, t from Member m left join Team t on m.username = t.name" // left 빼면 당연히 내부조인 
```

### 서브 쿼리

- 우리가 알고있는 SQL 서브쿼리와 거의 동일
- JPA 서브쿼리의 한계: 원래 JPA 표준에서는 WHERE 과 HAVING 에서만 서브쿼리가 가능 → 하이버네이트 구현체는 SELECT 도 가능
- FROM 절에서의 서브쿼리(인라인뷰)는 현재 JPQL 로 아예 불가능 → JOIN 으로 풀어서 해결해야함 → 여기까지 해도 안되면 그냥 네이티브로 해결 or 애플리케이션레이어에서 해결 (그냥 여러번의 쿼리 날림)

### JPQL 타입 표현과 기타식

- 문자와 숫자를 어떻게 JPQL 에서 표현하는가? 문자는 그냥 `'` 으로
- 숫자는 그냥 10L, 10D, 10F 다 표현 가능
- Boolean 은 TRUE, FALSE
- ENUM 을 바로 쓰고싶은경우는 패키지명을 다 입력해줘야함
- 다형성을 이용한 엔티티 타입: TYPE(m) = Member → `"select i from Item i where type(i) = Book"`
- 앵간한 표준 SQL 은 다 지원함

### 조건식(CASE 등등)

- (CASE 식, coalesce, nullif 등 문법 실습) → 이런 대부분의 표준함수는 디비 엔진과 상관없이 동작함

### JPQL 함수

- JPQL 의 표준함수 (디비와 상관없는), 사용자 정의함수를 DB 방언에 추가해서 사용 → 기본적으로 하이버네이트 내부에 `registerFunction` 으로 대부분 미리 등록되어있음
- (사용자 정의 함수 등록하는 방법 알려주심)

## **섹션 11. 객체지향 쿼리 언어2 - 중급 문법**

### 경로 표현식

- 경로표현식: 점(`.`) 찍어서 객체 그래프를 탐색하는 것
- 상태필드 → `m.username` 처럼 필드 값을 저장하기 위한 필드
    - 경로 탐색의 끝이라 더이상 탐색이 안됨
- 연관 필드 → `m.team` 처럼 연관관계를 위한 대상이 엔티티인 필드(`@xxxToOne`), `t.users` 처럼 대상이 컬렉션인 컬렉션값 연관필드(`@xxxToMany`)
    - 단일값 연관경로(엔티티 대상)는 묵시적 내부조인(innter join) 으로 탐색을 더 할 수 있음 (e.g. `m.team.xxx.name`) → 묵시적으로 조인하면 쿼리를 튜닝하기 쉽지않고, 어디서 어떻게 조인이 발생했는지 팔로우하기 쉽지않아서 실무나 큰 어플리케이션에선 직관적으로 판단할 수 있게 명시적으로 선언하는게 좋음
    - 컬렉션값 연관경로은 묵시적 내부조인은 발생하지만 더이상 탐색은 불가능 (e.g. `t.members`) 이뒤로안됨
    - 컬렉션 값 연관경로는 명시적 조인을 사용하면 더 탐색할 수 있음 (e.g. `"select m.username from Team t join t.members m"`
    - (강사님 피셜) 이런거 다 무시하고 일단 “묵시적조인”을 그냥 쓰면안됨 → 튜닝도 어렵고 조인이 어떻게 발생할지, 쿼리가 어떻게 나갈지 알아보기도 어려움
- 묵시적 조인은 항상 내부조인(inner join) 만 발생한다

### 페치 조인 1 - 기본

- fetch join 에 대해서 알아보고, 실무에서 진짜 중요함 → 디비에 있는게 아님(SQL 조인 종류가 아님)
- JPQL 에서 성능 최적화를 위해 사용 (연관된 것들을 SQL 한번에 받아올때 사용)
- e.g. `[LEFT [OUTER] | INNTER] JOIN FETCH {조인경로}` → `select m from Member m join fetch [m.team](http://m.team)` (이러면 SQL 에 Team 의 모든 정보도 select 함)
- (중요) 대부분의 N+1 문제는 그냥 거의 fetch join 으로 해결
- 지연로딩으로 셋팅을 해도 fetch join 이 항상 우선임
- DB 입장에서 1:N 조인하면 데이터가 fetch join 시 뻥튀기가됨 → `select t from Team t join fetch t.member` 하면 조인된 행만큼 똑같은 데이터가 중복해서 들어오니까
    - 중복은 SQL 의 DISTINCT 를 사용하면 되는데, JPQL 에서 distinct 를 사용하면 객체의 중복값도 제거 해준다 → `select distinct t from Team t join fetch t.members` 하면 됨
    - 근데 distinct 해도 sql 을 실행한 결과는 (멤버 데이터가 다르므로) 뻥튀기 된 중복 자체가 제거되지 않음 → 근데 JPQL 은 `distinct` 를 인지하고 컬렉션에서 똑같은 값을 한번 더 걸러줌
    - 당연하지만 N:1 관계는 뻥튀기가 되지않으므로 그냥 아무렇게나 써도 상관없음
- (정리하면) 일반 조인은 조인된 데이터 안에는 데이터가 안들어있고 getter 를 실행할때 개별로 호출, 페치조인은 조인된 데이터도 안에 데이터를 다 채워서 동시에 (한번에) 가져옴

### 페치 조인 2 - 한계

- 근데 페치 조인 대상에는 관례상 별칭을 사용할 수 없음 (e.g. `.. join fetch t.members as m` 이런식의 `as` 같은 별칭을 주면안됨 → 데이터를 조회할 때 SQL 로 엉뚱한 데이터가 넘어오게 작성될 수 있음)
- 유일하게 쓰는경우는 `select t Team t join fetch t.members m join fetch m.xxx` 뭐이런식으로 진짜 복잡한 페치의 경우 강사님도 써본 적 있음
- 객체그래프를 탐색하는 데이터를 다 조회하는게 좋음 → 근데 그 중에 몇개 데이터만 필요하면 (e.g. 멤버 5개만 필요하면) 팀에서 조회하는게 아니고, 멤버 도메인 가서 조회해야함
- 애초에 `@OneToMany` 같이 연관된 데이터는 다 가져오도록 설계되어있고, 그렇게 사용하는게 맞음
- 둘이상의 컬렉션은 페치조인 불가능함 (어떻게 야매로 가능해도 쓰면안됨 → 이래도 데이터가 잘 안맞음)
- 컬렉션을 페치조인하면 페이징 (e.g. setFirstResult 같은) API 를 사용할 수 없음 → 이것도 1:N 에서 페치조인하면 멤버 수만큼 데이터가 뻥튀기 되었을 때 페이징으로 결과를 짤라서 사용하면 데이터가 원래 데이터와 다를 수 있음 (정합성이 안맞음) → 하이버네이트 구현체에서 강제로 페이징을 사용하면 (얘네도 알 수 없기때문에) 그냥 다 가져와서 메모리에 적재해버림 → 이건 터지면 큰 장애로 발생할 수 있음
- 1:1, N:1 은 페치조인해도 페이징 가능함 → 따라서 멤버(N) 으로부터 팀(1) 로 방향을 바꿔서 해당 문제를 해결 할 수도 있음
- 페이징이 필요할 때 `@BatchSize(size=100)` 를 사용하면, LAZY 걸린 N 개의 문제도 한번에 여러개의 조회쿼리 (where 에 id 를 배치만큼 쭉 넣어버림) 가 나가서 해결할 수 있음 (배치사이즈는 관례상 1000 이하의 수) → 주로 글로벌로 사용하는 경우가 많은데, `hibernate.default.batch_fetch_size` 를 사용하면됨
- 그냥 페치조인을 잘못사용할 때 정합성이 깨지는 모든 경우의수는 사용에 주의해야됨 (거의쓰면안됨)
- 페치조인은 객체그래프를 유지할 때 사용하면 효과적임
- 최근의 성능 튜닝은 굳이 복잡한 sql 이 아니더라도 fetch join 같은 방법론들로 대부분 해결

### 다형성 쿼리

- (크게 중요한건 아님)
- 예를들어서 다형성을 활용하여 엔티티를 설계했을 때, 조회 대상을 특정 자식으로 한정 할 수 있음 → Item 중에 Book, Album 을 조회

```java
// JPQL
select i from Item i where type(i) in (Book, album)
// 여기서 type(i) 부분이 실제 SQL 쿼리의 DTYPE 으로 캐스팅됨
```

- 자바의 타입 캐스팅과 유사한 `TREAT` 가 있음 (JPA 2.1)

```java
// JPQL
"select i from Item i where treat(i as Book).auther = 'kim'"
-> (싱글테이블쿼리) select i.* from Item i where i.DTYPE = 'B' and i.auther = 'Kim'
```

### 엔티티 직접 사용

- 엔티티를 직접사용하면 SQL 에서 해당 엔티티의 기본키를 사용함

```java
// JPQL
"select count(m.id) from Member m" 대신 "select count(m) from Member m" 이렇게 해도 동일한 SQL이 실행됨
-> select count(m.id) as cnt from Member m
// where 문에 "where m.id = :member" 로 써도 똑같이 m.id=? 으로 나감
```

- 엔티티 직접사용 외래키 값 e.g. `where m.team = :team` 이렇게 써도 SQL 은 `where m.team_id = ?` 으로 나감

### Named 쿼리

- 엔티티에 `@NamedQuery` 를 사용해서 이름으로 이 쿼리를 호출할 수 있음 (재활용)
- 얘는 어노테이션이랑 xml 에서도 정의해놓을 수 있음 → xml 이 우선권을 가짐
- 얘는 애플리케이션 뜰때 (로딩될때) JPA 가 캐싱해서 들고있음 → 로딩시점에 이 쿼리가 맞는지 검증됨
- 네임드쿼리의 `name` 은 그냥 `entityName.abc` 로 관례로 많이 씀
- 지금 보면 이렇게 엔티티에 써놓는 경우는 거의없고, 나중에 Spring data JPA 에서 사용할 때 `@Query` 이 어노테이션을 사용하는게 바로 네임드쿼리라 실행시점에 문법오류를 바로알 수 있음 → 엔티티 상위에 있으면 엔티티만 지저분해짐

### 벌크 연산

- 만약에 JPA 변경감지 (더티체킹) 으로 업데이트를 하려면 데이터가 많아질수록 변경건에 대한 Update Query 가 너무많이 나감 → 한번에 동시에 업데이트(==벌크) 가 필요함
- 쿼리 맨뒤에 `.executeUpdate()` 를 쓰면 사용가능

```jsx
int resultCount = em.createQuery("update Member m set m.age = 20").executeUpdate(); // 반환값은 영향받은 행 갯수 
```

- 하이버네이트 구현체는 insert 랑 delete 같은것도 지원함
- (중요) 벌크연산은 영속성 컨텍스트를 무시하고 그냥 데이터베이스에 때려박음 → 해결법은 아래 2개
    - 영속성 컨텍스트가 빈 상태면 벌그연산을 가장먼저 실행
    - 영속성 컨텍스트에 데이터가 있으면 벌크연산 수행 후에 영속성 컨텍스트를 밀어버림 (다시조회함)
- 쿼리 날리면 AUTO 모드일 때 commit 하거나 query 나갈때 자동호출되니 고민안해도 되는데, 문제는 영속성컨텍스트 데이터가 있을때 벌크를 전송하고 바로 이어서 다시 값을 읽어오면 엉뚱한 값이 나오니까 습관적으로 벌크연산 이후엔 `em.clear()` 를 호출하는게 좋음
- 참고로, Spring Data JPA 쓰면 `@Modifying` 을 사용하면 자동으로 auto clear 해줌 → 이런걸 위해 이론을 잘 알고있어야 함

# 📋 메모

- 스프링부트에서 하이버네이트를 사용하면 Entity 를 생성할때 칼럼의 필드명을 카멜케이스 → 스네이크 케이스로 자동으로 바꿔준다
- TIP) 양방향 연관관계에서 다쪽에 있는 `List` 는 `= new ArrayList<>();` 로 초기화 해주는게 Add 할때 nullpointException 이 안뜨니까 관례로 많이 씀
- 실무에선 세터를 잘 안쓰고, 생성자에서 만들거나 빌더패턴을 사용
- (당연한거지만) 에러는 컴파일타임 → 런타임, 로딩시점 → 사용자사용시 순으로 좋은 에러

# 💡 팁 (단축키 등)

- tip) JQPL 문자열이 intellij 에서 문법오류로 반환할 경우 옵션+엔터 했을때 나오는창에서 `Inject language or reference` 에 Hibernate QL 로 지정할 수 있음

# 🔗 레퍼런스

-
