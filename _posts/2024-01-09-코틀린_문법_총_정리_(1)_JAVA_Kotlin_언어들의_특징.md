---
title: 코틀린 문법 총 정리 (1) JAVA, Kotlin 언어들의 특징
date: 2024-01-09 23:41:00 +09:00
categories: [JAVA, Kotlin]
tags:
  [
    JAVA,
    Kotlin
  ]


---

## JAVA, Kotlin 언어들의 특징

### 차이점

#### 언어 및 생태계

#### **자바**

- 자바는 객체지향 프로그래밍 언어로 JVM(JAVA Virtual Machine)에서 실행.
- 엔터프라이즈 애플리케이션 및 안드로이드 앱 개발 등 다양한 분야에서 사용 가능

#### **코틀린**

- JetBrains에서 개발한 JVM 및 안드로이드에서 실행되는 공식적인 안드로이드 언어.
- 자바와의 100% 상호 운용성을 제공하면서 더 간결하고, 안정한 문법 제공



### 타입 시스템

#### 자바

- 정적 타입 언어이며, 컴파일 타임에 타입을 확인함.
- 명시적으로 타입 선언해야 함. 
  - 단, 자바 10부터 도입된 `로컬 변수 타입 추론` 이라는 기능을 존재. 
  - 이는 var 키워드를 사용하여 변수를 선언할 때 컴파일러가 우변의 표현식을 기반으로 변수의 타입을 추론할 수 있게 하는 기능.
  - 이를 통해 코드의 가독성을 향상시키고 반복 코드를 줄일 수 있다.

#### 코틀린

- 정적 타입 언어이며, 자바와 마찬가지로 컴파일 타임에 타입 확인

- 타입 추론을 통해 코드를 간결하게 작성할 수 있도록 지원

  - **`val` (Value)**: `val`로 선언된 변수는 읽기 전용 변수. 이는 자바의 `final` 변수와 유사

    - `val`로 선언된 변수는 반드시 선언 시점에 초기화되거나 생성자에서 초기화되어야 함.

    - `val a = 10`는 a가 10으로 초기화되며 이후에 a의 값을 변경할 수 없다.

    - ```kotlin
      val number = 10
      // number = 15  // 컴파일 오류 발생, val 변수는 재할당할 수 없습니다.
      ```

      

  - **`var` (Variable)**: `var`로 선언된 변수는 변경 가능한 변수.

    - 이 변수에 값을 할당한 후에도 다른 값으로 변경할 수 있다.

    - `var`는 일반적인 변수 선언에 사용되며, 프로그램 실행 도중에 그 값이 변경될 수 있다.

    - `var a = 10`은 a가 10으로 초기화되지만, 이후에 `a = 15`와 같이 다른 값으로 변경할 수 있다.

    - ```kotlin
      var number = 10
      number = 15  // 가능, var 변수는 재할당할 수 있습니다.
      ```

      

### 문법 및 특징

#### 자바

- 상당히 명시적이고, 문장을 많이 적어줘야 하는 언어

- Null 안전성을 보장하기 위한 추가 노력 필요

- 모든 변수가 Null이 될 수 없다.

- 자바는 명시적 형 변환 사용.

- 자바는 Null 병합 연산자`??`를 지원하지 않음

  - C# 같은 언어에서 `??` 연산자는 왼쪽 피연산자가 null이 아닌 경우 왼쪽 값을 반환하고, null인 경우 오른쪽 값을 반환함

    - ex) Sting op = null; String str = op ?? "Not Setting"; 해당 값의 경우 `op`는 null이므로 `Not Setting` 문자열이 반환된다. `op`가 null이 아니라면 `op`가 지닌 문자를 반환

  - Java 8 이상에서 `Optional` 클래스를 사용하여 비슷한 기능을 구현

  - `Optional`은 값이 null일 수 있는 객체를 감싸고, null 체크 없이 여러 작업을 수행할 수 있는 메서드를 제공합니다. 예를 들어, `Optional.ofNullable(value).orElse(defaultValue)`는 `value`가 null이 아닐 경우 `value`를 반환하고, null일 경우 `defaultValue`를 반환

  - ```java
    import java.util.Optional;
    
    public class NullMergeExample {
        public static void main(String[] args) {
            String value = null;
            String defaultValue = "Default Value";
    
            String result = Optional.ofNullable(value).orElse(defaultValue);
            System.out.println(result); // 출력: "Default Value"
        }
    }
    ```



