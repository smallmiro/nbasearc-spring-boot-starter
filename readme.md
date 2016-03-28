# overview
* nbase-arc를 spring-boot에서 사용하기 편하도록 아래 내용을 추가했습니다.
 * spring-boot-autoconfigure
 * spring-boot-starter

# 변경 내역
* XML/ configuration 선언 없이 `spring.properties`의 선언 만으로 사용 가능
* 필요시 기존 방법대로 선언해서 사용 가능

## 기존 사용방법
* maven
```xml
<dependencies>
    <dependency>
        <groupId>com.navercorp</groupId>
        <artifactId>nbase-arc-java-client</artifactId>
        <version>1.4.6</version>
    </dependency>
</dependencies>
```

* XML 설정
```xml
    <bean id="gatewayConfig" class="com.navercorp.redis.cluster.gateway.GatewayConfig">
      <property name="zkAddress" value="zookeeper-address"/>
      <property name="clusterName" value="cluster-name"/>
    </bean>

    <bean id="redisClusterConnectionFactory" class="com.navercorp.redis.cluster.spring.RedisClusterConnectionFactory" destroy-method="destroy">
        <property name="config" ref="gatewayConfig"/>
    </bean>

    <bean id="redisTemplate" class="com.navercorp.redis.cluster.spring.StringRedisClusterTemplate">
        <property name="connectionFactory" ref="redisClusterConnectionFactory"/>
    </bean>
```

## 변경된 사용 방법
* maven
 * repository 추가
```xml
<repositories>
    <repository>
        <id>smallmiro-maven-releases</id>
        <url>https://github.com/smallmiro/smallmiro-maven-repo/raw/master/releases</url>
    </repository>
    <repository>
        <id>smallmiro-maven-snapshot</id>
        <url>https://github.com/smallmiro/smallmiro-maven-repo/raw/master/snapshots</url>
    </repository>
</repositories>
```
 * dependency 추가
```
<dependencies>
    <dependency>
        <groupId>com.smallmiro</groupId>
        <artifactId>nbasearc-spring-boot-starter</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

* spring-properties
```
nbase.arc.gateway.zkAddress=# user nbase-arc address #
nbase.arc.gateway.clusterName=# user cluster name #
nbase.arc.gateway.timeoutMillisec=3000
nbase.arc.gateway.healthCheckUsed=true
nbase.arc.gateway.healthCheckThreadSize=3
nbase.arc.gateway.healthCheckPeriodSeconds=10
nbase.arc.gateway.gatewaySelectorMethod=round-robin

nbase.arc.pool.initialSize=1
nbase.arc.pool.maxActive=4
nbase.arc.pool.maxIdle=2
nbase.arc.pool.minIdle=2
nbase.arc.pool.maxWait=2
```
* 예제 소스
```java
@Autowired
private StringRedisClusterTemplate redisTemplate;

@RequestMapping(value = "/get", method = RequestMethod.GET)
@ResponseBody
public Map<String, String> get() {
    String sample = redisTemplate.opsForValue().get(KEY);
    Map<String, String> re = new HashMap<String, String>();
    re.put("retrun", sample);
    return re;
}
```

# 예제 프로젝트
* nbasearc-spring-boot-sample
* IntegrationTest 작성 예제
```java 
@Test
public void getHello() throws Exception {
    String testValue = "SAMPLE";
    Map<String, Object> in = new HashMap<String, Object>();
    in.put("insert",testValue);

    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);
    HttpEntity<String> requestUpdate = new HttpEntity<String>(mapper.writeValueAsString(in), headers);

    ResponseEntity<Void> response1 = template.exchange(hostName + "/put", HttpMethod.PUT, requestUpdate, Void.class);
    assertThat(response1.getStatusCode(), equalTo(HttpStatus.OK));

    ResponseEntity<String> response2 = template.getForEntity(hostName + "/get", String.class);
    String body = response2.getBody();
    Map<String, Object> re = mapper.readValue(body, new TypeReference<Map<String, Object>>() {
    });
    assertThat((String) re.get("retrun"), equalTo(testValue));

}
```