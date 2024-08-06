# 스프링 JdbcTemplate

스프링 JdbcTemplate는 jdbc에서 중복 제거하고 만든 라이브러리임.

실무에서도 많이 사용함.  

- 순수한 jdbc와 동일한 환경 설정 → build.gradle에서 implement 기존과 동일하게.
- 스프링 JdbcTemplate과 MyBatis 같은 라이브러리는 JDBC API에서 본 반복 코드를 대부분 제거해준다. 하지만 SQL은 직접 작성해야 한다.

```java
package hello.hello_spring.repository;

import hello.hello_spring.domain.Member;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.simple.SimpleJdbcInsert;

import javax.sql.DataSource;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;

public class JdbcTemplateMemberRepository implements MemberRepository {

    private final JdbcTemplate jdbcTemplate; // insertion 받을 수 없다.

    //@Autowired 생성자 하나면 생략 가능
    public JdbcTemplateMemberRepository(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }

    @Override
    public Member save(Member member) { // Member 객체를 DB의 member table에 저장하는 method + DB에서 자동 생성된 id값을 member 객체에 설정한 후 반환.
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate); //SimpleJdbcInsert 객체 생성. 간단한 SQL 삽입 작업을 수행하기 위한 도우미 클래스.
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id"); // 삽입 작업이 수핼될 테이블 이름을 member로 지정. usingGeneratedKeyColumns("id")는 데이터베이스에서 자동 생성된 키 컬럼(이 경우 id)을 지정.삽입된 레코드의 자동 생성된 키 값을 반환받기 위해 필요.

        Map<String, Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());//Member 객체의 name 값을 parameters 맵에 추가. 이 값은 데이터베이스의 member 테이블의 name 컬럼에 삽입됨.

        Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters)); //jdbcInsert 객체를 사용하여 삽입 작업을 실행하고, 삽입된 레코드의 자동 생성된 키 값을 반환받는다.
        member.setId(key.longValue()); //반환된 키 값을 Member 객체의 id 필드에 설정.

        return member;
    }

    @Override
    public Optional<Member> findById(Long id) { //주어진 id를 사용하여 데이터베이스에서 Member 객체를 찾는 메서드. 반환 타입은 Optional<Member>입니다. 이는 찾은 Member 객체를 감싸서 반환하며, null일 가능성을 안전하게 처리
        List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id); // jdbcTemplate을 사용하여 SQL 쿼리를 실행. "select * from member where id = ?" 쿼리는 member 테이블에서 주어진 id와 일치하는 행을 선택.memberRowMapper()는 결과 집합을 Member 객체로 매핑하는 RowMapper를 반환.쿼리 결과는 Member 객체의 리스트로 저장됨.
        return result.stream().findAny(); // List 객체를 스트림(자바 8, 컬렉션 데이터를 저리하기 위한 다양한 메서드 제공)으로 변환. findAny()는 스트림에서 임의의 요소를 반환하는 메소드. 첫번째 요소 또는 병렬 처리 시 가장 먼저 발견된 요소 반환. 스트림이 비어있는 경우 빈 Optional 반환.
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = jdbcTemplate.query("select * from member where name = ?", memberRowMapper(), name);
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return jdbcTemplate.query("select * from member", memberRowMapper()); //query 메서드는 결과 집합의 각 행을 Member 객체로 매핑하여 List<Member> 형태로 반환.
    }

    //RowMapper는 데이터베이스의 결과 집합(ResultSet)을 객체로 매핑하는 역할을 합니다. 이 경우, ResultSet의 각 행을 Member 객체로 변환
    // 이 코드는 RowMapper<Member> 인터페이스를 구현하는 익명 클래스를 반환하는 메서드
    private RowMapper<Member> memberRowMapper(){
        return (rs, rowNum) -> {
            Member member = new Member();
            member.setId(rs.getLong("id"));
            member.setName(rs.getString("name"));
            return member;
        }; // mapping -> 객체 생성에 대한 것을 callback을 정함.
        /*return new RowMapper<Member>(){ //익명 클래스를 사용하여 RowMapper<Member> 인터페이스를 구현하고, 이 객체를 반환
            @Override
            public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                return member;
            }*/
    }
}

```

- 이후 SpringCofing에 조립
- 웹에서 확인할 필요없이 통합 테스트 돌려보면 된다
- 테스트 코드 잘 짜는 것이 중요.