# Java 기초 및 JVM 구조
### 1. Java 코드가 실행되기까지 어떤 과정을 거치는지?
<h4>1. 컴파일 단계</h4>
<ul>
  <li>.java 확장자의 소스 파일을 작성</li>
  <li>javac 컴파일러가 소스 파일을 <strong>바이트코드</strong>로 변환</li>
</ul>

<h4>2. 클래스 로딩 단계</h4>
<ul>
  <li>.class 파일이 JVM의 <strong>ClassLoader</strong>에 의해 메모리로 로딩</li>
  <li>클래스 정보는 <strong>메서드 영역</strong>에 저장</li>
</ul>

<h4>3. 실행 단계</h4>
<ul>
  <li>JVM의 <strong>실행 엔진</strong>이 바이트코드를 실행</li>
  <li>초기에는 <strong>인터프리터 방식</strong>으로 한 줄씩 읽고 실행</li>
  <li>자주 실행되는 코드는 <strong>JIT 컴파일러</strong>를 통해 <strong>기계어로 변환</strong>되어 성능 최적화</li>
</ul>

<h4>4. 메모리 관리</h4>
<p>JVM은 아래의 메모리 영역들을 사용</p>

<table border="1" cellspacing="0" cellpadding="6">
  <thead>
    <tr>
      <th>메모리 영역</th>
      <th>설명</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Heap</td>
      <td>객체가 저장되는 영역</td>
    </tr>
    <tr>
      <td>Stack</td>
      <td>메서드 호출 정보, 지역 변수 등이 저장되는 영역</td>
    </tr>
    <tr>
      <td>Method Area</td>
      <td>클래스 정보, static 변수, 상수 등이 저장됨</td>
    </tr>
    <tr>
      <td>PC Register, Native Method Stack</td>
      <td>JVM 내부적으로 사용하는 영역</td>
    </tr>
  </tbody>
</table>

<h4>5. Garbage Collection</h4>
<ul>
  <li>더 이상 참조되지 않는 객체는 <strong>GC</strong>가 자동으로 메모리를 해제</li>
  <li>개발자가 직접 <code>free()</code> 같은 메모리 해제 코드를 작성하지 않아도 됨</li>
</ul>

<br/>

### 2. Java에서 지역 변수와 인스턴스 변수는 각각 어느 메모리에 저장되며, 언제 생성되고 언제 사라지는지?
<table border="1" cellspacing="0" cellpadding="6">
  <thead>
    <tr>
      <th>항목</th>
      <th>지역 변수</th>
      <th>인스턴스 변수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>선언 위치</strong></td>
      <td>메서드 내부</td>
      <td>클래스 내부</td>
    </tr>
    <tr>
      <td><strong>저장 위치</strong></td>
      <td>Stack</td>
      <td>Heap</td>
    </tr>
    <tr>
      <td><strong>생성 시점</strong></td>
      <td>메서드 호출 시</td>
      <td>객체 생성 시</td>
    </tr>
    <tr>
      <td><strong>소멸 시점</strong></td>
      <td>메서드 종료 시</td>
      <td>객체가 GC에 의해 수거될 때</td>
    </tr>
    <tr>
      <td><strong>접근 방법</strong></td>
      <td>직접 변수명으로 사용</td>
      <td>객체명, 변수명으로 접근</td>
    </tr>
  </tbody>
</table>

<br/>

