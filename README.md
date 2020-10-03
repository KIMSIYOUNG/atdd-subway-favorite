# 지하철 회원 관리 및 즐겨찾기

## 미션 회고 및 학습 내용
### 미션 포인트 ⏰ 
   - 문서 자동화(API 문서화)를 학습한다.
   - 로그인(권한)을 관리하는 방법을 학습한다.
        - 사용자를 관리하는 방법을 학습한다.
            - 세션
            - 토큰
   - 스프링 MVC의 동작원리에 대해 파악한다.
   - 도메인의 관계에 대해 고민한다(Line, Station, LineStation, Member, Favorite)

### 학습 내용 📖
   - 스프링 MVC 아래의 동작원리를 개념적으로 파악함.
        - Filter -> DispatcherServlet -> CommonService -> HandlerMapping -> HandlerInterceptor -> Handler -> Exception | ViewResolving | JsonResponse 
   ![스크린샷 2020-10-03 오후 8 52 30](https://user-images.githubusercontent.com/49060374/94990941-a11cb080-05ba-11eb-84ce-684c53f8eb9f.png)
   - Interceptor와 ArgumentResolving을 통한 권한 관리 및 Resolving
   ```java
   @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String credential = authExtractor.extract(request, "Bearer");
        if (!jwtTokenProvider.validateToken(credential)) {
            throw new InvalidAuthenticationException("유효하지 않은 토큰입니다.");
        }
        String email = jwtTokenProvider.getSubject(credential);

        request.setAttribute("loginMemberEmail", email);
        return true;
    }

   ```
   - ConstraintValidator을 통해, Custom Validator를 학습
   ```java
    @Component
    public class EmailDuplicationValidator implements ConstraintValidator<EmailMatcher, String> {
        private final MemberService memberService;

        public EmailDuplicationValidator(MemberService memberService) {
            this.memberService = memberService;
        }

        @Override
        public boolean isValid(String value, ConstraintValidatorContext context) {
            return !memberService.isExistMember(value);
        }
    }
   ```
   - 유일하게 식별되어야 하며, 관심사가 다른 경우 엔티티로 분리하는 것이 바람직하다고 생각.(Favorite 엔티티로 분리)
   - 토큰을 사용하여 무상태성 및 서버 부하를 줄이면서 권한을 관리할 수 있음.
   - Spring Rest Docs를 활용하여 테스트 기반의 문서화를 학습함.

### 아직 어려운 개념들 😂
   - Spring Data JDBC를 사용하다보니, 항상 데이터를 가져와서 Stream을 돌고 다시 다른 데이터를 가져오기 위한 요청을 보내야함.
   - Join을 통해 한번에 DTO로 가져오는 것이 좋을까, 지금처럼 분리해서 가져오는 것이 좋을지 고민중인데 쉽게 답이 나오지 않음.
        - 서버가 분리되는 경우 각 API에 요청을 보내는 형태일 확률이 있으니 지금처럼 하는게 맞다고 생각함.
        - 다만 지금 서버가 분리되는 경우를 생각해야하나? 싶음.
        - 서버가 분리되더라도 데이터 파이프라인이 한 곳으로 집중된다면 조인과 비슷한 형태로 사용할 수 있을 것 같기도 함(예상)
        - 결론 - 한번에 가져오는게 성능상에 조금 더 좋아보임. 다만 이렇게 가져오기 시작하면 진입점이 모호해져서 유지보수의 복잡도가 올라갈 수 있다고 생각함.
   
---

<br/>

## 1단계 회원관리 기능

### 요구사항

- [x] 회원 정보를 관리하는 기능 구현
- [x] 자신의 정보만 수정 가능하도록 해야하며 로그인이 선행되어야 함
- [x] 토큰의 유효성 검사와 본인 여부를 판단하는 로직 추가
- [x] side case에 대한 예외처리 
- [x] 인수 테스트와 단위 테스트 작성
- [x] API 문서를 작성하고 문서화를 위한 테스트 작성
- [x] 페이지 연동

### 기능목록

- [x] 회원가입
    - [x] 비밀번호 확인
- [x] 로그인
- [x] 회원정보 조회
- [x] 수정
- [x] 삭제

## 2단계 즐겨찾기

### 요구사항

- [x] 즐겨찾기 기능을 추가(추가,삭제,조회)
- [x] 자신의 정보만 수정 가능하도록 해야하며 로그인이 선행되어야 함
- [x] 토큰의 유효성 검사와 본인 여부를 판단하는 로직 추가(interceptor, argument resolver)
- [x] side case에 대한 예외처리 필수
- [x] 인수 테스트와 단위 테스트 작성
- [x] API 문서를 작성하고 문서화를 위한 테스트 작성
- [x] 페이지 연동

### TO-DO

- [x] 즐겨찾기 인수테스트
    - [x] 회원가입
    - [x] 로그인
    - [x] 즐겨찾기 추가
    - [x] 즐겨찾기 조회
    - [x] 즐겨찾기 삭제
- [x] 즐겨찾기 단위테스트
    - [x] 예외처리
        - [x] 동일 경로
- [x] 즐겨찾기 문서화
- [x] 페이지 연동

### 추가 TO-DO

- [x] 즐겨찾기 중복 예외 처리
- [x] 나의 정보
    - [x] 이름만 수정가능
    - [x] 수정을 위해선 비밀번호를 입력해야함.
    - [x] 이메일은 변경 불가.
