- [요약](#요약)
- [문화](#문화)
- [기술](#기술)
- [내부 기술력 (AI 활용)](#내부-기술력-ai-활용)
- [운영](#운영)



## 요약

1. APM, 모니터링 도구를 통해 Observability를 확보하고, 이를 기반으로 자원의 효율성을 판단, 의사 결정에 근거를 두는 것을 목표로 한다.
2. MSA(마이크로 서비스 아키텍처) 기반으로 개발을 진행하는 것도 좋지만 Retry, Circuit Breaker, Timeout, Bulkhead Pattern을 적용하여 장애를 최소화하는 것이 중요하다.
3. 자원은 무한정한 것이 아니다. 비용 절감의 책임 또한 있다. (다만, 근거는 모니터링에 쌓인 데이터를 기준으로 → Alarm)

<br/>


## 문화

1. 전문성 및 레퍼런스: 공식 문서 제공 및 Handbook 제공에 필요성
2. 비용 절감을 할 수 있는 AMD CPU 지원에 필요성
3. Self Healing 시스템
    - 모니터링, 데이터 정합성을 위한 프로세싱 자동화, 이중화 구성, Failover 자동화
4. 기술은 실행하는 사람에 의해 증명 된다.
    - 명확한 책임과 집중할 수 있는 조직 체계와 기술 중심의 의사 결정 구조
    - 의사결정에 필요한 명확한 조직적 목표와 기준
    - 합리적인 기술 판단을 말할 수 있는 회의 문화
    - 함께 전문가가 되는 커뮤니티 형성이 필요함 (e.g. OpenStack 기술팀이 생각하는 개발자가 알아야 할 지식, 개발 언어 프레임워크 통일을 통한 트러블 슈팅 공유 문화)

<br/>

## 기술

1. AWS Graviton: ARM 기반의 CPU 개발을 통한 비용 절감
2. AWS DB: 다른 자원 보다 가격을 높게 측정된 자원
    - Relational: Amazon Aurora, Amazon RDS
    - Caching: Amazon Elasticache (redis -> valkey: 오픈 소스 생태계에 기여하는 문화 필요함)
    - Document: Amazon DocumentDB
3. 기본 제공하는 CPU, DB, Storage, Network, Monitoring 의 기능을 좀 더 세분화해서 제공하자.
    - 예:
        1. I/O Optimize 옵션, Cloud Watch, Aurora(블루 그린 배포 지원, 매이저 마이너 업그레이드, ...)
        2. ElasticCache는 오토스케일링, 99.999 가용성, 자동백업 기능 등을 제공

<br/>

## 내부 기술력 (AI 활용)

![img.png](/public/images/aws-summit-seoul-2025-1.png)

1. KMS(사내 지식 저장소를 취합하자) → AI 활용 → 슬랙을 통해 누구나 쉽게 사용

![img.png](/public/images/aws-summit-seoul-2025-2.png)


<br/>

## 운영

1. 모니터링 도구를 통해 자원의 관리 및 APM(Application Performance Management) 도구를 통해 자원의 성능을 관리
    - DataDog Saas 도입을 하면 All in one으로 관리가 가능함. 비용이 발생하지만 이에 대한 가치는 엄청나다고 생각함.
      모니터링 도구를 self hosted 하고 운영하는 과정과 더불어 그 시간에 비즈니스 로직에 대한 고민을 할 수 있다는 것에 강점이 있다.
    - 측정된 데이터를 기반으로 자원의 효율성의 적정성을 판단하자.
2. 모니터링 도구를 기반으로 이해 관계를 기반한 의사 결정을 하자. 즉, 데이터를 기반으로 의사 결정을 하자.

| Level         | 주요 사례                             | 1분당 결제 | 자원 투입 규모      |
| ------------- | ------------------------------------- | ---------- | ------------------- |
| (L) Low       | 01시 이후 심야 시간                   | 50         | 0.8x                |
| (N) Normal    | 평시 대응 단계                        | 100        | 1x                  |
| (H) High      | 선착순 광소, 신상 입고, 카카오톡 발송 | 250        | 2x                  |
| (V) Very High | 아이돌 굿즈, 대형 프로모션, 매월 1일  | 300~500    | 4x                  |
| (S) Special   | 올영세일                              | 500~2000   | 가용 자원 최대 투입 |


이때 모두 증설하는 것이 아니라, 증설이 필요한 자원을 선정하는 것이 중요하다. 추측하지 말고, 모니터링 도구를 기반으로 측정된 데이터만 신뢰하여 결정한다.

- Amazon CloudWatch
- Datadog, Grafana(Self Hosted), Signoz

