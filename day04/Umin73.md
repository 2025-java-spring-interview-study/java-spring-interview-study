# Spring과 API 설계 및 통신

## 1. REST API란 무엇인가요? RESTful 하다는 것은 무슨 의미인가요?
REST API란?  
REST 원칙을 잘 지키면서 HTTP 프로토콜을 기반으로 리소스를 처리하는 API

### REST의 6가지 원칙
1. 클라이언트-서버 구조  
    - **클라이언트와 서버의 역할을 분리**해야 함  
    - 클라이언트는 서버 내부 구조를 알 필요 없음  

2. 무상태성(Stateless)  
    - 서버는 요청 간 **클라이언트 상태 저장X**
    - 필요한 정보는 토큰, 파라미터 등에 포함

3. 캐시 처리 가능
    - 응답 데이터는 **캐싱**이 가능해야 함

4. 계층화 구조
    - 클라이언트는 프록시, 게이트웨이 등의 **중간 계층이 존재해도 이를 모른 채 서버로 요청** 가능

5. 일관된 인터페이스
    - **HTTP 메서드, URI, 응답 구조**가 일관되계 설계

6. 리소스 기반
    - URI는 무엇을 다룰지(리소스)를 중심에 두고
    - 어떻게 다룰지는 HTTP 메서드로 표현

  
## 2. HTTP 메서드(GET, POST, PUT, DELETE 등)는 각각 어떤 상황에서 사용해야 할까요?
| 메서드 | 의미       | 멱등성 | 사용 상황          |
| ------ | ---------- | ------ | ------------------ |
| `GET`    | 조회       | O      | 데이터 가져올 때   |
| `POST`   | 생성       | X      | 새 리소스 등록     |
| `PUT`    | 전체 수정  | O      | 리소스 전체 교체   |
| `PATCH`  | 부분 수정  | △      | 일부 필드만 수정   |
| `DELETE` | 삭제       | O      | 리소스 제거        |


## 3. HTTP 상태 코드 중 200, 404, 500의 차이와 각각의 의미를 설명해주세요.

### 상태 코드 번호대 별 의미
| 그룹  | 의미                                                  |
| ----- | ----------------------------------------------------- |
| `1xx` | 정보 전달 (일반적으로 사용 안 함)                      |
| `2xx` | 성공 (정상 응답)                                       |
| `3xx` | 리다이렉션 (클라이언트가 다른 주소로 이동해야 함)      |
| `4xx` | 클라이언트 오류 (요청 자체의 문제)                      |
| `5xx` | 서버 오류 (서버 처리 중 문제 발생)                      |

### 상태 코드 상세
| 코드  | 이름                    | 의미                                           | 사용 예시                                   |
| ----- | ----------------------- | ---------------------------------------------- | ------------------------------------------- |
| `200` | OK                      | 요청이 성공적으로 처리됨                       |                                              |
| `201` | Created                 | 리소스가 성공적으로 생성됨                     | POST로 등록, 작성 등                        |
| `204` | No Content              | 요청 성공. 하지만 응답 본문 없음               | 삭제 요청 등                                |
| `400` | Bad Request             | 잘못된 요청(문법 오류, 유효성 실패 등)         | 누락된 필드, 잘못된 JSON 형식               |
| `401` | Unauthorized            | 인증이 필요하거나 인증 실패                   | 토큰 누락/만료, 로그인 필요                 |
| `403` | Forbidden               | 인증은 됐지만 권한 없음                        | 일반 사용자가 관리자 기능 요청 시           |
| `404` | Not Found               | 요청한 리소스를 찾을 수 없음                   | 잘못된 URL, 없는 ID 조회                    |
| `409` | Conflict                | 요청이 현재 상태와 충돌                        | 중복 데이터 삽입, 리소스 충돌               |
| `500` | Internal Server Error   | 서버 내부 오류                                 | NullPointerException, DB 연결 오류 등       |


## 4. Spring Security를 사용하여 JWT 기반 인증을 구현한 경험을 설명해 주세요.
1. 로그인 요청
```java
POST /api/auth/login
Body: { "userId": "umin", "password": "1234" }
```
- 사용자가 로그인 요청
- 서버에서 사용자 인증 성공 -> Access Token ( + Refresh Token) 발급
- Refresh Token은 Redis에 저장
- 클라이언트에 토큰 + 사용자 정보 전달  

2. 인증된 API 요청
```java
GET /api/members/my
Header: Authorization: Bearer <Access Token>
```
- 로그인 이후 모든 API 요청에 Access Token 포함
- 서버는 **토큰으로 사용자 식별 및 인증 여부** 판단

3. JwtAuthenticationFilter 동작
- 요청 URI가 **화이트리스트**에 포함되는지 확인  
  -> YES: 필터 통과  
  -> NO: 토큰 검사 진행
- Authorization 헤더에서 **JWT** 추출
- 토큰 유효성 검사
  - `jwtTokenProvider.validateToken()`
- 사용자 식별자 추출
  - `uid = jwtTokenProvider.getUid(token)`
- DB 조회로 사용자 정보 로드
  - `userDetails = customUserDetailService.loadUserByUsername(uid)`
- 인증 객체 생성 후 SecurityContext에 등록
  - `UsernamePasswordAuthenticationToken` 생성
  - `SecurityContextHolder.getContext().setAuthentication(auth)`

4. 컨트롤러 진입
- 인증이 완료된 경우 컨트롤러 진입 가능
- `@AuthenticationPrincipal`등을 통해 사용자 정보 바로 주입 가능
```java
ResponseEntity<SuccessResponse<?>> createCategory(
    @AuthenticationPrincipal Users user, ...
) {
    // 로직
}
```

5. 응답 반환
- 컨트롤러에서 처리 후 JSON 등으로 응답 반환
- 클라이언트는 인증된 상태에서 정상 응답 수신

## 5. OAuth 2.0을 사용한 인증 방식에서 Access Token과 Refresh Token의 차이점과 사용 사례를 설명해 주세요.
- OAuth는 사용자가 직접 로그인 정보를 서비스에 제공하지 않고, 다른 서비스(구글, 카카오 등)가 대신 인증해주는 방식
- 즉, 서비스가 외부 리소스 API에 토큰 기반으로 접근할 수 있도록 허용하는 인증/인가 프로토콜

### 구글 로그인 흐름
| 단계 | 설명                                                              |
| -- | --------------------------------------------------------------- |
| 1  | 클라이언트 → 서비스에서 **구글 로그인 버튼 클릭**                             |
| 2  | 구글 로그인 및 동의 화면에서 권한 승인                                          |
| 3  | 구글 → 서비스로 **Authorization Code** 전달                        |
| 4  | 서비스 → 구글 인증 서버에 **Access Token 요청** (Refresh Token도 함께 발급) |
| 5  | 서비스 → Access Token으로 **구글 사용자 정보 API 요청**                  |
| 6  | 사용자 정보 기반 회원가입/로그인 처리, JWT 발급 후 클라이언트에 응답                       |

### Access Token vs Refresh Token
| 항목          | Access Token        | Refresh Token     |
| ----------- | ------------------- | ----------------- |
| **목적**      | API 요청 시 인증         | Access Token 재발급  |
| **수명**      | n분 ~ n시간      | n일 ~ n주       |
| **저장 위치**   | 클라이언트에 저장 및 요청 시 포함 | 서버(또는 Redis)에 저장  |
| **노출 시 위험** | 즉시 악용 가능            | 무한 재발급 가능, 더 위험   |
| **사용 시점**   | API 호출 시            | Access Token 만료 시 |
