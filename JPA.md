# JPA

Java Persistence API

- Jdbc template → 반복 코드는 제거 됐지만, SQL은 개발자가 작성해야 함.
- JPA는 SQL 쿼리도 JPA가 자동으로 처리해준다.
- 객체를 메모리에 넣듯이 JPA에 넣으면 JPA가 중간에서 DB에 SQL을 날리고 DB를 통해 데이터 가져오는 것을 다 처리해준다.

- JPA는 기존의 반복 코드는 물론이고, 기본적인 SQL도 JPA가 직접 만들어서 실행해준다.
- JPA를 사용하면, SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환할 수 있다.
- JPA를 사용하면 개발 생산성을 크게 높일 수 있다.

1. build.gradle 파일에 JPA, h2 데이터베이스 관련 라이브러리 추가

→ implementation 'org.springframework.boot:spring-boot-starter-data-jpa’

→ 내부에 jdbc 관련 라이브러리를 포함한다. 따라서 jdbc는 제거해도 된다

1. 스프링 부트에 JPA 설정 추가

```java
// resources/application.properties

spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.jpa.show-sql=true // JPA가 생성하는 SQL 출력. 
spring.jpa.hibernate.ddl-auto=none
/* JPA는 테이블을 자동으로 생성하는 기능 제공. none을 사용하면 해당 기능을 끊다.
create 사용하면 엔티티 정보(회원 객체)를 바탕으로 테이블도 직접 생성해준다. */
```

1. JPA 사용하려면 먼저 엔티티 매핑을 해야 함. 

![Untitled](JPA%20a94daac851814130a20f137d1e7f3747/Untitled.png)

![Untitled](JPA%20a94daac851814130a20f137d1e7f3747/Untitled%201.png)

- jpa, hibernate 라이브러리 들어와야 함.
- JPA는 인터페이스, 구현채로 Hibernate, Eclipse Link 등의 구현기술 있음.
- JPA는 JAVA 진영의 표준 인터페이스, 구현은 여러 업체들이 하는 것.

- JPA는 객체랑 ORM이라는 기술.
- ORM : Object, Relational, Mapping → object와 relational DB의 Table을 Mapping한다. → annotation이용하여 맵핑.

![Untitled](JPA%20a94daac851814130a20f137d1e7f3747/Untitled%202.png)

→ JPA가 관리하는 엔티티라고 표현. 

- 이후 pk 매핑 : (Primary Key) DB 테이블에서 각 레코드를 고유하게 식별하는 데 사용되는 하나 이상의 칼럼.

→ 현재 PK(ID)를 DB에서 생성해 줌 : 쿼리에 ID를 넣는 것이 아니라 DB에 값을 넣으면 DB가 ID를 자동생성  ← 아이덴터디라 함.

![Untitled](JPA%20a94daac851814130a20f137d1e7f3747/Untitled%203.png)

- 만약 DB의 칼럼 명이 username이면

![Untitled](JPA%20a94daac851814130a20f137d1e7f3747/Untitled%204.png)

→ 위 annoatation들을 가지고 DB와 Mapping.

→ annotation들을 바탕으로 JPA가 쿼리 작성. 

- `em`은 `EntityManager` 객체 →  JPA에서 `EntityManager`는 엔티티를 데이터베이스에 저장, 조회, 삭제 등의 작업을 관리
- EM의 find()은 DB에서 특정 엔티티 클래스를 PK로 조회하는 데 사용. 조회하려는 엔티티 클래스의 타립, PK값을 매개변수로 받음.

```java
package hello.hello_spring.repository;

import hello.hello_spring.domain.Member;
import jakarta.persistence.EntityManager;

import java.util.List;
import java.util.Optional;

public class JpaMemberRepository implements MemberRepository {

    // jpa는 EntityManager로 모든 게 동작. jpa 사용하려면 EM insert 받아야 함.
    // build.gradle에서 data-jpa 라이브러리를 받음 -> 스프링 부트가 자동으로 EM 생성, 현재 DB랑 연결까지 해줌
    // 만들어진 EM을 주입받으면 된다.
    // EM이 dataSource 내부적으로 들고 있어서 DB와의 통신을 내부에서 다 처리.

    private final EntityManager em;

    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }

    @Override
    public Member save(Member member) {
        em.persist(member);
        return member;
    } // jpa가 insert 쿼리 다 만들어서 DB에 넣고, member에 id setting까지 해줌.

    @Override
    public Optional<Member> findById(Long id) {
        return Optional.ofNullable(em.find(Member.class, id));
    }
    // name은 PK가 아니기 때문에 객체지향쿼리 jpql 언어 사용해야 함. sql이랑 거의 동일.
    // table 대상이 아닌 객체(정확한 entity)를 대상으로 쿼리 날림  -> sql로 번역됨.
    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result =  em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name) // 쿼리는 Member 엔티티를 대상으로 하며, name 필드가 주어진 파라미터와 일치하는 모든 Member 객체를 선택
                .getResultList(); //쿼리 실행 후 결과를 리스트 형태로 반환.
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class) // Member 엔티티를 조회하고, Member 엔티티 자체를 select함.
                .getResultList();
    }

    // PK 기반이 아니거나 여러 개의 리스트를 돌리는 경우, jpql 작성해야 함. 
}

```

- JPA 기술을 스프링에 감싸서 제곻하는 기술이 스프링 데이터 JPA. → jpql 안 짜도 된다.
- JPA 사용하려면 데이터 저장, 변경 시 트랜잭션 애노테이션 필요. →MemberService의 회원가입에서 필요.

- 조립하기

```java
package hello.hello_spring;

import hello.hello_spring.repository.*;
import hello.hello_spring.service.MemberService;
import jakarta.persistence.EntityManager;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration // 스프링 빈으로 관리됨.
public class SpringConfig {

    /*private DataSource dataSource; // 스프링이 데베 소스 주입.
    public SpringConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    // 경로를 연결해놨기에 Spring이 그걸 보고 스프링 빈도 자체적으로 생성해줌.
    // 데베소스 : 데베와 연결할 수 있는 접속 정보.*/
    private EntityManager em;
    @Autowired
    public SpringConfig(EntityManager em) {
        this.em = em;
    }

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        //return new MemoryMemberRepository();
        //return new JdbcMemberRepository(dataSource);
        //return new JdbcTemplateMemberRepository(dataSource);
        return new JpaMemberRepository(em);
    }
}

```

- 통합 테스트 돌려보기.

![Untitled](JPA%20a94daac851814130a20f137d1e7f3747/Untitled%205.png)

→ JPA:인터페이스,  Hibernate: 오픈소스 구현체.

- test에 transaction있으면 자동 rollback
- @commit 하면 DB에 반영됨.

- table 내의 member 삭제 : delete from member
- DB 내의 모든 테이블 삭제 : drop all objects