# Java 심화 및 JPA
### 1. 의존성 주입이란 무엇이며, 이를 적용했을 때의 장점을 설명해주세요
<h3>의존성 주입이란</h3>
<p>객체 간의 의존 관계를 외부에서 주입해주는 설계 패턴<br/>-> 클래스 간 결합도 낮추고, 유연하게 테스트 가능해짐</p>

<h3>의존성 주입의 장점</h3>

<b>강한 결합</b>
<pre><code>GmailEmailSender emailSender = new GmailEmailSender();</code></pre>
<ul>
  <li>클래스 내부에서 구체적인 구현체를 직접 생성</li>
  <li>구현체가 변경되면 <b>의존하는 클래스도 함께 수정</b>해야 함</li>
</ul>

<hr>

<b>약한 결합</b>
<pre><code>public class NotificationService {
    private final EmailSender emailSender;

    public NotificationService(EmailSender emailSender) {
        this.emailSender = emailSender;
    }
}
</code></pre>
<ul>
  <li>인터페이스(추상화)에 의존 → 구현체는 외부에서 주입</li>
  <li>예: GmailEmailSender → NaverEmailSender 교체 시 <b>코드 수정 없이 객체만 변경 가능</b></li>
    <li><b>유지보수 용이</b>  
    <ul>
      <li>구현체가 바뀌어도 의존 클래스는 그대로 사용 가능</li>
    </ul>
  </li>
  <li><b>테스트 편의성</b>  
    <ul>
      <li>Mock 객체, Stub 객체 등 테스트용 구현체 주입 가능</li>
      <li>단위 테스트 작성이 쉬움</li>
    </ul>
  </li>
</ul>

<br/>

### 2. 디자인 패턴 중 싱글톤 패턴을 설명하고, 실제 구현할 때 고려해야 할 사항을 설명하세요

싱글톤 패턴은 <b>인스턴스 하나만 생성</b>해서 공유하는 방식 <br/>
-> 메모리 사용 줄이고, 전역 접근이 필요한 객체 안전하게 관리 가능

싱글톤 패턴은
<ul>
<li>설정 클래스</li>
<li>로깅 서비스</li>
<li>DB 연결 객체</li>
<li>스프링 빈</li>
</ul>
등에 적용할 수 있음 <br/><br/>

<table border="1" cellpadding="8" cellspacing="0">
  <thead>
    <tr>
      <th>이슈</th>
      <th>문제</th>
      <th>해결방법</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>생성자 접근 제한</strong><br>(외부에서 new 금지)</td>
      <td>
        싱글톤은 인스턴스가 1개여야 하는데,<br>
        new로 여러 개 생성되면 의미가 없음
      </td>
      <td>
        생성자를 <code>private</code>으로 선언해 외부에서 인스턴스 생성을 막음
      </td>
    </tr>
    <tr>
      <td><strong>멀티스레드 환경</strong><br>(동시 getInstance() 호출)</td>
      <td>
        지연 초기화 시, 두 스레드가 동시에 null을 보고<br>
        각각 인스턴스를 생성할 수 있음
      </td>
      <td>
        <ul>
          <li><code>synchronized</code> 키워드로 동기화</li>
          <li>Double-Checked Locking 패턴</li>
          <li>클래스 로딩 시 초기화 방식 사용
            <pre><code>public class Singleton {
  private static final Singleton instance = new Singleton();
  public static Singleton getInstance() {
    return instance;
  }
}</code></pre>
          </li>
        </ul>
      </td>
    </tr>
    <tr>
      <td><strong>클래스 로더 문제</strong></td>
      <td>
        JVM 내에서 여러 클래스 로더가 존재할 수 있으며,<br>
        서로 다른 로더는 동일 클래스를 다르게 인식 → <br>
        서로 다른 인스턴스 생성 가능
      </td>
      <td>
        일반 Java/Spring에서는 문제 적음<br>
        WAS 환경에서는 주의 필요
      </td>
    </tr>
    <tr>
      <td><strong>직렬화 시 새로운 인스턴스 생성</strong></td>
      <td>
        직렬화된 객체를 역직렬화하면<br>
        <strong>새로운 인스턴스가 생성</strong>됨
      </td>
      <td>
        <code>readResolve()</code> 메서드 오버라이딩
        <pre><code>private Object readResolve() {
    return getInstance();
}</code></pre>
      </td>
    </tr>
    <tr>
      <td><strong>테스트 어려움</strong></td>
      <td>
        싱글톤은 <strong>앱 전체에서 공유되는 객체</strong><br>
        → 테스트 간 상태 공유되어 결과에 영향
      </td>
      <td>
        <ul>
          <li>싱글톤에 상태(state)를 두지 않는 설계</li>
          <li>테스트 전후 초기화 코드 작성</li>
          <li>DI 기반 구조 사용 (테스트 용이)</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