#### 코틀린

- 표현력이 풍부하고, 간결한 문법 제공

- Null 안정성, 확장 함수, 데이터 클래스 등의 특징

- 코틀린에서는 Java와 달리, null 병합 연산자가 언어 차원에서 직접 지원

  - 코틀린의 null 병합 연산자는 `?:` 

  - 연산자는 왼쪽 표현식이 null이 아니면 그 값을 반환하고, null이면 오른쪽 표현식의 값을 반환함.

  - ```kotlin
    fun main() {
        val value: String? = null
          //: String?: 이 부분은 변수의 타입을 지정. String은 문자열을 나타내며, 뒤에 붙은 ?는 이 변수가 null 값을 가질 수 있음을 나타냄
        val defaultValue = "Default Value"
    
        // value가 null이 아니면 value를, null이면 defaultValue를 반환합니다.
        val result = value ?: defaultValue
    
        println(result) // 출력: "Default Value"
    }
    ```

- 코틀린에서는 기본적으로 모든 타입이 null이 될 수 없으며(null-safe), 변수가 null을 가질 수 있도록 하려면 타입 뒤에 ?를 붙여야 함

- `as` 키워드를 통한 `타입 강제 변환(type casting)` 수행 가능

  - 타입 강제 변환은 하나의 타입을  다른 타입으로 명시적으로 변환하는 것을 의미함

  - ```kotlin
    fun main() {
        val obj: Any = "This is a string"
    
        // obj를 String 타입으로 강제 변환
        val str: String = obj as String
    
        println(str) // 출력: "This is a string"
    }
    ```

  - 주의할 점은 변환하려는 타입과 호환되지 않는 경우 `ClassCastException`이 발생할 수 있다.

  - ```kotlin
    fun main() {
        val obj: Any = 123
    
        // obj를 String 타입으로 안전하게 변환 시도
        val str: String? = obj as? String
    
        println(str) // 출력: null
    }
    ```

    - `obj`는 `Int` 타입의 값을 가지고 있으므로, `String`으로의 변환은 실패하고 `str`은 null 값을 가지게 됨
    - 안전한 타입 변환을 위해서는 `as?` 연산자를 사용하는 것이 좋음. 



### 비동기 프로그래밍

#### 자바

- Java에서 비동기 프로그래밍은 여러 방식으로 구현될 수 있는데, 그 중 `스레드`와 `락`을 사용하는 방식은 가장 기본적인 형태 중 하나

  - `스레드`는 동시에 실행할 수 있는 코드의 흐름을 의미하고, `락`은 여러 스레드가 동시에 같은 데이터에 접근하는 것을 조절하는 메커니즘으로 이를 통해 멀티스레딩과 동시성을 관리할 수 있다.

  - ### 스레드(Thread)

    - **스레드의 기본**: Java에서 스레드는 `Thread` 클래스를 이용해 생성할 수 있다. 각 스레드는 독립적인 실행 흐름을 가지며, 주로 병렬 처리나 비동기 작업에 사용됨.
    - **스레드 생성과 실행**: `Thread` 클래스를 상속하거나 `Runnable` 인터페이스를 구현하여 스레드를 만들 수 있다. `start()` 메서드를 호출하여 스레드를 실행시키면, 스레드는 별도의 실행 흐름에서 코드를 수행하게 됨.

  - ### 락(Lock)

    - **락의 필요성**: 여러 스레드가 동시에 같은 데이터를 수정하려 할 때, 데이터 무결성을 보장하기 위해 락이 필요함.
    - **락의 사용**: Java에서는 `synchronized` 키워드를 사용하여 락을 구현할 수 있다. 이 키워드가 적용된 메서드나 블록은 한 번에 하나의 스레드만 접근할 수 있도록 제한함.

  - ```java
    public class ExampleThread extends Thread {
        public void run() {
            // 스레드가 수행할 작업
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            ExampleThread thread = new ExampleThread();
            thread.start(); // 스레드 실행
        }
    }
    
    // 락을 사용하는 예시
    public class Counter {
        private int count = 0;
    
        public synchronized void increment() {
            count++;
        }
    }
    ```

  - 1. `ExampleThread` 클래스는 스레드를 구현하며, `run()` 메서드 안에 스레드가 수행할 작업을 정의. 
    2. `Main` 클래스에서는 이 스레드를 시작.
    3. `Counter` 클래스에서 `increment` 메서드는 `synchronized` 키워드를 사용하여 락을 구현. 이 메서드는 동시에 하나의 스레드만 접근할 수 있어, 여러 스레드가 동시에 `count` 값을 변경하는 것을 방지함.

