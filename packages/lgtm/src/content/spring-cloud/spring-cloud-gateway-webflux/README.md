
### ToC

- [용어집(Glossary)](#용어집glossary)
- [Predicate](#predicate)
  - [Shortcut 방식](#shortcut-방식)
  - [Fully Expanded Configuration 방식](#fully-expanded-configuration-방식)
  - [Host Route Predicate 추가 설명](#host-route-predicate-추가-설명)
  - [RemoteAddr Route Predicate 추가 설명](#remoteaddr-route-predicate-추가-설명)
  - [Weight Route Predicate 추가 설명](#weight-route-predicate-추가-설명)
- [Filter](#filter)

## 용어집(Glossary)

- Route: ID, URI, Predicate의 조합, Filter의 조합으로 정의 된다.
- Predicate: [Java 8의 Predicate](https://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html)이다. 입력은 Spring Framework의 `ServerWebExchange`이다. 이 Predicate가 true를 반환하면 해당 Route가 선택된다. HTTP 요청의 헤더나 파라미터 등을 기반으로 라우팅을 결정할 수 있다.
- Filter: Route에 적용되는 필터로, 요청과 응답을 변형할 수 있다. 예를 들어, 인증, 로깅, 헤더 추가 등을 수행할 수 있다.


## Predicate

Shortcut, Fully Expanded Configuration 방식이 있다.

### Shortcut 방식

```yaml
spring.cloud.gateway:
  routes:
    - id: after_route
      uri: https://example.org
      predicates:
        - Cookie=mycookie,mycookievalue
```

### Fully Expanded Configuration 방식

```yaml
spring.cloud.gateway:
  routes:
    - id: after_route
      uri: https://example.org
      predicates:
        - name: Cookie
          args:
            name: mycookie
            value: mycookievalue
```

모든 값의 조건은 정규 표현식(Regular Expression, Java 정규 표현식)으로 처리된다. 예외로는 `After`, `Before`, `Between` Predicate는 ISO 8601 형식의 날짜와 시간 문자열을 사용한다.

| Predicate | 설명 |
|-----------|------|
| After | 요청이 특정 시간 이후에 있을 때 라우팅 |
| Before | 요청이 특정 시간 이전에 있을 때 라우팅 |
| Between | 요청이 특정 시간 범위 내에 있을 때 라우팅 |
| Cookie | 요청의 쿠키가 특정 이름과 패턴과 일치할 때 라우팅 |
| Header | 요청의 헤더가 특정 이름과 패턴과 일치할 때 라우팅 |
| Host | 요청의 호스트가 특정 패턴과 일치할 때 라우팅 |
| Method | 요청의 HTTP 메소드가 특정 패턴과 일치할 때 라우팅 |
| Path | 요청의 경로가 특정 패턴과 일치할 때 라우팅 |
| Query | 요청의 쿼리 파라미터가 특정 이름과 패턴과 일치할 때 라우팅 |
| RemoteAddr | 요청의 원격 주소가 특정 패턴과 일치할 때 라우팅 |
| Weight | 요청의 가중치가 특정 그룹에 속할 때 라우팅 |


### Host Route Predicate 추가 설명

```yaml
spring.cloud.gateway:
  routes:
    - id: log_sub_host
      uri: https://example.org
      predicates:
        - Host={sub}.somehost.org
      filters:
      - name: LogSubFilter
```

```java
@Slf4j
@Component
public class LogSubFilter implements GatewayFilter {

    public LogSubFilter() {
        super(Object.class);
    }

    @Override
    public GatewayFilter apply(Object config) {
        return (exchange, chain) -> {
            Map<String, String> uriVariables = exchange.getAttribute(ServerWebExchangeUtils.URI_TEMPLATE_VARIABLES_ATTRIBUTE);
            String sub = uriVariables != null ? uriVariables.get("sub") : null;

            if (sub != null) {
                log.info("[LogSubFilter] Host 서브도메인: {}", sub);
            } else {
                log.info("[LogSubFilter] sub 값 없음");
            }
            return chain.filter(exchange);
        };
    }
}
```

- sub 도메인 값에 따라 로깅 레벨 다르게 처리
- sub 도메인 값에 따라 A/B 테스트 제어


### RemoteAddr Route Predicate 추가 설명

**예시:**

| 시나리오 | 설명 |
|-----------|------|
| 내부망 접근 | 일부 API는 특정 내부 IP에서만 접근 가능함 |
| 관리자 페이지 접근 제어 | 관리자 페이지는 특정 IP 대역에서만 허용 |
| Geo IP 기반 차단 | 특정 국가나 지역에서의 접근을 차단 |

그러나, 클라이언트 IP는 프록시 서버나 로드 밸런서에 의해 변경될 수 있다. 
그렇기에 이는 사용하지 않는 것이 좋을 것 같다.


### Weight Route Predicate 추가 설명

group, weight(int) 두 개의 인수를 사용한다.


```yaml
spring.cloud.gateway:
  routes:
    - id: weight_high
      uri: https://weighthigh.org
      predicates:
        - Weight=group1, 8
    - id: weight_low
      uri: https://weightlow.org
      predicates:
        - Weight=group1, 2
```

이렇게 설정하면 `group1` 그룹에 속하는 라우트는 총 10개의 요청 중 8개는 `weight_high`로, 2개는 `weight_low`로 라우팅된다.

**사용 예시:**

- A/B 테스트: 두 개의 버전의 서비스를 운영하면서 트래픽을 비율에 따라 분배할 수 있다.
- Canary 배포: 새로운 버전을 소수의 사용자에게만 배포하여 안정성을 검증할 수 있다. (spring config server와 함께 사용)
- 트래픽 분산 실험

## Filter

들어오는 HTTP 요청과 나가는 HTTP 응답을 변형할 수 있다. 예를 들어, 인증, 로깅, 헤더 추가 등을 수행할 수 있다.

| Filter | 설명 |
|-----------|------|
| AddRequestHeader | 요청 헤더에 값을 추가 |
| AddRequestHeadersIfNotPresent | 요청 헤더가 없는 경우에만 값을 추가 |
| AddRequestParameter | 요청 파라미터에 값을 추가 |
| AddResponseHeader | 응답 헤더에 값을 추가 |
