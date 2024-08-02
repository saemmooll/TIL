# 로그인 기능 구현

- Spring Security 사용.
- Spring Security 관리하는 클래스에 로그인 설정 추가.
    - 로그인 페이지의 URL은 `/user/login`, 로그인 성공 시 루트 URL(`/`)로 이동하도록 설정.
- User Controller에 URL 매핑 추가.
    - @GetMapping으로 `/user/login` URL 매핑. 매핑한 login 메서드에서 `login_form.html` 템플릿 출력하도록.
    - @PostMapping 방식의 메서드는 Spring Security가 대신 처리하므로 코드 작성할 필요 없음.
- 로그인 템플릿 작성
    - Spring Security의 로그인이 실패할 경우, 시큐러티 기능에 의해 로그인 페이지로 리다이렉트 됨.
    - 이때 로그인 페이지 매개변수로 error가 함께 전달됨(Spring Security의 규칙) → ‘사용자 ID 또는 비밀번호를 확인해 주세요.’라는 오류 메시지를 출력하도록 함.
        - 스프링 시큐리티는 로그인 실패 시 `http://localhost:8080/user/login?error`와 같이 error 매개변수 전달. 이때 템플릿에서 `${param.error}`로 error 매개변수가 전달되었는지 확인할 수 있음.
        
        ```html
        <html layout:decorate="~{layout}">
        <div layout:fragment="content" class="container my-3">
            <form th:action="@{/user/login}" method="post">
                <div th:if="${param.error}">
                    <div class="alert alert-danger">
                        사용자ID 또는 비밀번호를 확인해 주세요.
                    </div>
                </div>
                <div class="mb-3">
                    <label for="username" class="form-label">사용자ID</label>
                    <input type="text" name="username" id="username" class="form-control">
                </div>
                <div class="mb-3">
                    <label for="password" class="form-label">비밀번호</label>
                    <input type="password" name="password" id="password" class="form-control">
                </div>
                <button type="submit" class="btn btn-primary">로그인</button>
            </form>
        </div>
        </html>
        
        ```
        

- Spring Security에 무엇을 기준으로 로그인해야 하는지 설정하기.
    - 여러 로그인 수행 방법 중 가장 간단한 방법은 SecurityConfing.java와 같은 시큐리티 설정 파일에 사용자 id와 비밀번호를 직접 등록하여 인증을 처리하는 메모리 방식.
    - 회원 정보가 DB에 저장되어 있다면 **DB에서 회원 정보를 조회하여 로그인**하는 방법도 있음.
        - DB에서 사용자 조회하는 서비스 클래스(UserSecurityService.java) 만들고, 스프링 시큐러티에 등록하기.

- UserSecurityService.java 만들기 전
    - UserRepository 수정
        - UserRepository에 사용자 ID로 SiteUser 엔티티를 조회하는 findByusername 메서드 추가.
    - 스프링  시큐리티는 인증뿐만 아니라 권한도 관리. 사용자 인증 후 사용자에게 부여할 권한과 관련된 내용이 필요. 사용자가 로그인한 후, ADMIN 또는 USER와 같은 권한을 부여해야 함.
        - UserRole.java 클래스 생성
            - Enum 클래스 사용.
                - 자바에서 열거형(enumeration)을 정의하기 위해 사용되는 특별한 클래스.
                - 열거형은 관련된 상수 집합을 정의
                - 이러한 상수들에 대해 타입 안정성을 제공하는 방법 제공. 컴파일 타임에 상수 값들의 타입을 검사할 수 있어 잘못된 값이 사용되는 것을 방지.
                - 열거형 상수에 대한 유용한 메서드 제공.
        - 관리자, 사용자를 의미하는 상수를 만들고 각각에 ROLE_ADMIN, ROLE_USER 값 부여.
        - UserRole의 ADMIN과 USER 상수는 값을 변경할 필요가 없으므로 @Setter없이.
            
            ```java
            package com.mysite.sbb.user;
            
            import lombok.Getter;
            
            @Getter
            public enum UserRole {
                ADMIN("ROLE_ADMIN"),
                USER("ROLE_USER"); // 2개의 열거형 상수 정의. 각 상수는 문자열 값을 가짐.
            
                UserRole(String value) {
                    this.value = value;
                }
            
                private String value;
            }
            
            ```
            
            - value: 각 UserRole 열거형 상수에 연결된 문자열 값을 나타냄. 이 문자열은 역할(Role)을 구체적으로 나타냄.
                - 예를 들어, ADMIN 상수에는 ROLE_ADMIN이라는 값이 연결되어 있음.