- Java 5에서 도입된 `java.util.concurrent` 패키지에는 `ExecutorService`, `Future`, `Callable`과 같은 동시성 프로그래밍을 위한 핵심 클래스들이 포함되어 Java의 동시성 프로그래밍 기능을 크게 향상

  - Java 8 이상에서는 CompletableFuture를 통해 비동기 프로그래밍을 더 쉽게 구현가능

    - ### ExecutorService

      - **기능**: `ExecutorService`는 스레드 풀 관리를 위한 고급 API를 제공, 여러 스레드를 효율적으로 관리
      - **사용 방법**: `Executors` 클래스의 정적 메서드를 사용하여 인스턴스를 생성가능, `submit()` 메서드를 사용하여 스레드 풀에 태스크를 제출할 수 있으며, 스레드 풀은 이 태스크들을 관리하고 실행

    - ### Future

      - **기능**: `Future`는 비동기 연산의 결과를 나타내는 객체. 연산이 완료될 때까지 기다린 후 결과를 가져오거나, 연산이 완료되었는지 확인할 수 있음.
      - **사용 방법**: `ExecutorService`를 통해 태스크를 제출하면 `Future` 객체가 반환. `get()` 메서드를 사용하여 태스크의 결과를 가져올 수 있다.

    - ### Callable

      - **기능**: `Callable`은 `Runnable`과 유사하지만, 값을 반환하고 예외를 던질 수 있는 태스크를 나타냄
      - **사용 방법**: `Callable` 인터페이스를 구현하는 클래스를 작성하고, 이를 `ExecutorService`에 제출하여 실행함. `Future`를 통해 결과를 받을 수 있다.

    - ### CompletableFuture

      - **기능**: Java 8에서 도입된 `CompletableFuture`는 비동기 프로그래밍을 위한 강력한 도구. `Future`의 기능을 확장하며, 비동기 태스크의 결과를 조합하고, 에러를 처리하며, 연쇄적인 작업 흐름을 구성할 수 있게 해줌.
      - **사용 방법**: `CompletableFuture`를 사용하여 비동기 연산을 시작하고, `thenApply`, `thenAccept`, `thenCombine` 등의 메서드를 연쇄적으로 사용하여 비동기 연산의 결과를 처리할 수 있음. 또한, `exceptionally` 메서드를 통해 예외 처리도 할 수 있다.

    - ```java
      import java.util.concurrent.*;
      
      public class AsyncExample {
          public static void main(String[] args) throws ExecutionException, InterruptedException {
              ExecutorService executor = Executors.newFixedThreadPool(10);
      
              // Callable 사용 예시
              Future<String> future = executor.submit(() -> {
                  Thread.sleep(1000);
                  return "결과";
              });
      
              System.out.println("기다리는 중...");
              String result = future.get();  // 블록되어 결과를 기다립니다.
              System.out.println(result);
      
              // CompletableFuture 사용 예시
              CompletableFuture.supplyAsync(() -> {
                  return "비동기 결과";
              }).thenAccept(System.out::println);
      
              executor.shutdown();
          }
      }
      ```

    - `ExecutorService`와 `Future`, 그리고 `CompletableFuture`를 사용한 간단한 비동기 프로그래밍을 보여줌

    - 1.  `Callable`을 이용해 비동기 태스크를 생성하고, 
      2.  `Future`를 통해 그 결과를 기다리며,
      3.  `CompletableFuture`를 사용하여 비동기 결과를 처리함.

#### 코틀린

- 코루틴을 통해 비동기 코드 작성

- 코루틴은 코틀린에서 경량 스레드와 같은 역할을 하며, 동시성 프로그래밍을 간결하고 효율적으로 수행할 수 있게 기능

