---
theme: default
highlighter: shiki
lineNumbers: false
info: |
  ## Quick summary of Spring Boot's configuration possibilities
drawings:
  persist: false

# cover slide
class: "text-center"
---

# Configuring Spring Boot

Property sources, profiles and conditional beans

<img class="w-1/5 rounded mx-auto mt-5 shadow-lg" src="/SEA-041.jpg" alt="Background by Ivan Seal">

<div class="abs-br m-6 flex gap-2">
<!-- TODO update github link -->
  <a href="https://docs.spring.io/spring-boot/docs/2.1.9.RELEASE/reference/html/boot-features-external-config.html" target="_blank" alt="Spring Docs"
    class="text-xl icon-btn opacity-50 !border-none">
    <bx-bxl-spring-boot />
  </a>
  <a href="https://github.com/bencemol/slides-configuring-spring" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# Property sources

Priority from highest to lowest

1. command line arguments `java -jar app.jar --someProperty="someValue"`

<v-clicks>

2. Java System properties `System.getProperties()`
3. OS environment variables
4. Profile specific application properties `application-{profile}.[properties|yml]`
   1. outside of jar
   2. packaged in jar
5. Application properties `application.[properties|yml]`
   1. outside of jar
   2. packaged in jar

</v-clicks>

---

# YAML 👌

```yaml {all|3|4|5|6|7-10|11-14}
my:
  properties:
    number: 69
    string: lorem
    escaped: ':{[.&*#?|-<>+!%@\]}:'
    boolean: true
    list:
      - And your mom would stick a fork right into daddy's shoulder
      - And dad would throw the garbage all across the floor
      - As we would lay and learn what each other's bodies were for
    map:
      one: eins
      two: zwei
      three: drei
```

---

# Placeholders

Refer back to previously defined values (environment variables too)

```yaml {all|2-4|5-9|10-12|all}
# application.yml
app:
  name: Naomi
  description: Hey it's me, ${app.name}
spring:
  datasource:
    url: ${NAOMI_DATASOURCE_URL}
    username: ${NAOMI_DATASOURCE_USERNAME}
    password: ${NAOMI_DATASOURCE_PASSWORD}
jwt:
  expiration-time: ${JWT_EXPIRATION:86400000} # default 1 day
```

<v-click>

```bash
#!/bin/bash
docker run my-image -e "JWT_EXPIRATION=28800000"
```

</v-click>

---

# 🍵 in Java

Injecting properties directly

```java
@Service
public class AuthenticationService {

  @Value('${jwt.expiration-time:10000}')
  private Long jwtExpiration;
}
```

```java {none|all}
@Service
@RequiredArgsConstructor
public class AuthenticationService {

  private final Long jwtExpiration;

  public AuthenticationService(
    @Value('${jwt.expiration-time:10000}') Long jwtExpiration
  ) {
    this.jwtExpiration = jwtExpiration;
  }
}
```

---

# 🍵 in Java

Injecting through configuration classes

```java {all|1|2|all}
@ConfigurationProperties(prefix = "jwt")
@ConstructorBinding
@Getter
public class JwtProperties {

  private final Long expirationTime;
  private final String secret;
}
```

```java {none|all}
@Service
@RequiredArgsConstructor
public class AuthenticationService {

  private final JwtProperties jwtProperties;
}
```

---

# Property validation

JSR-303 `javax.validation`

```java
@ConfigurationProperties("jwt")
@Validated
public class JwtProperties {

  @Positive
  private Long expirationTime;
}
```

---

# Lists & Maps

List

```yaml
flyway:
  locations:
    - classpath:db/migration/{vendor}
    - classpath:db/feature/complaint-management/{vendor}
```

```java {none|4|all}
@ConfigurationProperties("flyway")
public class FlywayConfiguration {

  private List<String> locations;
}
```

---

# Lists & Maps

Map

```yaml
oauth2:
  client:
    registration:
      example-provider:
        provider: example
        clientId: 0oa2vomf0btxsaWo34x4
        clientSecret: PwX>NJdQ;C2V'M)\akC2syhw6[5#{sq
```

```java {none|6-10|4|all}
@ConfigurationProperties("oauth2.client")
public class RelyingPartyProperties {

  private Map<String, Registration> registration;

  public static class Registration {
    String provider;
    String clientId;
    String clientSecret;
  }
}
```

---

# Profiles

- `java -jar app.jar --spring.profiles.active=dev,feature1`
- profile specific properties `application-dev.yml`
- conditional bean initialization

<v-clicks>

```yaml
# application-dev.yml
springdoc:
  swagger-ui:
    enabled: true
  api-docs:
    enabled: true
```

```java
@Profile("dev")
@Configuration
public class MockSecurityConfig extends WebSecurityConfigurerAdapter {
  // ...
}
```

</v-clicks>

---

# Conditional beans

##### Conditional on other bean

```java
@ConditionalOnBean(Saml2RelyingPartyRegistrationConfiguration.class)
@Configuration
public class Saml2SecurityConfigurer {
  // ...
}
```

<v-clicks>
<div>

##### Conditional on property

```java
@ConditionalOnProperty(value = "flyway.repair-on-migrate", havingValue = "true")
@Bean
public FlywayMigrationStrategy repairStrategy() {
  // ...
}
```

</div>

<div>

##### Conditional

<div class="grid grid-cols-2 gap-2">

```java
public class MyCondition extends SpringBootCondition {
  // ...
}
```

```java
@Conditional(MyCondition.class)
@Bean
public MyBean myBean() {
  // ...
}
```

</div>
</div>
</v-clicks>
