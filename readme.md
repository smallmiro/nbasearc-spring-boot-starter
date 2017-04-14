# nBase Arc spring-boot-starter library

[![Build Status](https://travis-ci.org/smallmiro/nbasearc-spring-boot-starter.svg?branch=master)](https://travis-ci.org/smallmiro/nbasearc-spring-boot-starter)
[![Coverage Status](https://coveralls.io/repos/github/smallmiro/nbasearc-spring-boot-starter/badge.svg?branch=master)](https://coveralls.io/github/smallmiro/nbasearc-spring-boot-starter?branch=master)
[![Apache 2](http://img.shields.io/badge/license-Apache%202-red.svg)](http://www.apache.org/licenses/LICENSE-2.0)
[![Maven Central](https://img.shields.io/badge/Maven%20Central-v1.0-blue.svg)](http://search.maven.org/#artifactdetails%7Ccom.github.smallmiro%7Cnbasearc-spring-boot-starter%7C1.0%7Cjar)

[![GitHub release](https://img.shields.io/github/release/qubyte/rubidium.svg)](https://github.com/nhnent/socket.io-client-unity3d/releases)
[![license](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/nhnent/socket.io-client-unity3d/blob/master/LICENSE) 


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
 * dependency 추가
```xml
<dependencies>
    <dependency>
        <groupId>com.github.smallmiro</groupId>
        <artifactId>nbasearc-spring-boot-starter</artifactId>
        <version>1.0.1</version>
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
# Related Projects
* nbase-arc : https://github.com/naver/nbase-arc

# Licensed
```

Copyright 2010-2015 the original author or authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

```