- ```kotlin
  import kotlinx.coroutines.*
  
  fun main() = runBlocking { // 메인 스레드를 블록하지 않는 메인 코루틴
      val job = launch { // 새로운 코루틴 생성 및 시작
          delay(1000L) // 1초 동안 비동기적으로 대기
          println("World!") // 대기 후 출력
      }
      println("Hello") // 코루틴이 대기 중일 때 즉시 실행
      job.join() // 코루틴의 작업이 완료될 때까지 대기
  }
  ```

  1. `runBlocking` 블록은 메인 스레드에서 코루틴을 실행하며, 이 블록 내의 코드는 순차적으로 실행
  2. `launch`를 사용하여 새로운 코루틴을 시작. 이 코루틴은 `delay` 함수를 통해 1초 동안 비동기적으로 대기
  3. 메인 코루틴은 `launch` 코루틴이 대기하는 동안 "Hello"를 출력하고, `job.join()`을 호출하여 `launch` 코루틴의 작업이 완료될 때까지 대기
  4. `launch` 코루틴의 대기 시간이 끝나면 "World!"를 출력

  코루틴을 사용하면 비동기 코드를 더 간결하고 이해하기 쉽게 작성할 수 있으며, `콜백 지옥(callback hell)`을 피할 수 있다.



### 공통점

#### 자바와 코틀린

- **정적 타입 언어.** 컴파일러가 타입 오류를 잡아줄 수 있다.
  - 자바와 코틀린은 모두 정적 타입 언어. 정적 타입 언어는 변수나 함수의 타입을 컴파일 시점에 결정함.
  - 정적 타입 언어는 타입 오류를 컴파일 시점에 잡아낼 수 있다는 장점이 있음.
- **객체 지향 프로그래밍을 지원.** 클래스, 인터페이스, 상속, 캡슐화, 다형성 등을 지원함.
  - 자바와 코틀린은 모두 객체 지향 프로그래밍을 지원.
  - 객체 지향 프로그래밍은 `클래스, 인터페이스, 상속, 캡슐화, 다형성` 등의 개념을 사용하여 프로그램을 작성하는 방식.
  - 객체 지향 프로그래밍은 프로그램의 구조를 명확하게 하고, 유지 보수성을 높이는 데 도움이 됨.
- **JVM에서 실행됨.** 자바와 코틀린 모두 JVM에서 실행되는 바이트 코드로 컴파일되므로, 코드 혼용 가능.
  - 자바와 코틀린은 모두 JVM에서 실행됨
  - JVM은 자바 가상 머신으로, 자바 바이트 코드를 실행하는 역할을 함
  - JVM은 자바와 코틀린의 호환성을 높이는 데 중요한 역할을 함.



### 상호 운영성

#### 자바와 코틀린

- 코틀린은 `100%` 자바와 `상호 운영`이 가능하며, 자바 코드를 코틀린으로 변환하거나, 코틀린 코드를 자바로 변환 가능.
  -  코틀린이 `자바 가상 머신(JVM) `위에서 실행되도록 설계되었기 때문임.
  - 기존 자바 프로젝트에 코틀린 코드를 손쉽게 추가가능
  - 반대로 코틀린 코드에 자바에서도 사용가능
  - 코틀린 코드 내에서 자바 라이브러리를 사용할 수 있고, 자바 프로젝트에서도 코틀린으로 작성된 클래스나 함수를 사용할 수 있음.
- 그러나 코틀린이 제공하는 몇몇 특징들은 자바에서 직접적으로 사용할 수 없거나, 사용하기 위해서는 추가적인 고려가 필요할 수 있다.
  - 코틀린의 `널 안정성(null-safety) `특성이나 `확장 함수(extension functions) `같은 기능들은 자바에서는 자연스럽게 사용되지 않을 수 있음.
  - **최소 JDK 요구 사항**: 코틀린 1.3 이상은 JDK 6 이상에서 동작하도록 설계
  - **특정 JDK 버전에서의 추가 기능**: 코틀린은 일부 JDK 버전에서만 제공되는 특정 기능을 활용할 수 있음 예를 들어, `JDK 8의 람다 표현식과 스트림 API`는 코틀린 코드에서도 활용될 수 있음
  - **호환성 모드**: 코틀린은 JDK 버전에 따라 `호환성 모드`를 제공하여, 더 낮은 버전의 JDK에서도 코틀린 코드가 잘 동작하도록 지원

