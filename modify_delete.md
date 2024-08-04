# 수정 및 삭제 기능

## 수정

- 질문 수정 위한 탬플릿 새로 만들지 않고 기존의 질문 등록 Form 사용해도 된다.
- 수정 시 html 파일의 <form> 태그의 th:action 속성을 삭제해야 함.
    - 이때 CSRF 값이 자동으로 생성되지 않아서 CSRF값을 설정하기 위해 hibben 형태로 input 요소를 작성하여 추가해야 함.
    - 스프링 시큐리티를 사용할 때 반드시 CSRF 값이 필요하기 때문에 수동으로라도 추가해야 함.
    
    ```html
     <form th:object="${questionForm}" method="post">
            <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
    ```
    
    - <form>태그의 action 속성없이 폼을 전송하면 action 속성이 없더라도 자동으로 현재 URL(웹 브라우저에 표시되는 URL 주소)을 기준으로 전송되는 규칙이 있음
        - 질문 등록 시 브라우저에 표시되는 URL은`/question/create`이어서 action 속성이 지정되지 않더라도 POST로 폼 전송할 때 action 속성으로 `/question/create`
        가 자동으로 설정됨.
        - 질문 수정 시에 브라우저에 표시되는 URL은 `/question/modify/2`와 같은 URL이기 때문에 POST로 폼 전송할 때 action 속성에 `/question/modify`와 같은 URL이 설정됨.
    
    ---
    

## 질문 삭제

- 질문 상세 화면에 [삭제] 버튼 추가
    - 
        
        ```html
        (... 생략 ...)
        <!-- 질문 -->
        <h2 class="border-bottom py-2" th:text="${question.subject}"></h2>
        <div class="card my-3">
            <div class="card-body">
                (... 생략 ...)
                <div class="my-3">
                    <a th:href="@{|/question/modify/${question.id}|}" class="btn btn-sm btn-outline-secondary"
                        sec:authorize="isAuthenticated()"
                        th:if="${question.author != null and #authentication.getPrincipal().getUsername() == question.author.username}"
                        th:text="수정"></a>
                    <a href="javascript:void(0);" th:data-uri="@{|/question/delete/${question.id}|}"
                        class="delete btn btn-sm btn-outline-secondary" sec:authorize="isAuthenticated()"
                        th:if="${question.author != null and #authentication.getPrincipal().getUsername() == question.author.username}"
                        th:text="삭제"></a>
                </div>
            </div>
        </div>
        (... 생략 ...)
        
        ```
        
    - [삭제] 버튼을 클릭하면 자바스트립트 코드가 실행되도록 구현.
    - [수정] 버튼과 달리 `href` 속성값을 `javascript:void(0)`로 설정하고 삭제를 실행할 URL을 얻기 위해 `th:data-uri` 속성을 추가한 뒤, [삭제] 버튼을 클릭하는 이벤트를 확인하기 위해 `class` 속성에 `delete` 항목을 추가.
        - `javascript:void(0)`: HTML에서 링크(<a> 태그)를 클릭했을 때 페이지가 이동하지 않도록 함. 링크 클릭 시 자바스크립트만 실행될 뿐 아무런 동작도 하지 않음.
        - `th:data-uri`: 타임리프 엔진에서 데이터 속성을 설정하기 위해 사용. 자바 스트립트에서 사용할 URL을 동적으로 설정할 수 있게 함. 특정 리소스를 삭제할 때 필요한 URL을 설정할 수 있음.
            - `data-uri` 속성에 설정한 값은 클릭 이벤트 발생 시 별도의 자바스크립트 코드에서 `this.dataset.uri`를 사용하여 그 값을 얻어 실행할 수 있음.
        - `delete`클래스가 있는 요소를 클릭하면 `th:data-uri`의 속성 URL로 삭제 요청을 보냄. (특정 리소스 삭제 가능)
    - [삭제] 버튼을 클릭했을 때 삭제 확인 메시지와 같은 별도의 확인 절차를 중간에 넣기 위해  `href`에 삭제를 위한 URL 직접 사용하지 않음.

### 삭제를 위한 자바스크립트

- 자바스크립트는 웹페이지에 동적인 기능을 추가할 때 사용하는 스크립트 언어. HTML, CSS와 함께 사용.
- 자바스크립트를 활용해 [삭제] 버튼을 클릭했을 때 ‘정말로 삭제하시겠습니까?’와 같은 메시지를 담은 확인 창 호출.

