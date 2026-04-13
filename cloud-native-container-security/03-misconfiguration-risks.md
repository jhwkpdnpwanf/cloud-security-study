# Misconfiguration 위험


## 목차  

1. Misconfiguration 개요  
   1.1 Kubernetes OWASP Risk Top 10 변화 분석 (2022 → 2025)  
   1.2 주요 변화 분석  
   1.3 공격 흐름 변화 (Threat Evolution)  
   1.4 Misconfiguration의 역할  
   1.5 보안 전략 방향  

2. Misconfiguration 사례  
   2.1 노출된 etcd 서버 위험  
   2.2 etcd at-rest 암호화 미설정  
   2.3 감사 로그(Audit Log) 미설정  
   2.4 Privileged 컨테이너 실행  
   2.5 Root 사용자 컨테이너 실행  
   2.6 HostPath 마운트 남용  

3. Misconfiguration과 IaC의 필요성  

4. 출처  


<br>

## Misconfiguration 개요 

### Kubernetes OWASP Risk Top 10 변화 분석 (2022 → 2025)

Kubernetes 환경에서의 보안 위협은 설정 오류(Misconfiguration)를 중심으로 지속적으로 발생해왔으며, OWASP Kubernetes Top 10의 변화는 이러한 보안 트렌드의 방향성을 명확하게 보여줍니다.  

다음은 2025년 Kubernetes Top 10을 기준으로, 2022 대비 변화된 내용을 정리한 것입니다.  

| 2025 | 2022 대응 항목 | 변화 의미 |
|------|---------------|----------|
| K01: Insecure Workload Configurations | K01 동일 | 여전히 가장 핵심적인 보안 문제입니다 |
| K02: Overly Permissive Authorization | K03 (RBAC) | 권한 문제는 지속되며 더욱 중요해지고 있습니다 |
| K03: Secrets Management Failures | K08 | Secret 관리의 중요성이 유지되고 있습니다 |
| K04: Lack of Cluster Policy Enforcement | K04 | 정책 기반 보안 필요성이 지속되고 있습니다 |
| K05: Missing Network Segmentation Controls | K07 | 네트워크 격리 문제는 여전히 주요 위협입니다 |
| K06: Overly Exposed Kubernetes Components | (신규) | 외부 노출 및 공격 표면이 확대되었습니다 |
| K07: Misconfigured and Vulnerable Components | K09 + K10 | 구성 오류와 취약 컴포넌트 문제가 통합되었습니다 |
| K08: Cluster to Cloud Lateral Movement | (신규) | 클라우드 확장 공격이 등장하였습니다 |
| K09: Broken Authentication Mechanisms | K06 | 인증 취약점 문제는 여전히 지속되고 있습니다 |
| K10: Inadequate Logging and Monitoring | K05 | 탐지 및 대응의 중요성이 증가하고 있습니다 |


<br>

### 주요 변화 분석

2025년에는 전체적으로 공격자의 실제 행동 흐름과 클라우드 환경을 반영하는 방향으로 발전하였습니다.  

특히 다음 두 가지 변화가 핵심입니다.  

- Kubernetes 내부 문제에서 클라우드 연계 문제로 확장
- 정적 설정 취약점에서 공격 흐름 기반 보안으로 전환 

대표적으로 다음 항목이 새롭게 강조되고 있습니다.  

- Overly Exposed Kubernetes Components  
- Cluster to Cloud Lateral Movement  

이는 클라우드 인프라 전체 공격 체인의 일부로 인식되고 있음을 의미합니다.  

<br>

### 공격 흐름 변화 (Threat Evolution)

2022년까지의 공격은 주로 Kubernetes 내부에서 종료되는 구조였습니다.  

Container → Pod → Node → Cluster  

하지만 2025년에는 공격 흐름이 확장되었습니다.  

Container → Pod → Node → Cluster → Cloud IAM → External Resource  

이러한 변화는 다음을 의미합니다.  

- Kubernetes는 더 이상 종점이 아니라 중간 거점입니다  
- 단일 취약점이 전체 클라우드 침해로 이어질 수 있습니다  
- IAM, API, 네트워크가 통합된 공격 경로가 형성되고 있습니다  

<br>

### Misconfiguration의 역할

이러한 모든 공격 흐름의 시작점은 대부분 Misconfiguration입니다.  

대표적인 사례는 다음과 같습니다.  

- Privileged 컨테이너 실행  
- 과도한 RBAC 권한 설정  
- API Server 외부 노출  
- NetworkPolicy 미적용  
- Secret 관리 미흡  

Misconfiguration은 단순한 설정 실수가 아니라, 공격자가 내부 진입 및 확장을 수행할 수 있는 초기 침투 지점(foothold)을 제공합니다.  

특히 Kubernetes 환경에서는 선언형 설정(YAML)과 자동화된 배포가 결합되기 때문에, 하나의 잘못된 설정이 대규모 환경 전체에 빠르게 확산될 수 있습니다.  

<br>

### 보안 전략 방향

Kubernetes 보안에서 Misconfiguration은 전체 공격 체인의 출발점으로 작용합니다.  

따라서 보안 전략은 다음과 같이 설계되어야 합니다.  

- 설정 단계에서의 사전 예방 (Shift Left)  
- 배포 이후 런타임 탐지 (Shift Right)  
- ATT&CK 기반 공격 흐름 대응  

결과적으로 Misconfiguration을 통제하는 것은 Kubernetes 보안의 핵심이며, 이는 단일 기술이 아닌 정책, 자동화, 그리고 운영 전략이 결합된 접근이 필요합니다.  

