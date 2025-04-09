---
title: "Test Driven Development with JUnit5"
excerpt: "[Spring] Test Driven Development with JUnit5 Part1"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2025-04-08
---


### 들어가면서
  - 팀의 개발 스타일이나 프로젝트 요구 사항, 개발 기간에 따라 개발 방법과 테스트 기법들(Behavior-Driven Development, Acceptance Test-Driven Development, Feature-Driven Development..)이 달라지지만 JUnit과 Mocking을 통한 Test-Driven Developemnt를 지향하면서 익숙해지기 위해 사용해야 되는 적합한 상황과 방법에 대해 자세히 알아보고자 합니다

  - Common Development

    ![image info](/assets/img/CommonDevelopment.png)
    <img src="/assets/img/CommonDevelopment.png" alt="" width="0" height="0">

    - 소비자의 요구사항이 처음부터 명확하지 않을 수 있다(처음부터 완벽한 설계는 어렵다)
    - 자체 버그 검출 능력 저하 또는 소스코드의 품질이 저하될 수 있다
    - 자체 테스트 비용이 증가할 수 있다

  - Test-Driven Development

    ![image info](/assets/img/TestDrivenDevelopment.png)
    <img src="/assets/img/TestDrivenDevelopment.png" alt="" width="0" height="0">

    - 설계 단계에서 프로그래밍 목적을 반드시 미리 정의해야만 하고 무엇을 테스트해야 할지 정도는 미리 정의(테스트 케이스 작성)해야 한다
    - 테스트 코드를 작성하는 도중에 버그나 수정사항이 발생하면 테스트 케이스에 추가하고 설계를 개선한다 
    - 이후 테스트가 통과한 코드만을 코드 개발 단계에서 실제 코드로 작성한다

<br />

---
> ### Test-Driven Development(TDD)

  - Test First Development(TFD) + Refactoring : Test를 먼저 작성하고 계속적인 Refactoring을 진행하면서 코드를 더 나은 방향을 수정한다
  - Software 개발에서 테스트가 선행되어야 하는 원칙을 따른 개발 방법으로 코드 작성 전에 테스트 코드를 작성하고 해당 테스트가 통과할수 있도록 코드를 구현하는 방식
  - **Red-Green-Refactor** : TDD의 핵심 사이클(Red->Green->Refactor)

  ![image info](/assets/img/RedGreenRefactor.png)
  <img src="/assets/img/RedGreenRefactor.png" alt="" width="0" height="0">

  - #### Red-Green-Refactor

    - Red : 실패하는 테스트 작성
      - TDD의 첫번쨰 단계는 아직 Class, Method가 구현되지 않은 상태에서 테스트를 먼저 작성 -> 테스트는 반드시 실패한다
      - 지 않았기 때문에 
      - **어떤 기능이 필요하며 어떻게 동작해야 하는지 정의**

      ```java
      import org.junit.jupiter.api.Test;
      import static org.junit.jupiter.api.Assertions.assertEquals;

      public class CalculatorTest {
      
          @Test
          public void testAdd() {
              Calculator calc = new Calculator();
              assertEquals(5, calc.add(2, 3)); 
          }
      }
      ```

    - Green : 테스트를 통과하는 코드 작성
      - 미리 작성해놓은 실패하는 테스트에 대하여 통과하도록 코드를 작성
      - 코드의 구현은 **최소한**으로 하며 테스트를 성공적으로 통과시킨다
      - **테스트가 성공하도록 필요한 최소한의 기능만을 구현현**

      ```java
      public class Calculator {
      
          public int add(int a, int b) {
              return a + b; 
          }
      }
      ```
    - Refactor : 코드 리팩토링
      - 테스트가 통과한 코드에 대해 구조를 개선, 중복 제거, 가독성을 높이는 작업
      - **기존의 기능은 유지하면서 코드 품질을 개선**

      ```java
      public class Calculator {

          public int add(int... numbers) {
              int sum = 0;
              for (int num : numbers) {
                  sum += num;
              }
              return sum; 
          }
      }
      ```

      
