# Spring Security Architecture

이 가이드는 Spring Security의 기본 설계와 기본 구성 요소에 대한 입문서입니다. 우리는 응용 프로그램 보안의 매우 기본적인 부분만 다룹니다. 그러나 이를 통해 Spring Security를 사용하는 개발자들이 겪는 혼란을 해소할 수 있습니다. 이를 위해 웹 응용 프로그램에서 보안이 어떻게 적용되는지 필터를 사용하고, 보다 일반적으로 메서드 주석을 사용하여 살펴봅니다. 이 가이드는 보안 응용 프로그램이 어떻게 작동하는지, 어떻게 커스터마이징 될 수 있는지 이해가 필요할 때, 또는 응용 프로그램 보안에 대해 어떻게 생각해야 하는지 배울 필요가 있을 때 사용하십시오.

이 가이드는 가장 기본적인 문제 이상을 해결하기 위한 설명서나 레시피로 의도되지 않았습니다(그 외의 문제에 대해서는 다른 출처를 참조하십시오), 그러나 초보자와 전문가 모두에게 유용할 수 있습니다. Spring Boot는 보안 응용 프로그램에 대한 일부 기본 동작을 제공하기 때문에 자주 언급되며, 전체 아키텍처와 어떻게 조화되는지 이해하는 데 유용할 수 있습니다.

| Note | 모든 원칙은 Spring Boot를 사용하지 않는 응용 프로그램에도 동등하게 적용됩니다. |
| ---- | ------------------------------------------------------------ |

------

## Authentication and Access Control (인증 및 접근 제어)

------

응용 프로그램 보안은 두 가지 더 혹은 덜 독립된 문제로 요약됩니다: 인증(당신은 누구인가요?)과 권한 부여(당신이 할 수 있는 것은 무엇인가요?). 때때로 사람들은 "authorization" 대신 "접근 제어"라고 말하기도 하는데, 이것은 혼란스러울 수 있지만, "authorization"이 다른 곳에서 중복되기 때문에 그렇게 생각하는 것이 도움이 될 수 있습니다. Spring Security는 인증과 권한 부여를 분리하도록 설계된 아키텍처를 가지고 있으며, 둘 다 전략과 확장 지점을 가지고 있습니다.

------

#### Authentication(인증)

------

인증의 주요 전략 인터페이스는 `AuthenticationManager`로, 하나의 메서드만 가지고 있습니다:

```java
public interface AuthenticationManager {
  Authentication authenticate(Authentication authentication)
    throws AuthenticationException;
}
```

`AuthenticationManager`는 `authenticate()` 메서드에서 3가지 중 하나를 수행할 수 있습니다:

- 입력이 유효한 주체(principal)를 나타내는 경우 검증할 수 있으면 `Authentication`을 반환합니다(일반적으로 `authenticated=true`로 함).
- 입력이 유효하지 않은 주체를 나타낸다고 생각되면 `AuthenticationException`을 던집니다.
- 결정할 수 없는 경우 `null`을 반환합니다.

`AuthenticationException`은 런타임 예외입니다. 일반적으로 응용 프로그램 스타일이나 목적에 따라 일반적인 방식으로 처리됩니다. 다시 말해, 사용자 코드가 이를 캐치하고 처리할 것으로 예상되지 않습니다. 예를 들어, 웹 UI는 인증에 실패했다는 페이지를 렌더링할 수 있으며, 백엔드 HTTP 서비스는 컨텍스트에 따라 `WWW-Authenticate` 헤더가 있거나 없는 401 응답을 보낼 수 있습니다.

`AuthenticationManager`의 가장 일반적으로 사용되는 구현은 `AuthenticationProvider` 인스턴스의 체인에 위임하는 `ProviderManager`입니다. `AuthenticationProvider`는 `AuthenticationManager`와 조금 유사하지만, 주어진 `Authentication` 타입을 지원하는지 호출자가 쿼리할 수 있도록 추가 메서드가 있습니다:

```java
public interface AuthenticationProvider {

	Authentication authenticate(Authentication authentication)
			throws AuthenticationException;

	boolean supports(Class<?> authentication);
}
```