<Br/>

### 3. synchronized는 어떤 방식으로 동작하나요?
<h3>synchronized란?</h3>
<p>
  Java에서 <b>멀티스레드 환경에서의 동기화</b>를 보장하기 위한 키워드<br>
  여러 스레드가 동시에 접근할 수 있는 <b>임계 영역</b>에 대해<br>
  <b>동시에 하나의 스레드만 접근</b>할 수 있도록 제어함
</p>

<h3>동작 방식 요약</h3>
<ol>
  <li>
    <b>Monitor Lock 획득</b><br>
    모든 Java 객체에는 모니터 락이 존재<br>
    스레드가 <code>synchronized</code> 블록이나 메서드에 진입할 때 해당 객체 또는 클래스의 모니터 락을 획득함
  </li>
  <li>
    <b>BLOCKED 상태 대기</b><br>
    이미 다른 스레드가 락을 보유 중 -> 나머지 스레드는 락이 해제될 때까지 BLOCKED 상태로 대기
  </li>
  <li>
    <b>락 자동 해제</b><br>
    락을 가진 스레드가 블록을 빠져나오면, 락은 자동으로 해제 -> 다음 스레드 진입 가능
  </li>
</ol>

<h3>Monitor Lock이란?</h3>
<ul>
  <li>자바에서 동기화를 구현하기 위한 핵심 장치</li>
  <li><b>객체 단위로 존재</b>하며, 객체마다 하나의 모니터 락이 존재</li>
  <li>동시에 하나의 스레드만 해당 락을 획득할 수 있음</li>
  <li><code>synchronized</code> 키워드는 이 락을 기반으로 동기화를 수행</li>
</ul>

<br/>

### 4. 고아 객체(Orphan Removal)는 무엇이고, 언제 사용하는가요?
<h3>고아 객체란?</h3>
<ul>
<li>JPA에서 연관된 자식 엔티티가 부모 엔티티로부터 제거되었을 때, 해당 자식 엔티티를 자동으로 DB에서 삭제해주는 기능</li>
<li><code>@OneToMany</code> 또는 <code>@OneToOne</code> 관계에서 이 <code>orphanRemoval=true</code> 옵션을 설정해야 함</li>
</ul>

<h3>언제 사용하는가?</h3>
<ul>
<li>부모-자식 간 생명주기를 같이 관리하고 싶을 때(자식이 부모에서 제거되면 자동 삭제)</li>
<li>ex- 댓글, 첨부파일, 장바구니 항목 등...</li>
</ul>

<br/>

### 5. 양방향 연관관계에서 순환 참조가 발생할 수 있는데, 이를 어떻게 해결했나요?

이러한 경우 주로 DTO를 분리해서 처리하는 방식으로 해결했었습니다.
DTO를 분리하는 방식으로 해결하면 필요한 정보만 전달하여 순환 참조를 막을 수 있고, 응답 구조도 명확하게 관리할 수 있었기 때문입니다.
또, 상황에 따라 <code>@JsonIgnore</code>도 사용했었습니다.

<br/>

### 6. JPA에서 락을 거는 방법은 어떤 것이 있나요?
<table border="1" cellpadding="8" cellspacing="0">
  <thead>
    <tr>
      <th>항목</th>
      <th>비관적 락</th>
      <th>낙관적 락</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>동작 방식</td>
      <td>데이터를 먼저 락으로 선점해서 다른 트랜잭션 차단</td>
      <td>먼저 작업 후 커밋 시 충돌 여부 검사</td>
    </tr>
    <tr>
      <td>장점</td>
      <td>
        - 충돌 완전 방지 가능<br>
        - 정합성 우선 환경에 적합
      </td>
      <td>
        - 락 사용 X → 성능 우수<br>
        - 데드락 없음
      </td>
    </tr>
    <tr>
      <td>단점</td>
      <td>
        - 성능 저하<br>
        - 데드락 가능성<br>
        - 읽기조차 락 소모
      </td>
      <td>
        - 충돌 시 롤백 필요<br>
        - 충돌 많으면 성능 저하
      </td>
    </tr>
    <tr>
      <td>사용 예시</td>
      <td>
        - 동시에 같은 데이터를<br>자주 수정하는 환경
      </td>
      <td>
        - 읽기 위주 시스템<br>
        - 충돌이 드문 다중 사용자 환경
      </td>
    </tr>
    <tr>
      <td>구현 방식</td>
      <td>DB 수준의 락<br>(예: SELECT FOR UPDATE)</td>
      <td>버전 필드 사용<br>(예: @Version)</td>
    </tr>
    <tr>
      <td>데드락</td>
      <td>발생 가능성 있음</td>
      <td>발생하지 않음</td>
    </tr>
  </tbody>
</table>