- UserSecurityService 서비스 생성
    - 스프링 시큐리티가 로그인 시 사용할 UserSecurityService는 스프링 시큐리티가 제공하는 UserDetailsService 인터페이스 구현해야 함. ****
    - UserDetailsService 인터페이스는 loadUserByUsername 메서드를 구현하도록 강제.
        - username으로 스프링 시큐리티 User 객체를 조회하여 리턴하는 메서드.
        
        ```java
        import java.util.ArrayList;
        import java.util.List;
        import java.util.Optional;
        
        import org.springframework.security.core.GrantedAuthority;
        import org.springframework.security.core.authority.SimpleGrantedAuthority;
        import org.springframework.security.core.userdetails.User;
        import org.springframework.security.core.userdetails.UserDetails;
        import org.springframework.security.core.userdetails.UserDetailsService;
        import org.springframework.security.core.userdetails.UsernameNotFoundException;
        import org.springframework.stereotype.Service;
        
        import lombok.RequiredArgsConstructor;
        
        @RequiredArgsConstructor
        @Service
        public class UserSecurityService implements UserDetailsService {
        
            private final UserRepository userRepository;
        
            @Override
            public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
                Optional<SiteUser> _siteUser = this.userRepository.findByusername(username);
                if (_siteUser.isEmpty()) {
                    throw new UsernameNotFoundException("사용자를 찾을 수 없습니다.");
                }
                SiteUser siteUser = _siteUser.get();
                List<GrantedAuthority> authorities = new ArrayList<>();
                if ("admin".equals(username)) {
                    authorities.add(new SimpleGrantedAuthority(UserRole.ADMIN.getValue()));
                } else {
                    authorities.add(new SimpleGrantedAuthority(UserRole.USER.getValue()));
                }
                return new User(siteUser.getUsername(), siteUser.getPassword(), authorities);
            }
        }
        ```
        
        - 스프링 시큐리티는  `loadUserByUsername` 메서드에 의해 리턴된 `User` 객체의 비밀번호가 사용자로부터 입력받은 비밀번호와 일치하는지 검사하는 기능을 내부에 가지고 있음.

- 스프링 시큐리티 설정 수정하기
    - `AuthenticationManager` bean 생성.
        - 사용자 인증 시 앞에서 작성한  `UserSecurityService`와 `PasswordEncoder`를 내부적으로 사용하여 인증과 권한 프로세스 처리.
        
- 로그인 화면 수정하기
    - 로그인 링크를 내비게이션 바에 추가.
    - navbar.html 템플릿 수정.
    
- 로그인 후 내비게이션 바에 ‘로그인’이 아닌 ‘로그아웃’ 링크가 표시되어야 함. 반대로 로그아웃 상태에서는 ‘로그인’ 링크로 바뀌어야 함.
    - 스프링 시큐리티의 타임리프 확장 기능을 사용하여 사용자의 로그인 상태를 확인해야 함.
        - `sec:authorize="isAnonymous()"`: 로그인되지 않은 경우에 해당 요소(로그인 링크) 표시됨.
        - `sec:authorize="isAuthenticated()"`: 로그인된 경우에 해당 요소(로그아웃 링) 표시됨.
            - `sec:authorize` 속성은 로그인 여부에 따라 요소를 출력하거나 출력하지 않게 함.
            
- 로그아웃 기능 구현하기
    - 이 역시 스프링 시큐리티 사용.
    - `SecurityConfig` 파일을 수정