`supports()` 메서드의 `Class<?>` 인수는 실제로 `Class<? extends Authentication>`입니다(`authenticate()` 메서드로 전달되는 것만 지원하는지 물어볼 때만 사용됩니다). `ProviderManager`는 `AuthenticationProviders`의 체인에 위임하여 동일한 응용 프로그램에서 여러 다른 인증 메커니즘을 지원할 수 있습니다. `ProviderManager`가 특정 `Authentication` 인스턴스 타입을 인식하지 못하면 건너뜁니다.

`ProviderManager`에는 선택적 부모가 있으며, 모든 제공자가 `null`을 반환하면 참조할 수 있습니다. 부모가 없으면 `null` `Authentication`은 `AuthenticationException`을 발생시킵니다.

때때로 응용 프로그램에는 보호된 리소스의 논리적 그룹(예: `/api/**`와 같은 경로 패턴과 일치하는 모든 웹 리소스)이 있으며, 각 그룹은 전용 `AuthenticationManager`를 가질 수 있습니다. 종종 각각은 `ProviderManager`이며 부모를 공유합니다. 그러면 부모는 모든 제공자에 대한 폴백으로 작동하는 "글로벌" 리소스의 종류가 됩니다.

![ProviderManagers with a common parent](https://github.com/spring-guides/top-spring-security-architecture/raw/main/images/authentication.png)

그림 1. `ProviderManager`를 사용한 `AuthenticationManager` 계층구조

------

#### Customizing Authentication Managers(인증 매니저 커스터마이징)

------

Spring Security는 응용 프로그램에서 일반적인 인증 매니저 기능을 빠르게 설정할 수 있도록 일부 구성 도우미를 제공합니다. 가장 일반적으로 사용되는 도우미는 인메모리, JDBC 또는 LDAP 사용자 세부 정보를 설정하거나 사용자 정의 `UserDetailsService`를 추가하기에 좋은 `AuthenticationManagerBuilder`입니다. 다음 예는 전역(부모) `AuthenticationManager`를 구성하는 응용 프로그램을 보여줍니다:

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {

   ... // 여기에 웹 관련 내용

  @Autowired
  public void initialize(AuthenticationManagerBuilder builder, DataSource dataSource) {
    builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
      .password("secret").roles("USER");
  }

}
```

이 예는 웹 응용 프로그램과 관련이 있지만, `AuthenticationManagerBuilder`의 사용법은 더 넓게 적용됩니다(웹 응용 프로그램 보안이 어떻게 구현되는지 자세한 내용은 [Web Security](https://spring.io/guides/topicals/spring-security-architecture/#web-security) 참조). `AuthenticationManagerBuilder`가 `@Bean`의 메서드로 `@Autowired`되는 것에 주목하세요. 이것이 전역(부모) `AuthenticationManager`를 구축하게 만듭니다. 반면, 다음 예를 고려해 보세요:

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {

  @Autowired
  DataSource dataSource;

   ... // 여기에 웹 관련 내용

  @Override
  public void configure(AuthenticationManagerBuilder builder) {
    builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
      .password("secret").roles("USER");
  }

}
```

구성기의 메서드에서 `@Override`를 사용했다면, `AuthenticationManagerBuilder`는 "로컬" `AuthenticationManager`를 구축하는 데만 사용되며, 이것은 전역 것의 하위 항목이 될 것입니다. Spring Boot 응용 프로그램에서는 전역 것을 다른 빈으로 `@Autowired`할 수 있지만, 직접 노출하지 않는 한 로컬 것은 그렇게 할 수 없습니다.

Spring Boot는 `AuthenticationManager` 유형의 자체 빈을 제공하지 않는 한 기본 전역 `AuthenticationManager`(단 하나의 사용자만 있음)를 제공합니다. 기본값은 활성화된 사용자 정의 전역 `AuthenticationManager`가 필요하지 않는 한 걱정할 필요가 없을 정도로 충분히 안전합니다. `AuthenticationManager`를 구축하는 어떤 구성도 수행하면, 보호하는 리소스에 지역적으로 수행하고 전역 기본값에 대해 걱정하지 않을 수 있습니다.

------

#### Autorization or Access Control(권한 부여 또는 접근 제어)

------

인증이 성공하면 권한 부여로 이동할 수 있으며, 여기의 핵심 전략은 `AccessDecisionManager`입니다. 프레임워크에서 제공하는 세 가지 구현이 있으며, 모두 `AccessDecisionVoter` 인스턴스의 체인에 위임합니다. 이는 `ProviderManager`가 `AuthenticationProviders`에 위임하는 것과 유사합니다.

`AccessDecisionVoter`는 `Authentication` (주체를 나타냄)과 보안 `Object` (ConfigAttributes로 꾸며진)를 고려합니다:

```java
boolean supports(ConfigAttribute attribute);

boolean supports(Class<?> clazz);

int vote(Authentication authentication, S object,
        Collection<ConfigAttribute> attributes);
```

`AccessDecisionManager`와 `AccessDecisionVoter`의 서명에서 `Object`는 완전히 일반적입니다. 이것은 사용자가 접근하려는 것을 나타내며 (웹 리소스나 Java 클래스의 메서드가 가장 일반적인 경우입니다). `ConfigAttributes`도 상당히 일반적이며, 리소스에 대한 권한 수준을 결정하는 메타데이터로 보안 `Object`를 꾸미는 것을 나타냅니다. `ConfigAttribute`는 인터페이스로, 하나의 메서드만 있으며 (이것은 매우 일반적이며 `String`을 반환합니다), 따라서 이러한 문자열은 리소스 소유자의 의도를 어떤 방식으로든 인코딩하며, 누가 그것에 접근할 수 있는지에 대한 규칙을 표현합니다. 전형적인 `ConfigAttribute`는 사용자 역할의 이름 (예: `ROLE_ADMIN` 또는 `ROLE_AUDIT`)이며, 특별한 형식을 가지거나 평가해야 하는 표현식을 나타낼 수 있습니다.

대부분의 사람들은 기본 `AccessDecisionManager`, 즉 `AffirmativeBased`를 사용합니다 (어떤 투표자든 긍정적으로 반환하면 접근이 허용됩니다). 모든 커스터마이징은 투표자에서 일어나며, 새로운 것을 추가하거나 기존 것의 작동 방식을 수정함으로써 이루어집니다.

Spring Expression Language (SpEL) 표현식인 `ConfigAttributes`를 사용하는 것이 매우 일반적입니다. 예를 들어, `isFullyAuthenticated() && hasRole('user')`와 같습니다. 이것은 표현식을 처리하고 그들에 대한 컨텍스트를 만들 수 있는 `AccessDecisionVoter`에 의해 지원됩니다. 처리할 수 있는 표현식의 범위를 확장하려면 `SecurityExpressionRoot`와 때로는 `SecurityExpressionHandler`의 커스텀 구현이 필요합니다.





------

## Web Security(웹 보안)

------

웹 계층(사용자 인터페이스와 HTTP 백엔드용)에서의 Spring Security는 Servlet `Filters`를 기반으로 하므로, 먼저 일반적으로 `Filters`의 역할을 살펴보는 것이 도움이 됩니다. 다음 그림은 단일 HTTP 요청에 대한 처리기의 전형적인 계층을 보여줍니다.

![Filter chain delegating to a Servlet](https://github.com/spring-guides/top-spring-security-architecture/raw/main/images/filters.png)

고객이 애플리케이션에 요청을 보내고, 컨테이너는 요청 URI의 경로를 기반으로 어떤 필터와 어떤 서블릿이 적용되는지 결정합니다. 대부분의 경우 하나의 서블릿만 단일 요청을 처리할 수 있지만, 필터는 체인을 형성하므로 순서가 있습니다. 실제로 필터는 요청을 직접 처리하려는 경우 체인의 나머지를 거부할 수 있습니다. 필터는 또한 하류 필터와 서블릿에서 사용되는 요청 또는 응답을 수정할 수 있습니다. 필터 체인의 순서는 매우 중요하며, Spring Boot는 두 가지 메커니즘을 통해 이를 관리합니다: `Filter` 유형의 `@Beans`는 `@Order`를 가질 수 있거나 `Ordered`를 구현할 수 있으며, 그 자체로 순서를 가진 `FilterRegistrationBean`의 일부가 될 수 있습니다. 일부 판매용 필터는 상대적으로 서로 어떤 순서를 선호하는지 신호를 보내는 데 도움이 되는 자체 상수를 정의합니다 (예를 들어, Spring Session의 `SessionRepositoryFilter`는 `Integer.MIN_VALUE + 50`의 `DEFAULT_ORDER`를 가지며, 이것은 체인에서 먼저 나오길 원한다는 것을 알려주지만, 그 앞에 다른 필터가 올 수 있다는 것을 배제하지 않습니다).

Spring Security는 체인의 단일 `Filter`로 설치되며, 구체적인 유형은 곧 다룰 이유로 `FilterChainProxy`입니다. Spring Boot 애플리케이션에서 보안 필터는 `ApplicationContext`의 `@Bean`이며, 모든 요청에 적용되도록 기본적으로 설치됩니다. 이는 `SecurityProperties.DEFAULT_FILTER_ORDER`에 의해 정의된 위치에 설치되며, 이는 다시 `FilterRegistrationBean.REQUEST_WRAPPER_FILTER_MAX_ORDER` (요청을 래핑하면서 그 행동을 수정할 필터가 가져야 할 Spring Boot 애플리케이션의 최대 순서)에 의해 고정됩니다. 그러나 그것보다 더 많은 것이 있습니다: 컨테이너의 관점에서 볼 때, Spring Security는 단일 필터이지만 그 안에는 특별한 역할을 하는 추가 필터가 있습니다. 다음 이미지가 이 관계를 보여줍니다:

![Spring Security Filter](https://github.com/spring-guides/top-spring-security-architecture/raw/main/images/security-filters.png)

그림 2. Spring Security는 단일 물리적 `Filter`이지만 내부 필터의 체인에 처리를 위임합니다.

실제로 보안 필터에는 더 깊은 계층의 간접 참조가 하나 더 있습니다: 일반적으로 컨테이너에 `DelegatingFilterProxy`로 설치되며, 이것은 반드시 Spring의 `@Bean`일 필요가 없습니다. 프록시는 항상 `@Bean`이며, 일반적으로 `springSecurityFilterChain`이라는 고정된 이름을 가진 `FilterChainProxy`에 위임합니다. 모든 보안 로직이 내부적으로 필터(또는 체인)의 체인으로 배열된 `FilterChainProxy`입니다. 모든 필터는 동일한 API를 가지고 있으며 (모두 서블릿 사양의 `Filter` 인터페이스를 구현합니다), 체인의 나머지를 거부할 기회가 있습니다.

동일한 최상위 `FilterChainProxy`에서 Spring Security에 의해 관리되는 여러 필터 체인이 있을 수 있으며, 모두 컨테이너에게 알려져 있지 않습니다. Spring Security 필터에는 필터 체인 목록이 있으며, 일치하는 첫 번째 체인에 요청을 전송합니다. 다음 그림은 요청 경로를 기반으로 일치시키는 디스패치를 보여줍니다 (`/foo/**`는 `/**` 이전에 일치합니다). 이것은 매우 일반적이지만 요청과 일치시키는 유일한 방법은 아닙니다. 이 디스패치 프로세스의 가장 중요한 특징은 한 번에 하나의 체인만이 요청을 처리한다는 것입니다.

![Security Filter Dispatch](https://github.com/spring-guides/top-spring-security-architecture/raw/main/images/security-filters-dispatch.png)

그림 3. Spring Security의 `FilterChainProxy`는 일치하는 첫 번째 체인에 요청을 전달합니다.

맞춤형 보안 구성이 없는 기본 Spring Boot 애플리케이션에는 여러 개(일반적으로 n=6이라고 부릅니다)의 필터 체인이 있습니다. 처음 (n-1) 체인은 `/css/**`와 `/images/**`와 같은 정적 리소스 패턴과 오류 뷰 `/error`를 무시하기 위해 존재합니다. (경로는 `SecurityProperties` 구성 빈에서 `security.ignored`로 사용자가 제어할 수 있습니다.) 마지막 체인은 모든 경로(`/**`)와 일치하며 인증, 권한 부여, 예외 처리, 세션 처리, 헤더 작성 등의 로직을 더 활발히 포함합니다. 기본적으로 이 체인에는 총 11개의 필터가 있지만, 일반적으로 사용자가 어떤 필터를 언제 사용하는지에 대해 걱정할 필요가 없습니다.

| 참고 | Spring Security 내부의 모든 필터가 컨테이너에 알려져 있지 않다는 사실은 중요하며, 특히 Spring Boot 애플리케이션에서 기본적으로 `Filter` 유형의 모든 `@Beans`가 자동으로 컨테이너에 등록되기 때문에 중요합니다. 따라서 보안 체인에 사용자 정의 필터를 추가하려면, `@Bean`으로 만들지 않거나, 컨테이너 등록을 명시적으로 비활성화하는 `FilterRegistrationBean`에 래핑해야 합니다. |
| ---- | ------------------------------------------------------------ |



------

#### Creating and Customizing Filter Chains(필터 체인 생성 및 사용자 정의)

------

Spring Boot 애플리케이션의 기본 후퇴 필터 체인(즉, `/**` 요청 매처가 있는 체인)은 `SecurityProperties.BASIC_AUTH_ORDER`의 미리 정의된 순서를 가집니다. `security.basic.enabled=false`를 설정하여 완전히 끌 수 있거나, 후퇴로 사용하고 더 낮은 순서로 다른 규칙을 정의할 수 있습니다. 후자를 하려면, `WebSecurityConfigurerAdapter`(또는 `WebSecurityConfigurer`) 유형의 `@Bean`을 추가하고 다음과 같이 클래스에 `@Order`를 추가합니다:

```java
@Configuration
@Order(SecurityProperties.BASIC_AUTH_ORDER - 10)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/match1/**")
     ...;
  }
}
```

이 빈은 Spring Security에 새 필터 체인을 추가하고 후퇴보다 앞에 순서를 지정하도록 합니다.

많은 애플리케이션은 한 리소스 집합과 다른 리소스 집합에 대해 완전히 다른 접근 규칙을 가집니다. 예를 들어, UI와 백엔드 API를 호스팅하는 애플리케이션은 UI 부분에 대한 로그인 페이지로의 리다이렉트와 함께 쿠키 기반 인증을 지원하고, API 부분에 대한 401 응답과 함께 토큰 기반 인증을 지원할 수 있습니다. 각 리소스 집합은 고유한 순서와 자체 요청 매처를 가진 자체 `WebSecurityConfigurerAdapter`를 가집니다. 일치하는 규칙이 겹치면, 가장 먼저 순서가 지정된 필터 체인이 선택됩니다.

------

#### Request Matching for Dispatch and Authorization(디스패치 및 권한 부여를 위한 요청 매칭)

------

보안 필터 체인(또는 동등하게 `WebSecurityConfigurerAdapter`)은 HTTP 요청에 적용할지 여부를 결정하기 위해 요청 매처를 사용합니다. 특정 필터 체인을 적용하기로 결정하면, 다른 체인은 적용되지 않습니다. 그러나 필터 체인 내에서 `HttpSecurity` 구성기에 추가 매처를 설정함으로써 권한 부여에 대한 더 세밀한 제어를 할 수 있습니다. 예를 들면 다음과 같습니다:

```java
@Configuration
@Order(SecurityProperties.BASIC_AUTH_ORDER - 10)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/match1/**")
      .authorizeRequests()
        .antMatchers("/match1/user").hasRole("USER")
        .antMatchers("/match1/spam").hasRole("SPAM")
        .anyRequest().isAuthenticated();
  }
}
```

Spring Security를 구성할 때 가장 쉽게 실수하는 것 중 하나는 이러한 매처들이 다른 프로세스에 적용된다는 것을 잊는 것입니다. 하나는 전체 필터 체인에 대한 요청 매처이고, 다른 하나는 적용할 접근 규칙을 선택하기만 하는 것입니다.

------

#### Combining Application Security Rules with Actuator Rules(애플리케이션 보안 규칙과 Actuator 규칙 결합하기)

------

Spring Boot Actuator를 관리 엔드포인트로 사용하는 경우, 그것들을 보안하려고 할 것이며 기본적으로 그렇게 됩니다. 사실, Actuator를 보안 애플리케이션에 추가하자마자, actuator 엔드포인트에만 적용되는 추가 필터 체인이 생깁니다. 이것은 actuator 엔드포인트만 일치시키는 요청 매처로 정의되며, `ManagementServerProperties.BASIC_AUTH_ORDER` 순서를 가지고 있어, 기본 `SecurityProperties` 후퇴 필터보다 5개 적으므로 후퇴 전에 참조됩니다.

애플리케이션 보안 규칙을 actuator 엔드포인트에 적용하려면, actuator보다 먼저 순서가 지정된 필터 체인을 추가하고 모든 actuator 엔드포인트를 포함하는 요청 매처를 갖도록 할 수 있습니다. actuator 엔드포인트에 대한 기본 보안 설정을 선호한다면, 가장 쉬운 방법은 actuator보다 늦게, 하지만 후퇴보다 먼저 필터를 추가하는 것입니다 (예: `ManagementServerProperties.BASIC_AUTH_ORDER + 1`). 예를 들면 다음과 같습니다:

```

@Configuration
@Order(ManagementServerProperties.BASIC_AUTH_ORDER + 1)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/foo/**")
     ...;
  }
}
```



| 참고 | 웹 계층의 Spring Security는 현재 Servlet API에 연결되어 있으므로, 실제로는 임베디드나 그 외의 서블릿 컨테이너에서 애플리케이션을 실행할 때만 적용됩니다. 그러나, Spring MVC 또는 Spring 웹 스택의 나머지 부분과 연결되어 있지 않으므로, 예를 들어 JAX-RS를 사용하는 모든 서블릿 애플리케이션에서 사용할 수 있습니다. |
| ---- | ------------------------------------------------------------ |

------

## Method Security(메서드 보안)

------

웹 애플리케이션을 보호하기 위한 지원 외에도 Spring Security는 Java 메서드 실행에 접근 규칙을 적용하기 위한 지원을 제공합니다. Spring Security에게 이것은 단순히 "보호된 리소스"의 다른 유형일 뿐입니다. 사용자에게는 동일한 형식의 `ConfigAttribute` 문자열(예: 역할 또는 표현식)을 사용하여 접근 규칙이 선언되지만 코드 내에서 다른 위치에 선언된다는 것을 의미합니다. 첫 번째 단계는 메서드 보안을 활성화하는 것입니다. 예를 들어, 우리 애플리케이션의 최상위 설정에서 다음과 같이 할 수 있습니다:

```

@SpringBootApplication
@EnableGlobalMethodSecurity(securedEnabled = true)
public class SampleSecureApplication {
}
```

그런 다음 메서드 리소스를 직접 꾸밀 수 있습니다:

```java
@Service
public class MyService {

  @Secured("ROLE_USER")
  public String secure() {
    return "Hello Security";
  }

}
```

이 예제는 보안 메서드가 있는 서비스입니다. Spring이 이 유형의 `@Bean`을 생성하면 프록시되며, 호출자는 메서드가 실제로 실행되기 전에 보안 인터셉터를 통과해야 합니다. 접근이 거부되면, 호출자는 실제 메서드 결과 대신 `AccessDeniedException`을 받게 됩니다.

메서드에 보안 제약 조건을 적용하기 위해 사용할 수 있는 다른 주석도 있으며, 특히 메서드 매개변수와 반환 값에 대한 참조를 포함하는 표현식을 작성할 수 있는 `@PreAuthorize`와 `@PostAuthorize`가 있습니다.

| 팁   | 웹 보안과 메서드 보안을 결합하는 것이 흔하지 않지 않습니다. 필터 체인은 사용자 경험 기능, 예를 들어 인증 및 로그인 페이지로의 리다이렉트 등을 제공하며, 메서드 보안은 더 세분화된 수준에서 보호를 제공합니다. |
| ---- | ------------------------------------------------------------ |

------

## Working with Threads(스레드 작업하기)

------

Spring Security는 근본적으로 스레드 바인딩되어 있으며, 현재 인증된 주체(principal)를 다양한 하위 소비자에게 사용할 수 있게 해야 하기 때문입니다. 기본 구성 요소는 `SecurityContext`로, `Authentication`을 포함할 수 있으며(사용자가 로그인하면 명시적으로 `authenticated`인 `Authentication`입니다). `SecurityContextHolder`의 정적 편의 메서드를 통해 `SecurityContext`에 항상 액세스하고 조작할 수 있으며, 이는 차례로 `ThreadLocal`을 조작합니다. 다음 예제는 이러한 배열을 보여줍니다:

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
assert(authentication.isAuthenticated);
```

이것은 사용자 애플리케이션 코드에서 하는 일반적인 작업이 **아니지만**, 예를 들어 사용자 정의 인증 필터를 작성해야 하는 경우 유용할 수 있습니다(그럼에도 불구하고, `SecurityContextHolder`를 사용할 필요 없이 사용할 수 있는 Spring Security의 기본 클래스가 있습니다).

웹 엔드포인트에서 현재 인증된 사용자에게 액세스해야 하는 경우, 다음과 같이 `@RequestMapping`에서 메서드 매개변수를 사용할 수 있습니다:

```java
@RequestMapping("/foo")
public String foo(@AuthenticationPrincipal User user) {
  ... // do stuff with user
}
```

이 주석은 현재 `Authentication`을 `SecurityContext`에서 가져와 메서드 매개변수를 생성하기 위해 `getPrincipal()` 메서드를 호출합니다. `Authentication`의 `Principal` 유형은 인증을 검증하는 데 사용된 `AuthenticationManager`에 따라 다르므로, 사용자 데이터에 대한 타입-세이프 참조를 얻기 위한 유용한 작은 방법이 될 수 있습니다.

Spring Security가 사용 중인 경우, `HttpServletRequest`의 `Principal`은 `Authentication` 유형이므로 직접 사용할 수도 있습니다:

```java
@RequestMapping("/foo")
public String foo(Principal principal) {
  Authentication authentication = (Authentication) principal;
  User = (User) authentication.getPrincipal();
  ... // do stuff with user
}
```

이것은 Spring Security가 사용되지 않을 때 작동하는 코드를 작성해야 하는 경우 때때로 유용할 수 있습니다(`Authentication` 클래스를 로드하는 데 더 방어적이어야 할 것입니다).

------

#### Processing Secure Methods Asynchronously(비동기로 보안 메서드 처리하기)

------

`SecurityContext`가 스레드 바인딩되어 있으므로 보안 메서드를 호출하는 배경 처리(예: `@Async`와 함께)를 하려면 컨텍스트가 전파되도록 해야 합니다. 이것은 배경에서 실행되는 작업(`Runnable`, `Callable` 등)과 함께 `SecurityContext`를 래핑하는 것으로 귀결됩니다. Spring Security는 이를 더 쉽게 만들기 위해 `Runnable` 및 `Callable`에 대한 래퍼와 같은 도우미를 제공합니다. `SecurityContext`를 `@Async` 메서드로 전파하려면, `AsyncConfigurer`를 제공하고 `Executor`가 올바른 유형인지 확인해야 합니다:

```java
@Configuration
public class ApplicationConfiguration extends AsyncConfigurerSupport {

  @Override
  public Executor getAsyncExecutor() {
    return new DelegatingSecurityContextExecutorService(Executors.newFixedThreadPool(5));
  }

}
```

이 설정은 `SecurityContext`를 비동기로 실행되는 작업에 전파하는 데 사용되므로, 이로 인해 보안 메서드가 비동기 방식으로 처리될 때에도 동일한 보안 컨텍스트가 유지됩니다.