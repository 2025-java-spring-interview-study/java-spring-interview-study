# 배포 및 운영

## 1. 프로젝트 하면서 배포 자동화(CI/CD)는 어떻게 구성했나요?
먼저, main, develop, 이슈별 브랜치 전략으로 관리
- main 브랜치: 실제 배포 브랜치
- develop 브랜치: 통합 테스트 및 기능 점검

main 브랜치에서 push 발생 -> Github Actions가 자동으로 실행

### Workflow 동작 방식
1. 환경 세팅
    - JDK 버전 세팅
    - Gradle 빌드 실행
2. Docker 이미지 생성 및 배포 준비
    - 빌드된 애플리케이션을 Docker 이미지로 패키징
    - AWS ECR에 push
3. EC2 서버 배포 자동화
    - EC2 서버로 SSH 접속
    - 최신 Docker 이미지 pull
    - `docker compose down` 후 `docker compose up -d` 실행


## 2. DB 컨테이너의 데이터는 컨테이너가 내려가면 사라지는데, 이를 어떻게 영속성 있게 보장했나요?
프로젝트에서 PostgreSQL 컨테이너가 내려가더라도 데이터 유실되지 않도록 **호스트 디렉터리 마운트 방식** 적용

### 데이터 볼륨 설정
`docker-compose.yml`에서 PostgreSQL 데이터 디렉터리를 호스트의 `./postgres_data` 폴더와 연결
```
services:
  db:
    image: postgres:15
    volumes:
      - ./postgres_data:/var/lib/postgresql/data
```
-> 실제 데이터가 호스트의 `postgres_data` 폴더에 남아 있음  
-> 컨테이너 삭제하거나 이미지 새로 빌드 해도 괜찮음

### 운영 환경 안정성
```
restart: unless-stopped
```
-> 옵션을 주어 컨테이너 비정상 종료되더라도 자동으로 재시작, 데이터 유실 방지


## 배포 시점에서 테스트 자동화는 어떻게 적용했나요?
### 테스트 및 배포 검증
1. 환경 세팅: JDK 설치
2. 빌드: Gradle 빌드 수행
3. 테스트 실행:
    - 서비스 로직은 **JUnit**으로 검증
    - 컨트롤러 계층은 **MockMvc**를 활용해 요청-응답 동작 확인
4. 배포 중단 조건:
    - 테스트 실패 시 배포 프로세스가 자동으로 중단


## 장애 발생 시 로그를 어떻게 추적하나요?
애플리케이션 로그는 컨테이너 내부 경로를 호스트 디렉터리에 마운트 하는 방식으로 관리
`docker-compose.yml`에서
```
volumes:
  - .shared_logs:/app/shared_logs
```
이런 식으로 설정
-> 로그 파일이 컨테이너 외부(EC2 서버나 로컬PC)에 저장, 컨테이너 내려가도 데이터 보존

### 로그 기록 방식
- **Slf4j** 기반 로깅 사용
- GlobalExceptionHandler를 통해 예외 발생 시 공통된 포맷으로 기록

### 장애 추적
- EC2 서버에 SSH 접속 후 로그 파일 직접 확인 또는 docker logs 명령어 활용

### ELK 미사용 이유
- 소형 인스턴스 사용 중 -> 메모리/CPU 리소스 부족 문제 발생
  -> ELK 대신 Slf4j + 볼륨 마운트 방식 활용

### Q. 실제로 어떤 문제를 해결했는지??
SSE 기반 알림 기능 구현했을 때, 커넥션 수 늘어나면서 커넥션 풀 자원이 빠르게 소모되는 문제 발생했었습니다. 로그를 통해 emitter 키가 중복되어 동일한 하용자가 여러 커넥션을 열고 있었고, 불필요한 DB 조회도 같이 발생한다는 것을 확인 했습니다. 이후 emitter 키 구조를 변경해서 동일 사용자 중복 연결을 막았고, 불필요한 DB 조회를 제거해 커넥션 부담을 줄였었습니다.