### 3. Java에서 예외(Exception)와 오류(Error)의 차이점은?
<table border="1" cellspacing="0" cellpadding="6">
  <thead>
    <tr>
      <th>항목</th>
      <th>Exception (예외)</th>
      <th>Error (오류)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>상속 계층</strong></td>
      <td>Throwable → Exception</td>
      <td>Throwable → Error</td>
    </tr>
    <tr>
      <td><strong>의미</strong></td>
      <td>개발자가 예측 가능하고 처리 가능한 문제</td>
      <td>시스템 또는 JVM의 치명적 문제</td>
    </tr>
    <tr>
      <td><strong>예시</strong></td>
      <td>NullPointerException, IOException 등</td>
      <td>OutOfMemoryError, StackOverflowError 등</td>
    </tr>
    <tr>
      <td><strong>처리 방식</strong></td>
      <td>try-catch, throws 등으로 직접 처리 가능</td>
      <td>대부분 처리 불가, 복구 어려움</td>
    </tr>
    <tr>
      <td><strong>주요 발생 위치</strong></td>
      <td>애플리케이션 로직 내부</td>
      <td>JVM, 런타임 시스템</td>
    </tr>
  </tbody>
</table>

<h4>Q. 예외 상황을 평소에 어떻게 처리하시나요?</h4>
<p>
  테스트 코드 작성 시, <strong>given - when - then</strong> 형식으로 시나리오를 분리해 예외 상황을 미리 정의하고 검증하는 편입니다.<br />
  특히 발생할 수 있는 예외 케이스를 <strong>when 절에서 명확히 트리거</strong>하고, <strong>then 절에서 적절한 예외 메시지나 HTTP 응답 코드 등을 검증</strong>합니다.
</p>

<br/>

### 4. main 메서드는 왜 public static void main(String[] args) 형식이어야 하는지?
<p><code>public static void main(String[] args)</code>는 Java 애플리케이션의 <strong>entry point</strong>로, JVM이 프로그램 실행 시 가장 먼저 호출해야 함</p>
<table border="1" cellspacing="0" cellpadding="6">
  <thead>
    <tr>
      <th>키워드</th>
      <th>의미</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>public</td>
      <td>JVM이 클래스 외부에서 메서드에 접근해야 하므로 <strong>접근 제한 없이 열려 있어야 함</strong></td>
    </tr>
    <tr>
      <td>static</td>
      <td>JVM이 객체를 생성하지 않고도 호출할 수 있도록 <strong>static 메서드</strong>로 선언</td>
    </tr>
    <tr>
      <td>void</td>
      <td><strong>JVM이 main 메서드의 반환값을 사용하지 않기 때문에</strong> 반환값이 없는 void로 지정</td>
    </tr>
    <tr>
      <td>String[] args</td>
      <td><strong>커맨드라인 인자</strong>를 받아 실행 시 외부 값을 전달할 수 있도록 설계된 매개변수</td>
    </tr>
  </tbody>
</table>

<br/>

### 5. 메서드를 반복 호출하면 Stack 영역에서는 어떤 일이 발생하는지?
<p>
Java에서 메서드를 호출할 때마다 JVM의 <strong>Stack</strong>에는 해당 메서드의 호출 정보가 담긴 <strong>Stack Frame</strong>이 쌓임
</p>

<ul>
  <li>스택 프레임에는 <strong>매개변수, 지역 변수, 리턴 주소</strong> 등의 정보가 포함</li>
  <li>메서드 실행이 끝나면 해당 스택 프레임은 제거</li>
</ul>

<p>
하지만 <strong>재귀 호출이나 반복 호출이 과도하게 발생</strong>하면 스택 프레임이 계속 쌓이게 되고,<br />
결국 JVM이 할당한 <strong>Stack 영역의 메모리 한계를 초과</strong>
</p>

<p>
→ <strong>StackOverflowError</strong> 발생
</p>

<br/>

### 6. String은 불변 객체인데, 그렇다면 문자열을 반복해서 연결할 때 어떤 문제가 생기고 어떻게 해결할 수 있을지?
<p>
Java에서 String 문자열을 수정하면 기존 문자열이 바뀌는 것이 아니라 <strong>새로운 문자열 객체가 생성</strong></p>

<pre><code>String str = "Hello";
str += " World";  // → "Hello"랑 " World"를 연결해 새로운 String 객체 생성</code></pre>