```jsx
<script type='text/javascript'>
const delete_elements = document.getElementsByClassName("delete");
Array.from(delete_elements).forEach(function(element) {
    element.addEventListener('click', function() {
        if(confirm("정말로 삭제하시겠습니까?")) {
            location.href = this.dataset.uri;
        };
    });
});
</script>

```

- const: 변수 상수화, 읽기 전용
- 화면 출력이 완료된 후에 자바스크립트가 실행되는 것이 좋기 때문에 자바스크립트 코드는 </body> 태그 바로 위에 삽입하는 것이 좋음. → layout.html 수정하면 상속하는 템플릿은 자바스크립트 삽입 위치 신경 쓰지 않아도 된다.

### 질문 서비스와 컨트롤러 수정

- 서비스에 질문 데이터 삭제하는 delete 메서드 추가
- 컨트롤러에 [삭제] 버튼 클릭 시 삭제 URL처리할 수 있도록 메서드 추가
    
    ```java
    @PreAuthorize("isAuthenticated()")
        @GetMapping("/delete/{id}")
        public String questionDelete(Principal principal, @PathVariable("id") Integer id) {
            Question question = this.questionService.getQuestion(id);
            if(!question.getAuthor().getUsername().equals(principal.getName())) {
                throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "삭제권한이 없습니다.");
            }
            this.questionService.delete(question);
            return "redirect:/";
        }
    }
    ```
    

---

## 답변 삭제 및 수정

- 비슷한 방법으로 답변 수정 및 삭제 기능 추가
- 답변 수정 기능을 위한 템플릿이 따로 없으므로 답변 수정 시 사용할 템플릿이 추가로 필요.

---

- Spring MVC에서 컨트롤러 메서드를 정의할 때, 매개변수의 순서는 실제로 중요하지 않습니다. Spring MVC는 매개변수의 타입과 어노테이션을 기반으로 적절한 값을 주입하기 때문에, 매개변수의 순서가 메서드의 동작에 영향을 미치지 않습니다.

---

## 수정 일시 표시하기

- 질문 상세 화면에 수정 일시가 나타나도록 기능 추가

- question_detail.html 수정

```html
(... 생략 ...)
<!-- 질문 -->
<h2 class="border-bottom py-2" th:text="${question.subject}"></h2>
<div class="card my-3">
    <div class="card-body">
        <div class="card-text" style="white-space: pre-line;" th:text="${question.content}"></div>
        <div class="d-flex justify-content-end">
            <div th:if="${question.modifyDate != null}" class="badge bg-light text-dark p-2 text-start mx-3">
                <div class="mb-2">modified at</div>
                <div th:text="${#temporals.format(question.modifyDate, 'yyyy-MM-dd HH:mm')}"></div>
            </div>
            <div class="badge bg-light text-dark p-2 text-start">
                <div class="mb-2">
                    <span th:if="${question.author != null}" th:text="${question.author.username}"></span>
                </div>
                <div th:text="${#temporals.format(question.createDate, 'yyyy-MM-dd HH:mm')}"></div>
            </div>
        </div>
        (... 생략 ...)
    </div>
</div>
(... 생략 ...)
<!-- 답변 반복 시작 -->
<div class="card my-3" th:each="answer : ${question.answerList}">
    <div class="card-body">
        <div class="card-text" style="white-space: pre-line;" th:text="${answer.content}"></div>
        <div class="d-flex justify-content-end">
            <div th:if="${answer.modifyDate != null}" class="badge bg-light text-dark p-2 text-start mx-3">
                <div class="mb-2">modified at</div>
                <div th:text="${#temporals.format(answer.modifyDate, 'yyyy-MM-dd HH:mm')}"></div>
            </div>
            <div class="badge bg-light text-dark p-2 text-start">
                <div class="mb-2">
                    <span th:if="${answer.author != null}" th:text="${answer.author.username}"></span>
                </div>
                <div th:text="${#temporals.format(answer.createDate, 'yyyy-MM-dd HH:mm')}"></div>
            </div>
        </div>
        (... 생략 ...)
    </div>
</div>
<!-- 답변 반복 끝  -->
(... 생략 ...)

```