---
> ### JUnit5 Annotation
 
  - Java에서 가장 널리 사용되는 단위 테스트 Framework
  - JUnit은 TDD의 중요한 도구로 테스트를 정의하고 실행할 수 있도록 해준다

  - #### Method Test Annotation

    - @Test & @ParameterizedTest
      - @Test : Test Method를 정의하는 Annotation(JUint5에서 실행할 수 있는 테스트로 인식)
      - @ParameterizedTest : Test Method를 여러러 Parameter와 함께 실행할 수 있도록 해준다(@ValueSource, @CsvSource, @MethodSource)

    ![image info](/assets/img/TddValue.png)
    <img src="/assets/img/TddValue.png" alt="" width="0" height="0">

    ```java
      import org.junit.jupiter.params.ParameterizedTest;
      import org.junit.jupiter.params.provider.ValueSource;
      import static org.junit.jupiter.api.Assertions.*;

      public class CalculatorTest {
        
        @ParameterizedTest
        @ValueSource(ints = {1, 2, 3, 4})
        void testAdditionWithMultipleValues(int number) {
          assertEquals(5, number + 1);  // 1+3, 2+3, 3+3, 4+3 모두 5와 비교
          }
      }
    ```


    - @DisplayName
      - Test Class, Method에 Custom된 이름을 부여하여 가독성을 높인다
    - @Nested
      - Test Class 내에 중첩된 Test Class를 정의(Test를 논리적으로 그룹화화)
    - @Tag
      - Test를 그룹화하하고 특정 그룹의 Test만 선택적으로 실행 및 특정 환경에서만 실행행할 수 있도록 해준다
    - @Disabled
      - Test를 비활성화(일시적으로 실행하고 싶지 않은 Test에 사용)
    - @Timeout
      - Test가 지정된 시간 내에 완료되지 않으면 Fail
      - 긴 실행 시간의 Test를 진행하는 경우 최대 실행 시간을 설정하여 제한 가능

      ```java
        import org.junit.jupiter.api.Test;
        import org.junit.jupiter.api.Timeout;

        import java.util.concurrent.TimeUnit;

        public class CalculatorTest {
        
            @Test
            @Timeout(value = 500, unit = TimeUnit.MILLISECONDS)
            void testTimeout() throws InterruptedException {
                Thread.sleep(100);  // 테스트는 500ms 이내에 종료되어야 한다.
            }
        }
      ```
    - @ExtendWith
      - JUnit5 확장 기능을 등록할 떄 사용
      - Test 실행 전에 추가적인 로직을 실행할 수 있도록 해준다다
        
      ```java
        import org.junit.jupiter.api.Test;
        import org.junit.jupiter.api.extension.ExtendWith;

        @ExtendWith(CustomTestExtension.class)
        public class CalculatorTest {
        
            @Test
            void testAddition() {
                System.out.println("Test is running...");
            }
        }
      ```
    - @RegisterExtension
      - Test Class에 Extension을 Field로 등록
      - Field로 등록한 Extension이 해당 Class의 모든 Test에서 사용되도록 한다다
        
      ```java
        import org.junit.jupiter.api.Test;
        import org.junit.jupiter.api.extension.RegisterExtension;

        public class CalculatorTest {
        
            @RegisterExtension
            static CustomTestExtension extension = new CustomTestExtension();

            @Test
            void testAddition() {
                System.out.println("Test with registered extension");
            }
        }
      ```

    - @TempDir
      - Temp Directory를 생성하여 Test에 활용하고 Test 종료 후 Directory를 자동으로 삭제한다
    
      ```java
        import org.junit.jupiter.api.Test;
        import org.junit.jupiter.api.io.TempDir;

        import java.nio.file.Path;

        public class FileTest {
        
            @Test
            void testTempDir(@TempDir Path tempDir) {
                System.out.println("Temporary Directory: " + tempDir);
            }
        }
      ```

  - #### LifeCycle Method Annotation
    - @BeforeEach
      - Test Method가 실행되기 전에 **초기화 작업**을 실행하는 Method에 사용
      
    - @BeforeAll
      - Test가 실행되기 전에 한 번만 실행(static Method에만 사용)

    - @AfterEach
      - Test Method가 실행된 후에 **정리 작업**을 할 떄 사용(리소스 해제 작업 등..)

    - @AfterAll
      - 모든 Test가 끝난 후 한 번만 실행(static Method에만 사용)



<br />

---