<p>
이렇게 <code>+</code> 연산을 반복하면 <strong>매번 새로운 객체가 Heap에 생성되고 버려지기 때문에</strong>,<br />
메모리 낭비와 GC 부하도 증가해 성능 저하로 이어질 수 있음
</p>

<h4>해결 방법: <code>StringBuilder</code> / <code>StringBuffer</code></h4>
<p>
<code>StringBuilder</code>나 <code>StringBuffer</code>는 내부적으로 <strong>가변 배열(char[])</strong>을 사용<br />
그래서 문자열을 수정할 때도 <strong>새 객체를 만들지 않고 기존 버퍼 내에서 수정</strong>되어 성능이 훨씬 좋음
</p>
<table border="1" cellspacing="0" cellpadding="6">
  <thead>
    <tr>
      <th>항목</th>
      <th>String</th>
      <th>StringBuilder</th>
      <th>StringBuffer</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>가변성</strong></td>
      <td>Immutable</td>
      <td>Mutable</td>
      <td>Mutable</td>
    </tr>
    <tr>
      <td><strong>스레드 안전성</strong></td>
      <td>고려할 필요 X</td>
      <td>X</td>
      <td>O (synchronized)</td>
    </tr>
    <tr>
      <td><strong>성능</strong></td>
      <td>느림 (객체 반복 생성)</td>
      <td>빠름</td>
      <td>다소 느림 (동기화 비용)</td>
    </tr>
    <tr>
      <td><strong>사용 용도</strong></td>
      <td>문자열 변경이 거의 없을 때</td>
      <td>단일 스레드에서 문자열 조작</td>
      <td>멀티스레드에서 문자열 조작</td>
    </tr>
  </tbody>
</table>

<br/>

### 7. 자바에서는 왜 모든 타입을 객체로 만들지 않고 기본형 타입을 따로 제공하는지?
<p>
Java는 <strong>성능과 메모리 효율성</strong>을 위해 기본형(primitive type)과 참조형(reference type)을 명확히 구분함
</p>

<h4>기본형 (Primitive Type)</h4>
<ul>
  <li><code>int</code>, <code>boolean</code>, <code>double</code> 등의 타입은 객체가 아닌 <strong>값 자체를 Stack 메모리에 저장</strong></li>
  <li>값을 직접 저장하므로 <strong>생성 비용이 없고, 접근 속도가 빠르며, 메모리 사용이 효율적</strong></li>
</ul>

<h4>모든 타입이 객체라면?</h4>
<ul>
  <li>단순한 숫자 연산이나 비교조차도 <strong>Heap 메모리 사용 + 객체 생성 비용</strong>이 발생</li>
  <li>그로 인해 <strong>GC 부담 증가</strong>, <strong>성능 저하</strong> 등의 문제가 생길 수 있음</li>
</ul>

<table border="1" cellspacing="0" cellpadding="6">
  <thead>
    <tr>
      <th>항목</th>
      <th>기본형</th>
      <th>참조형</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>예시</strong></td>
      <td><code>int</code>, <code>boolean</code>, <code>char</code>, <code>float</code> 등</td>
      <td><code>String</code>, <code>Integer</code>, <code>Object</code>, <code>Array</code> 등</td>
    </tr>
    <tr>
      <td><strong>저장 위치</strong></td>
      <td>Stack 메모리</td>
      <td>Heap 메모리 (참조는 Stack)</td>
    </tr>
    <tr>
      <td><strong>메모리 사용량</strong></td>
      <td>적음 (값만 저장)</td>
      <td>많음 (객체 구조 + 참조 주소 저장)</td>
    </tr>
    <tr>
      <td><strong>속도</strong></td>
      <td>빠름</td>
      <td>느림</td>
    </tr>
    <tr>
      <td><strong>객체 여부</strong></td>
      <td>아님</td>
      <td>맞음</td>
    </tr>
    <tr>
      <td><strong>null 가능 여부</strong></td>
      <td>X</td>
      <td>O</td>
    </tr>
  </tbody>
</table>