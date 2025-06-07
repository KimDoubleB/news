---
layout: post
title: Spring Boot - API Versioning
description: Spring Boot에서 API 버전 관리하는 4가지 방법
summary: Spring Boot에서 API 버전 관리하는 4가지 방법 - URI, Request Parameter, Header, Content Negotiation
tags: spring-boot api versioning java
minute: 4
type: blog
---

API를 개발하다보면 API 버전에 대한 생각을 하게 된다.

버저닝(Versioning)을 하지 않으면 input/output이 변경되었을 때 이를 사용하고 있던 클라이언트에게 제대로 서비스할 수 없기 때문이다.

이러한 API Versioning을 하기 위해 지금까지 단순히 URI에 version을 명시하는 방향으로만 진행해왔는데, 더 다양한 방법이 있어 이를 정리해놓고자 한다.

<br/>

---

## URI Versioning / URI 이용하기

```java
// v1
@RestController
@RequestMapping("/api/uri/v1")
public class UriVersionControllerV1 {

    @GetMapping("/versions")
    public ResponseEntity<List<String>> getVersions() {
        return ResponseEntity.ok(List.of("v1.1", "v1.2", "v1.3"));
    }

}

// v2
@RestController
@RequestMapping("/api/uri/v2")
public class UriVersionControllerV2 {

    @GetMapping("/versions")
    public ResponseEntity<List<String>> getVersions() {
        return ResponseEntity.ok(List.of("v2.1", "v2.2", "v2.3"));
    }

}
```

앞서 이야기한 것처럼 URI에 특정 버전을 명시하는 방법이다.

구현하기 쉽고, URI를 통해 바로 버전을 확인할 수 있으므로 직관적이다.

하지만 URI 자체가 길어지고, URI를 깔끔하게 관리하기는 어렵다.

<br/>

---

## Request parameter Versioning

```java
@RestController
@RequestMapping("/api/parameter")
public class ParameterVersionController {

    @GetMapping(value = "/versions", params = "version=1")
    public ResponseEntity<List<String>> getVersionsV1() {
        return ResponseEntity.ok(List.of("v1.1", "v1.2", "v1.3"));
    }

    @GetMapping(value = "/versions", params = "version=2")
    public ResponseEntity<List<String>> getVersionsV2() {
        return ResponseEntity.ok(List.of("v2.1", "v2.2", "v2.3"));
    }

}
```

특정 값의 Request parameter를 통해서 버전을 구분할 수도 있다.

URI Versioning에 비해 URI를 깔끔하게 관리할 수 있지만, Client 측에서 적절한 version parameter를 꼭 고려해야만 한다.

<br/>

---

## Header Versioning

```java
@RestController
@RequestMapping("/api/header")
public class HeaderVersionController {

    @GetMapping(value = "/versions", headers = "X-API-Version=1")
    public ResponseEntity<List<String>> getVersionsV1() {
        return ResponseEntity.ok(List.of("v1.1", "v1.2", "v1.3"));
    }

    @GetMapping(value = "/versions", headers = "X-API-Version=2")
    public ResponseEntity<List<String>> getVersionsV2() {
        return ResponseEntity.ok(List.of("v2.1", "v2.2", "v2.3"));
    }

}
```

Custom header를 통해 버전을 구분할 수 있다. Request parameter 방식과 장단점은 유사하다.

만약 여기에 더불어 gateway 또는 reverse proxy를 사용하고 있다면 header에 대한 제한이 없는지 확인해봐야 할 필요가 있다.

<br/>

---

## Content negotiation Versioning

```java
@RestController
@RequestMapping("/api/header/accept")
public class AcceptHeaderVersionController {

    @GetMapping(value = "/versions", produces = "application/double.b.api.v1+json")
    public ResponseEntity<List<String>> getVersionsV1() {
        return ResponseEntity.ok(List.of("v1.1", "v1.2", "v1.3"));
    }

    @GetMapping(value = "/versions", produces = "application/double.b.api.v2+json")
    public ResponseEntity<List<String>> getVersionsV2() {
        return ResponseEntity.ok(List.of("v2.1", "v2.2", "v2.3"));
    }

}
```

[HTTP Content negotiation 방식](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Content_negotiation)을 이용한 Versioning 방법이다.

HTTP의 특성에 따라 custom header/request parameter 없이도 버전 관리를 할 수 있다.

이 부분도 Client 측에서 `Accept` header를 잘모르면 위 방식이 이해가지 않을 것 같아 Request 예시를 작성해보았다.

```bash
### accept header -> version 1
GET localhost:8080/api/header/accept/versions
Accept: application/double.b.api.v1+json

### accept header(content negotiation) -> version 1 - order
GET localhost:8080/api/header/accept/versions
Accept: application/double.b.api.v1+json, application/double.b.api.v2+json,

### accept header -> version 2
GET localhost:8080/api/header/accept/versions
Accept: application/double.b.api.v2+json

### accept header(content negotiation) -> version 2 - order
GET localhost:8080/api/header/accept/versions
Accept: application/double.b.api.v2+json, application/double.b.api.v1+json,

### accept header(content negotiation) -> version 1 - q factor
GET localhost:8080/api/header/accept/versions
Accept: application/double.b.api.v1+json;q=0.3, application/double.b.api.v2+json;q=0.1

### accept header(content negotiation) -> version 2 - q factor
GET localhost:8080/api/header/accept/versions
Accept: application/double.b.api.v1+json;q=0.7, application/double.b.api.v2+json;q=0.9
```

<br/>

## 결론

이렇듯 4가지 방법에 대해 알아보았다.

당연히 정답은 없다. 팀/조직에서 적절한 방법을 활용해 사용하면 된다.

대신 중요한 것은 **'일관되게 사용하고 있는가'** 인 것 같다.

