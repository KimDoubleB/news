---
layout: post
title: "Trace Sampling / 트레이스 샘플링"
summary: "트레이스 샘플링의 개념과 Head/Tail Sampling의 차이점, 그리고 실제 적용 시 고려사항들을 정리한 글"
tags: trace sampling observability opentelemetry
minute: 5
type: blog
---

Trace를 학습하다보면 "Sampling", "샘플링"에 대해 필수적으로 알아야 한다.

근데 주변에서 가끔 헷갈려하시는 분들이 있어 글로 좀 정리해보자 한다.

본 글의 내용은 아래 Opentelemetry 공식문서 글을 참조했다.

**[OpenTelemetry - Sampling](https://opentelemetry.io/docs/concepts/sampling/)**

<br/>

---

## Sampling? 샘플링?

<img alt="Image" src="https://private-user-images.githubusercontent.com/37873745/465695746-cd730481-ed95-418e-990e-31cc2cf2ccd6.jpeg?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTIzODQ4NjksIm5iZiI6MTc1MjM4NDU2OSwicGF0aCI6Ii8zNzg3Mzc0NS80NjU2OTU3NDYtY2Q3MzA0ODEtZWQ5NS00MThlLTk5MGUtMzFjYzJjZjJjY2Q2LmpwZWc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwNzEzJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDcxM1QwNTI5MjlaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1mMTBiZGYxMTE3MzY5NjIzNDNmYzY3YzA1ZWY3ZDI4YzYwMGNiNjc3MzM0MTc5ZTg5NTdmZWUxNzE1OGUyYTIxJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.wC7Bo100TteQdOJ2eAs49BS5Fc99Z_VAeBw7w8CGDWQ" />


**Sampling / 샘플링**

- 모든 가시성 지표를 저장하지 않고, <mark>대표적인 데이터만(대표성을 띄는 데이터만) 저장하는 것</mark>
- '대표성'이라는 건 작은 그룹이 더 큰 그룹을 정확하게 대표할 수 있다는 원칙

<br/>

"Sampling 됬어?" 같은 개념을 헷갈리는 사람들이 있다. 헷갈리지 말자.
- **Sampled (샘플링 됬다)**: Trace/Span이 전송됨. <mark>전체 데이터의 대표</mark>가 된 것
- **Not Sampled (샘플링 되지 않았다)**: Trace/Span이 전송/처리되지 않은 것

<br/>

---

## 샘플링을 왜 사용할까?

**"비용"과 "접근성/용이성" 측면이 있다.**

근데 접근성/용이성도 개발자의 "비용"과 관련이 있으므로 그냥 **"비용" 때문에 사용한다**라고 볼 수 있다.
- 샘플링은 **"비용"을 줄이는 가장 효과적인 방법 중 하나**이다.
- 대용량 시스템에서 잘 만들어진 트레이싱 샘플링 구조에선 <mark>1% 그 이하의 샘플링 비율로도 나머지 99% 데이터를 매우 정확하게 대표</mark>한다.

<br/>

샘플링이 비용을 줄이기 위한 기능이지만, 사실 <mark>새로운 비용</mark>을 만들기도 한다.
1. **Tail Sampling** 같이 효과적으로 데이터를 샘플링하는데 드는 직접적인 컴퓨팅 비용 (계산 비용)
2. 더 많은 애플리케이션, 시스템, 데이터가 관련될 때, 효과적인 샘플링 방식을 유지하는데 드는 **엔지니어링 비용**
3. 비효과적인 샘플링 기술로 인해 중요한 정보를 놓치는 간접적인 **기회비용**

샘플링 자체는 데이터를 줄여(대표성 있는 데이터만 저장해) 비용을 줄이는데 효과적이나, <mark>제대로 수행되지 않으면 오히려 예상치 못한 비용</mark>이 발생할 수 있다.

<br/>

---

## 언제 샘플링을 사용해야 할까?

### 샘플링이 필요한 경우

- 초당 1000개 이상의 Trace를 생성하고 있는 경우
- 대부분의 Trace 데이터가 데이터 변동이 적은 정상 트래픽을 표현하는 경우
- 오류, 높은 지연시간 같이 문제를 나타내는 일반적인 기준이 있는 경우
- 오류, 높은 지연시간 외에 이러한 정보를 표현할 수 있는 도메인 별 기준이 있는 경우
- 데이터 샘플링 할지/말지를 결정할 수 있는 일반적인 규칙이 있는 경우
- 서비스를 구분할 수 있어 고용량/저용량 서비스를 다르게 샘플링 할 수 있는 경우
- 샘플링 되지 않은 데이터를 저비용 저장 시스템으로 라우팅 할 수 있는 경우
- 전체 예산이 제한적인 경우
- 관찰가능성에 대해 제한된 예산이 있지만 효과적인 샘플링에 시간을 투자할 수 있는 경우

<br/>

### 샘플링이 적합하지 않은 경우

- 매우 적은 Trace 데이터를 생성하고 있는 경우 (초당 수십개 정도)
- 관찰가능성을 데이터 집계로만 사용하고자 하는 경우
- 데이터 삭제를 금지하는 규제가 존재하는 경우

<br/>

---

## 샘플링 방법

샘플링은 크게 2가지 방법으로 나뉜다.

- **Head Sampling**
- **Tail Sampling**

<br/>

### Head Sampling

**가능한 빨리 샘플링 여부를 결정하는 기술**

Trace 전체를 검사하지 않고, Trace/Span을 샘플링 여부 결정 결정한다.

가장 일반적인 Head Sampling은 **Consistent Probability Sampling**이 있으며, **Deterministic Sampling(결정론적 샘플링)**이라고도 한다.

간단히 설명하면 **Trace ID와 샘플링 하는 비율 기반으로 샘플링 여부를 결정하는 방식**이다. 이를 통해 전체 Trace가 누락된 Span 없이 5% 같은 일관된 비율로 샘플링 된다.

<br/>

#### 장점

- 이해 및 구성이 쉽다
- 효율적으로 동작한다. Trace 전체를 볼 필요 없으니 계산이 복잡하지 않다
- Trace 수집 파이프라인 어디서나 수행이 가능하다

<br/>

#### 단점

**전체 Trace 데이터를 기반으로 샘플링 결정을 내리는 것이 아니라는 단점이 있다.**

예를 들어, 오류가 있는 모든 Trace는 샘플링 되도록 보장하고 싶어도 할 수 없다. 이러한 경우에는 **Tail Sampling이 필요**하다.

<br/>

<br/>

### Tail Sampling

**Trace 내의 모든 또는 대부분의 Span을 고려해 샘플링 여부를 결정하는 방식**

Head Sampling에서는 불가능한 Trace/Span의 정보를 이용하여 특정 기준에 따라 샘플링 여부를 결정할 수 있다.

<img alt="Image" src="https://github.com/user-attachments/assets/77737785-d158-49a2-aaea-371fcb90d3c9" />

<br/>

#### 예시

- 오류가 포함된 Trace를 100% 샘플링 하기
- 전체 지연시간 기준으로 어느 이상이 되는 Trace를 샘플링 하기
- Trace 내 하나 이상의 Span에 있는 특정 속성 존재 혹은 값 기준으로 Trace 샘플링 하기 (배포 버전에 맞게 샘플링 등)
- 저용량 서비스에서 오는 Trace, 고용량 서비스에서 오는 Trace를 구분해서 샘플링 비율을 다르게 샘플링 하기

예시에서 알 수 있듯 **Head Sampling보다 훨씬 더 정교하게 샘플링이 가능**하다. 샘플링 필요한 대규모 시스템의 경우, 데이터 볼륨/균형을 맞추기 위해 **Tail Sampling을 사용하는 것이 거의 항상 필요**하다.

<br/>

#### 단점

장점만 있지는 않다.
- **구현의 어려움**: 시스템의 변화에 따라 샘플링 전략도 계속 변화해야 함 (설정하고 잊어버려서는 안됨). 대규모/분산 시스템에서 이러한 전략을 구현하는 규칙도 크고 정교할 수 있음
- **운영의 어려움**: Tail Sampling을 구현하는 컴포넌트는 많은 양의 데이터를 받아들이고 저장할 수 있는 stateful system이여야 함. 트래픽 패턴에 따라 많은 양의 컴퓨팅 노드가 필요할 수 있음. 고로, Tail Sampling 컴포넌트를 모니터링해서 샘플링 로직에 필요한 리소스가 여유로운지 모니터링해야 함
- 관찰가능성을 위해 유료 벤더를 이용하는 경우, 벤더에서 제공하는 Tail Sampling으로 제한될 수 있음

<br/>

### Head + Tail Sampling 조합

**Tail Sampling과 Head Sampling을 따로 사용해야만 하는 것은 아니다.**

Telemetry Pipeline이 과부하 되는 것을 방지하기 위해 함께 사용될 수도 있다. 
매우 많은 양의 Trace 데이터를 생성하는 서비스의 경우, 먼저 **Head Sampling을 통해 Trace를 작은 비율로 줄이고 Telemetry Pipeline에서 Tail Sampling을 이용하여 더 정교한 로직을 통해 Sampling 여부를 결정**할 수 있다.

<br/>

---

## OpenTelemetry Collector 지원

이에 대해서 [OpenTelemetry Collector](https://github.com/open-telemetry/opentelemetry-collector-contrib)에서도 지원한다.
- **Filter processor**: 전달된 정보를 이용해 필터링 여부 결정할 수 있음 (단순 결정)
- **Probabilistic Sampling processor**: 퍼센트 정해놓고 샘플링 → 헤드 샘플링
- **Tail Sampling processor**: 테일 샘플링. 복잡한 조건도 가능하고, probabilistic sampling도 가능함

<br/>

---

## 마무리

트레이스 샘플링은 관찰가능성 시스템에서 비용을 효과적으로 관리하면서도 중요한 정보를 놓치지 않는 핵심 기술이다. 

**Head Sampling**은 구현이 쉽고 효율적이지만 제한적이고, **Tail Sampling**은 더 정교하지만 복잡하고 비용이 많이 든다. 실제 환경에서는 두 방식을 적절히 조합해서 사용하는 것이 일반적이다.

중요한 것은 <mark>샘플링 전략을 한 번 설정하고 끝내는 것이 아니라, 지속적으로 모니터링하고 최적화해야 한다는 점</mark>이다. 시스템이 변화하면 샘플링 전략도 함께 변화해야 효과적인 관찰가능성을 유지할 수 있다.