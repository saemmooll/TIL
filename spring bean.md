# 자바 코드로 직접 스프링 빈 등록하기

직접 설정 파일에 등록

- MemberService와 MemberRepository에 있는 @component 다 삭제하고 시작. controller는 그대로 두기.
- SpringConfig 파일 하나 만들기.

```java
package hello.hello_spring;

import hello.hello_spring.repository.MemberRepository;
import hello.hello_spring.repository.MemoryMemberRepository;
import hello.hello_spring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringConfig {

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository()); // 스프링 빈에 등록되어 있는 멤버 리퍼지토리를 멤버 서비스에 주입. 
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}

```

- Controller는 컴포넌트 스캔으로 올라감. + 오토와이드

---

DI: 의존 관계 주입.

1. 필드 주입
- 생성자 빼고 필드에 오토 와이어드
- 그닥 좋지 않음. 중간에 바꿀 수 없음. 스프링 뜰 때 넣어주고 중간에 바꿀 수 있는 방법이 없음.

![Untitled](%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20%E1%84%8F%E1%85%A9%E1%84%83%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%8C%E1%85%B5%E1%86%A8%E1%84%8C%E1%85%A5%E1%86%B8%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%87%E1%85%B5%E1%86%AB%20%E1%84%83%E1%85%B3%E1%86%BC%E1%84%85%E1%85%A9%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%202c9170458fe4440386ad54534e513e26/Untitled.png)

1. setter 주입
- setter 통해서 주입.
- 생성은 생성대로 되고, setter는 나중에 호출되어 멤버 서비스 들어감.
- 멤버 컨트롤러를 호출했을 때, public으로 열려있어야 함.

→ 근데 MemberService을 중간에 바꿀 일 거의 없고, public하게 노출되어 있음.

- 아무 개발자나 호출할 수 있게 열려있음.  호출되면 안되는 메서드가 호출되고 있음.  → 조립(생성)시점에 생성자로 한 번만 조립해놓고 다음에는 변경을 못하도록 막아버리는 것이 좋다.

```java
@Controller
public class MemberController {
    private MemberService memberService;

    @Autowired
    public void setMemberService(MemberService memberService) {
        this.memberService = memberService;
    }
}

```

1. **생성자 주입**: 의존 관계가 실행 중에 동적으로 변하는 경우는 거의 없으므로 생성자 주입을 권장한다. 
- 생성자를 통해서 서비스가 컨트롤러에 주입된다.

![Untitled](%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20%E1%84%8F%E1%85%A9%E1%84%83%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%8C%E1%85%B5%E1%86%A8%E1%84%8C%E1%85%A5%E1%86%B8%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%87%E1%85%B5%E1%86%AB%20%E1%84%83%E1%85%B3%E1%86%BC%E1%84%85%E1%85%A9%E1%86%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%202c9170458fe4440386ad54534e513e26/Untitled%201.png)

- 실무에서는 주로 정형화된 컨트롤러, 서비스 , 리퍼지토리와 같은 코드는 컨포넌트 스캔을 사용. **정형화되지 않거나 상황에 따라 구현 클래스를 변경해야 하면 설정을 통해 스프링 빈으로 등록한다**.

→ 이 강의는 아직 데이터 저장소가 선정되지 않았음을 가정해서 인터페이스로 설계하고 현재 구현체로 멤버 리퍼지토리를 쓰고 있다. 

→ 나중에 메모리 멤버 리포지토리를 다른 리포지토리(데베에 실제 연결하는 리포지토리)로 바꿀 예정. 

→ 기존 운영 중이던 코드를 하나도 손대지 않고 바꾸는 방법 있음.

⇒ 구현체 변경해야 함.

```java
package hello.hello_spring;

import hello.hello_spring.repository.MemberRepository;
import hello.hello_spring.repository.MemoryMemberRepository;
import hello.hello_spring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringConfig {

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new DbMemberRepository();
    }
}
```

⇒ 데베 연결 시 빨간 부분과 같게만 고치면 된다. 

** 주의: @Autowired를 통한 DI는 hellocontroller, MemberService와 같이 스프링이 관리하는 객체에서만 동작. 스프링 빈으로 등록하지 않고 내가 직접 생성한 객체에서는 동작하지 않는다. 

- @Autowired가 작동하려면 해당 객체가 스프링 에 등록이 되고, 스프링이 관리를 해야 함. 스프링 빈으로 등록되어 있어야 함.
- MemberService의 생성자에 @Autowired를 적어놓고 컴포넌트 스캔 또는 직접 스프링 빈 등록 둘 중 어떠한 것도 하지 않으면, @Autowired는 작동하지 않음.
- MemberService 클래스 내에 직접 new에서 멤버 서비스를 생성하는 경우, 이 객체에 @Autowired 작동하지 않음. → 직접 생성한 것이므로 스프링 컨테이너에 올라가지 않기 때문.