<br>

## Misconfiguration 사례

Misconfiguration은 Kubernetes 환경에서 가장 빈번하게 발생하는 보안 문제이며, 실제 공격 시나리오의 시작점으로 작용합니다.  

<br>

### 노출된 etcd 서버 위험

etcd는 Kubernetes 클러스터의 모든 상태 정보와 Secret 데이터를 저장하는 핵심 구성요소입니다.  

하지만 etcd가 외부 네트워크에 노출되거나 접근 통제가 미흡한 경우, 공격자는 인증 없이 내부 데이터를 직접 조회할 수 있습니다.  

이 경우 다음과 같은 피해가 발생할 수 있습니다.  

- Secret (DB 계정, API Key 등) 탈취  
- 전체 클러스터 상태 정보 유출  
- 인증 정보 기반 추가 공격 수행  

etcd는 반드시 내부 네트워크에 격리되어야 하며, 접근 제어 및 인증 설정이 필수적으로 적용되어야 합니다.  

<br>

### etcd at-rest 암호화 미설정

Kubernetes에서 Secret은 기본적으로 etcd에 평문 형태로 저장됩니다.  

따라서 at-rest 암호화가 적용되지 않은 경우, etcd 접근만으로 모든 민감 정보가 노출될 수 있습니다.  

이는 다음과 같은 상황에서 특히 위험합니다.  

- etcd 백업 파일 유출  
- 내부 사용자 또는 공격자의 접근  
- 클러스터 권한 상승 이후 데이터 조회  

at-rest 암호화는 Secret 보호를 위한 기본 보안 설정이며, 미적용 시 데이터 유출 위험이 매우 높습니다.  

<br>

### 감사 로그(Audit Log) 미설정

Kubernetes Audit Log는 클러스터 내에서 발생하는 모든 API 요청을 기록하는 핵심 보안 요소입니다.  

Audit Log가 활성화되어 있지 않은 경우 다음과 같은 문제가 발생합니다.  

- 공격 행위 추적 불가능  
- 권한 오용 및 내부자 행위 식별 불가  
- 사고 발생 이후 포렌식 분석 불가  

특히 Kubernetes 환경에서는 API 기반 제어가 중심이기 때문에, Audit Log는 탐지 및 대응의 필수 요소입니다.  

<br>

### Privileged 컨테이너 실행

Privileged 컨테이너는 호스트와 거의 동일한 수준의 권한을 가지며, 컨테이너 격리 구조를 무력화할 수 있습니다.  

이러한 설정이 존재할 경우 공격자는 다음과 같은 행위를 수행할 수 있습니다.  

- Host 파일 시스템 접근  
- 커널 기능 호출  
- 디바이스 제어  

이는 Container Escape로 이어질 수 있으며, 동일 노드 내 모든 컨테이너에 영향을 미칠 수 있습니다.  

<br>

### Root 사용자 컨테이너 실행

컨테이너 내부에서 root 사용자로 실행되는 경우, 공격자는 더 높은 권한으로 시스템을 제어할 수 있습니다.  

특히 다음과 같은 위험이 존재합니다.  

- 권한 상승 공격 용이  
- 파일 시스템 및 프로세스 제어 가능  
- 보안 격리 약화  

root 실행은 최소 권한 원칙에 위배되며, 반드시 제한되어야 합니다.  

<br>

### HostPath 마운트 남용

HostPath 볼륨은 컨테이너가 호스트 파일 시스템에 직접 접근할 수 있도록 허용합니다.  

이 설정이 남용될 경우 다음과 같은 위험이 발생합니다.  

- 민감한 시스템 파일 접근 (/etc, /var 등)  
- 컨테이너 탈출 가능성 증가  
- 호스트 기반 공격 수행  

특히 HostPath는 매우 강력한 권한을 가지기 때문에, 필요한 경우에만 제한적으로 사용되어야 합니다.  

<br>

## Misconfiguration과 IaC의 필요성

위에서 살펴본 Misconfiguration 사례들은 대부분 설정 단계에서 발생하며, 운영 환경에 반영된 이후에는 탐지 및 대응이 어려운 경우가 많습니다.  

특히 Kubernetes 환경에서는 YAML 기반 선언형 설정이 반복적으로 사용되기 때문에, 동일한 설정 오류가 대규모 환경에 빠르게 확산될 수 있습니다.  

이러한 문제를 해결하기 위해서는 설정을 코드로 관리하고, 배포 이전에 검증하는 방식이 필요합니다.  

즉, Infrastructure as Code(IaC)를 기반으로 한 보안 검증이 필수적입니다.  

IaC 기반 접근은 다음과 같은 장점을 제공합니다.  

- 설정 변경 이력 추적 가능  
- 자동화된 보안 검증 수행  
- 잘못된 설정의 사전 차단  
- 일관된 보안 정책 적용  

따라서 Kubernetes 보안은 코드 기반 관리에서 시작하여, 자동화된 검증과 정책 강제를 통해 안전한 배포를 보장하는 방향으로 발전해야 합니다.  

이는 이후 다룰 IaC 기반 보안 통제와 DevSecOps 환경에서 핵심적인 역할을 수행합니다.  


<br>
<br>

## 출처 

- https://owasp.org/www-project-kubernetes-top-ten/  
- https://www.sentinelone.com/blog/mastering-kubernetes-security-top-strategies-recommended-by-owasp-2/  
- https://www.armosec.io/blog/kubernetes-misconfigurations/
- https://www.dynatrace.com/news/blog/kubernetes-misconfiguration-attack-paths-and-mitigation/