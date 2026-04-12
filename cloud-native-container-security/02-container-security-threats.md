# 컨테이너 환경의 보안 위협

1. 컨테이너 보안  
   1.1 컨테이너 보안 위협 개요  
   1.2 CSA Top Threat  
   1.3 호스트 레벨 보안 위협 및 대응방안  

2. 컨테이너 보안 위협 유형  
   2.1 이미지 레지스트리 보안 위협  
   2.2 공급망 공격  
   2.3 악성 컨테이너 이미지  
   2.4 오케스트레이터 보안 위협  
   2.5 권한 상승 공격  
   2.6 컨테이너 탈출  
   2.7 데이터 유출 및 손실  
   2.8 API 보안 위협  
   2.9 IAM 위협  
   2.10 내부자 위협  

3. Kubernetes 보안 위협  
   3.1 Kubernetes 공격 표면  
   3.2 MITRE ATT&CK for Kubernetes  

4. Kubernetes 공격 벡터  
   4.1 ServiceAccount Token 탈취  
   4.2 Kubelet API 악용  
   4.3 RBAC 권한 상승  
   4.4 API Server 노출  
   4.5 Container Escape  
   4.6 Secret 유출  
   4.7 내부 API 악용  

5. Kubernetes 보안 운영 전략  
   5.1 ATT&CK 기반 전술 매핑  
   5.2 방어 우선순위 설정  
   5.3 탐지 및 대응 전략  
   5.4 로그 수집 및 분석 전략  

6. 출처  


<br>


## 컨테이너 보안

### 컨테이너 보안 위협 개요 

컨테이너 환경은 경량성과 빠른 배포라는 장점을 가지지만, 호스트 커널 공유 구조와 동적 오케스트레이션 특성으로 인해 기존 VM 기반 환경과는 다른 보안 위협이 존재합니다. 주요 위협은 다음 네 가지 범주로 구분할 수 있습니다.

- #### 커널 공유 리스크 

컨테이너는 독립된 OS를 가지지 않고 호스트 커널을 직접 공유합니다. 따라서 커널 취약점이 발생할 경우 컨테이너 격리가 무너질 수 있으며, 컨테이너 탈출(Container Escape)을 통해 호스트 시스템까지 침해될 가능성이 존재합니다. 또한 커널 패닉이 발생하면 해당 호스트에서 실행 중인 모든 컨테이너가 동시에 중단되는 단일 장애 지점(SPOF) 문제가 발생할 수 있습니다.  

- #### 네트워크 및 런타임 위협
컨테이너 런타임에는 네트워크를 통해 지속적으로 상호작용하기 때문에, 내부 공격 표면이 확장됩니다. 기본적으로 클러스터 내부 통신이 열려 있는 경우가 많아, 하나의 컨테이너가 침해되면 다른 Pod나 서비스로 수평 이동(lateral movement)이 가능해집니다. 또한 런타임 환경에서 reverse shell 실행, 악성 프로세스 주입, 외부 C2 서버 통신 등의 행위가 발생할 수 있으며, 이는 정적 분석만으로는 탐지하기 어렵습니다.

- #### 이미지 레지스트리 위협
컨테이너는 이미지 기반으로 배포되므로 이미지 자체가 공격 벡터가 됩니다. 신뢰되지 않은 공개 레지스트리에서 이미지를 가져오는 경우 악성 코드가 포함될 수 있으며, 취약한 라이브러리가 포함된 이미지로 인해 SCA(Security Composition Analysis) 이슈가 발생할 수 있습니다. 또한 공격자는 정상 이미지와 유사한 이름을 사용하는 typosquatting 기법이나 CI/CD 파이프라인을 변조하는 공급망 공격을 통해 악성 이미지를 배포할 수 있습니다.

- #### 오케스트레이터 취약점
Kubernetes와 같은 오케스트레이터는 컨테이너 환경 전체를 제어하는 핵심 구성요소로, 설정 오류나 권한 관리 실패 시 클러스터 전체가 위험에 노출됩니다. 특히 RBAC 설정이 과도하게 열려 있는 경우 공격자가 cluster-admin 수준의 권한을 획득할 수 있으며, API Server나 kubelet에 대한 접근을 통해 컨테이너 실행 제어가 가능해집니다. 또한 etcd에 저장된 Secret 데이터가 암호화되지 않았거나 접근 통제가 미흡한 경우 민감 정보 유출로 이어질 수 있습니다.

<br>

### CSA Top Threat
CSA(Cloud Security Alliance)는 클라우드 환경에서 발생할 수 있는 주요 보안 위협을 정리한 Top Threats 보고서를 통해, 조직이 우선적으로 고려해야 할 보안 리스크를 제시합니다. 

- #### 데이터 유출 (Data Breach)
클라우드 환경에서 가장 치명적인 위협으로, 민감한 데이터가 외부로 노출되는 사고를 의미합니다. 주로 접근 통제 미흡, 암호화 부족, 또는 취약한 인증 구조로 인해 발생하며, 컨테이너 환경에서는 잘못된 권한 설정이나 Secret 관리 미흡으로 인해 데이터베이스 자격 증명, API 키 등이 노출될 수 있습니다.

- #### 잘못된 설정 (Misconfiguration)
클라우드 보안 사고의 상당수는 설정 오류에서 발생합니다. 공개된 S3 버킷, 과도하게 열린 보안 그룹, 또는 인증 없이 접근 가능한 API 등이 대표적입니다. 컨테이너 환경에서는 Kubernetes 설정 오류, NetworkPolicy 미적용, privileged 컨테이너 실행 등이 주요 위험 요소로 작용합니다.

- #### 계정 탈취 (Account Hijacking)
피싱, credential stuffing, 키 유출 등을 통해 계정을 탈취하는 공격입니다. 클라우드에서는 단일 계정이 광범위한 리소스에 접근할 수 있기 때문에, 계정 탈취 시 피해 범위가 매우 큽니다. 특히 API Key, Access Token, GitHub Secret 등이 노출될 경우 자동화된 공격으로 이어질 수 있습니다.

- #### 불안전한 API (Insecure APIs)
클라우드 서비스는 API 기반으로 동작하기 때문에 API 보안이 매우 중요합니다. 인증/인가가 제대로 적용되지 않거나 입력값 검증이 부족한 경우, 공격자는 이를 통해 시스템 내부로 침투할 수 있습니다. 특히 컨테이너 환경에서는 내부 서비스 간 API 호출이 많아 공격 표면이 확대됩니다.

- #### 내부자 위협 (Insider Threat)
조직 내부 사용자 또는 권한을 가진 외부 협력자가 의도적 또는 비의도적으로 보안을 위협하는 경우입니다. 클라우드 환경에서는 접근 권한이 넓고 로그 추적이 복잡하기 때문에, 내부자의 오용이나 실수로 인한 피해가 크게 확대될 수 있습니다.


- #### 워크로드 오설정 (Workload Misconfiguration)
컨테이너 및 Kubernetes 워크로드 설정 오류로 인해 보안 위험이 발생하는 경우를 의미합니다. 예를 들어 privileged 컨테이너 실행, root 사용자로 컨테이너 구동, read-only 파일시스템 미적용, 과도한 Linux capability 허용 등이 대표적인 사례입니다. 이러한 설정은 공격자가 컨테이너 내부에서 권한을 확장하거나 호스트 시스템으로 탈출할 수 있는 발판을 제공할 수 있습니다. 또한 리소스 제한(CPU/Memory)이나 보안 정책(Pod Security Standard)이 적용되지 않은 경우, 서비스 장애나 보안 사고로 이어질 가능성이 높습니다.


<br>


### 호스트 레벨 보안 위협 및 대응방안

호스트는 컨테이너가 실행되는 기반 환경으로, 하나의 OS와 커널을 여러 컨테이너가 공유하기 때문에 침해 시 영향 범위가 매우 큽니다. 공격자는 커널 취약점, 잘못된 시스템 설정, 또는 과도한 권한을 가진 컨테이너를 통해 호스트에 접근할 수 있으며, 이는 동일 노드 내 모든 컨테이너로 확산될 수 있습니다. 특히 Docker daemon이나 container runtime에 대한 접근 권한이 노출될 경우, 컨테이너 생성 및 제어 권한까지 탈취되어 시스템 전체 장악으로 이어질 수 있습니다.  

#### 보안 위협
- 커널 취약점 악용을 통한 Container Escape  
- Privileged 컨테이너를 통한 Host 권한 상승  
- Docker socket(`/var/run/docker.sock`) 노출  
- 불필요한 서비스 및 패키지로 인한 공격 표면 증가  
- SSH 및 계정 관리 미흡으로 인한 직접 침투  

#### 대응 방안
- 커널 및 OS 정기 패치 (LTS 기반 운영)  
- privileged 및 root 컨테이너 실행 제한 (최소 권한 원칙 적용)  
- Docker socket 외부 노출 금지 및 접근 제어  
- Host OS Hardening (CIS Benchmark 기반 설정)  
- SSH 접근 통제 강화 (Key 기반 인증, MFA 적용)  
- 런타임 보안 도구(Falco, eBPF 등) 통한 행위 기반 탐지  

<br>

## 컨테이너 보안 위협 유형 (Threat Scenarios)

컨테이너 환경에서는 이미지, 런타임, 오케스트레이터, IAM 등 여러 계층이 결합되며 다양한 공격 시나리오가 발생합니다. 아래는 실무에서 핵심적으로 고려해야 할 주요 위협 유형입니다.  

<br>

### 이미지 레지스트리 보안 위협 (Image Registry Threats)

컨테이너 이미지는 애플리케이션 실행의 출발점이기 때문에, 이미지 자체가 공격 벡터가 될 수 있습니다. 신뢰되지 않은 레지스트리에서 이미지를 가져오거나 검증 없이 배포할 경우 악성 코드가 포함될 가능성이 존재합니다. 특히 공개 레지스트리에서는 정상 이미지와 유사한 이름을 사용하는 typosquatting 공격이 빈번하게 발생하며, 내부 레지스트리 역시 접근 통제가 미흡할 경우 이미지 변조로 이어질 수 있습니다.  

위험 흐름 예시는 다음과 같습니다.  

- 개발자가 외부 이미지를 그대로 사용  
- 이미지에 포함된 취약점 또는 악성 코드 존재  
- 정상 배포 프로세스를 통해 운영 환경 반영  
- 서비스 내부에서 공격 수행  


<br>

### 공급망 공격 (Supply Chain Attacks)

공급망 공격은 애플리케이션 코드가 아닌, 빌드 및 배포 과정 자체를 공격하는 방식입니다. 공격자는 CI/CD 파이프라인, 빌드 서버, 또는 오픈소스 의존성을 타겟으로 하여 정상적인 배포 흐름에 악성 코드를 삽입합니다.  

이 공격의 특징은 다음과 같습니다.  

- 개발자가 아닌 배포 경로를 공격  
- 정상 프로세스를 통해 악성 코드 확산  
- 탐지가 매우 어려움 (정상 코드처럼 보임)  

대표적인 흐름으로는 `빌드 환경 침해 → 이미지 생성 시 악성 코드 삽입 → 정상 배포 → 전체 서비스 감염`  흐름으로 진행됩니다.

<br>

### 악성 컨테이너 이미지 (Malicious Container Images)

악성 컨테이너 이미지는 공격 목적을 가지고 제작된 이미지로, 내부에 백도어, 크립토마이너, 정보 탈취 코드 등이 포함됩니다.  

공격자는 다음과 같은 방식으로 이를 유포합니다.  

- Docker Hub 등에 악성 이미지 업로드  
- 정상 이미지와 유사한 이름 사용 (typosquatting)  
- 인기 이미지 이름을 위장  

사용자가 해당 이미지를 실행하는 순간, 컨테이너는 정상 서비스가 아니라 공격 도구로 동작하게 됩니다. 특히 내부 네트워크 접근 권한을 이용해 lateral movement가 발생할 수 있습니다.  

<br>

### 오케스트레이터 보안 위협 (Kubernetes Security Threats)

Kubernetes는 컨테이너 환경의 제어 계층으로, 공격자가 이 영역을 장악하면 전체 클러스터를 통제할 수 있습니다. 따라서 가장 중요한 공격 표면입니다.  

주요 공격 지점은 다음과 같습니다.  
- RBAC 오설정 → 과도한 권한 획득  
- API Server 노출 → 인증 없이 접근  
- kubelet 접근 → 컨테이너 실행 제어  
- etcd 노출 → Secret 탈취  


이 중 하나라도 침해되면, `Pod 생성/삭제 조작 → 이미지 교체 → 전체 서비스 변조` 와 같이 치명적으로 흘러갈 수 있습니다. 

<br>

### 권한 상승 공격 (Privilege Escalation)

권한 상승은 공격자가 초기 침투 이후 더 높은 권한을 확보하는 단계입니다. 컨테이너 환경에서는 설정 실수로 인해 이 공격이 쉽게 발생할 수 있습니다.  

주요 원인은 다음과 같습니다.  

- privileged 컨테이너 실행  
- root 사용자 기반 컨테이너  
- 과도한 Linux capability 허용  

공격자는 이를 통해 파일 시스템 접근, 디바이스 접근, 커널 기능 사용 권한을 확보하며, 이후 컨테이너 탈출이나 시스템 장악으로 이어집니다.  

권한 상승은 대부분의 공격 시나리오에서 중간 단계로 작용합니다.  

<br>

### 컨테이너 탈출 (Container Escape)

컨테이너 탈출은 격리된 환경을 벗어나 호스트 OS에 접근하는 공격으로, 컨테이너 보안에서 가장 위험한 시나리오 중 하나입니다.  

컨테이너는 호스트 커널을 공유하기 때문에, 커널 취약점이 존재할 경우 공격자는 격리를 우회할 수 있습니다. 탈출이 성공하면 해당 노드에 존재하는 모든 컨테이너에 영향을 줄 수 있습니다.  

공격 흐름은 `컨테이너 내부 침투 → 권한 상승 → 커널 취약점 exploit → Host 접근 → 전체 노드 장악`으로 흘러갑니다. 

<br>

### 데이터 유출 및 손실 (Data Breach & Data Loss)

데이터 유출은 민감한 정보가 외부로 노출되는 위협으로, 실제 비즈니스 피해로 직접 연결됩니다.  

컨테이너 환경에서는 다음과 같은 경로를 통해 발생합니다.  

- 환경 변수에 포함된 Secret 노출  
- 로그에 민감 정보 출력  
- Kubernetes Secret 접근 통제 미흡  
- 외부로 데이터 전송  

특히 클라우드 환경에서는 데이터가 중앙 집중화되어 있기 때문에, 한 번의 유출로 대규모 피해가 발생할 수 있습니다.  

<br>

### API 보안 위협 (API Security Threats)

컨테이너 및 마이크로서비스 환경에서는 API가 핵심 통신 수단이며, 공격 표면이 크게 확장됩니다.  

문제는 내부 API가 외부 API보다 보안이 약하게 설계되는 경우가 많다는 점입니다. 인증 및 인가가 미흡하거나 입력값 검증이 부족할 경우 공격자는 이를 통해 시스템 내부로 침투할 수 있습니다.  

또한 API는 내부 서비스 간 이동 경로로 활용되며 lateral movement의 핵심 역할을 수행합니다.  

<br>

### IAM 위협 (Identity & Access Management Threats)

IAM은 클라우드 및 컨테이너 환경에서 가장 중요한 보안 통제 요소입니다. 하지만 설정 오류가 발생할 경우 공격자가 정상 사용자 권한으로 시스템에 접근할 수 있습니다.  

대표적인 문제는 다음과 같습니다.  

- 과도한 권한 부여 (Over-privileged Role)  
- Access Key 및 Token 노출  
- ServiceAccount 오용  

특히 Kubernetes와 클라우드 IAM이 연계된 환경에서는, 단일 자격 증명 탈취로 전체 인프라 접근이 가능해지는 구조적 위험이 존재합니다.  

<br>

### 내부자 위협 (Insider Threats)

내부자 위협은 조직 내부 사용자 또는 권한을 가진 인원이 보안을 위협하는 경우를 의미합니다.  

컨테이너 환경에서는 자동화된 배포와 광범위한 권한 구조로 인해, 단순한 실수도 대규모 사고로 이어질 수 있습니다. 또한 내부자는 정상 권한을 가지고 있기 때문에 탐지가 어렵고, 로그 및 감사 체계가 부족한 경우 추적이 거의 불가능합니다.  

이 위협은 기술적 대응에 더하여, 정책, 권한 관리, 감사 체계까지 함께 고려되어야 합니다.  

<br>

## Kubernetes 보안 위협


### Kubernetes 공격 표면 (Attack Surface)

Kubernetes 환경은 다양한 구성 요소로 이루어져 있으며, 각 요소는 공격자가 활용할 수 있는 공격 표면으로 작용합니다. 단일 취약점이 아니라 여러 구성 요소가 연결되며 공격이 확장되는 구조를 가지기 때문에, 전체 구조를 이해하는 것이 중요합니다.  

주요 공격 표면은 다음과 같습니다.  

- Kubernetes API Server (클러스터 제어 인터페이스)  
- kubelet (노드 단위 컨테이너 실행 제어)  
- etcd (클러스터 상태 및 Secret 저장소)  
- Container Runtime (Docker, containerd)  
- Pod / Container 내부 애플리케이션  
- 네트워크 계층 (Pod 간 통신, Service)  

이러한 요소들은 각각 독립적인 공격 지점이 아니라, 공격자가 단계적으로 활용하는 연결된 경로로 작용합니다.  

<br>

### MITRE ATT&CK for Kubernetes

MITRE ATT&CK for Kubernetes는 Kubernetes 환경에서 발생하는 공격을 전술(Tactic)과 기술(Technique)로 구조화한 프레임워크입니다. 단순히 취약점을 나열하는 것이 아니라, 공격자가 어떤 흐름으로 시스템을 침해하는지를 단계별로 설명하는 것이 특징입니다.  

컨테이너 환경에서는 초기 침투 이후 권한 상승, 내부 이동, 데이터 유출까지 이어지는 공격 체인이 형성되며, ATT&CK은 이를 다음과 같은 흐름으로 설명합니다.  

- Initial Access → 취약한 Pod 또는 API를 통한 침투  
- Execution → 컨테이너 내부 코드 실행  
- Privilege Escalation → RBAC 및 설정 오류 악용  
- Lateral Movement → 다른 Pod 및 서비스로 이동  
- Persistence → 악성 Pod 생성 및 유지  
- Exfiltration → 데이터 외부 유출  

이 프레임워크를 활용하여 전체 공격 흐름을 기반으로 보안 전략을 설계할 수 있습니다.  

<br>

## Kubernetes 공격 벡터 (Attack Scenarios)

다음은 실제 Kubernetes 환경에서 발생할 수 있는 대표적인 공격 벡터입니다. 각 시나리오는 MITRE ATT&CK 흐름과 연결되어 있으며, 공격자가 어떻게 시스템을 장악하는지 단계적으로 보여줍니다.  

<br>

### ServiceAccount Token 탈취

Pod 내부에 자동으로 마운트되는 ServiceAccount Token을 탈취하여 Kubernetes API에 접근하는 공격입니다.  

공격 흐름:  
1. 공격자가 취약한 컨테이너에 접근  
2. `/var/run/secrets/...` 경로에서 Token 탈취  
3. API Server 호출을 통해 권한 확인  
4. RBAC 범위 내 리소스 접근 및 확장  

결과:  
클러스터 내부 리소스 조회, Pod 생성, 권한 상승으로 이어질 수 있습니다.  

<br>

### Kubelet API 악용

인증이 미흡하게 설정된 kubelet API를 통해 노드 단위 제어 권한을 획득하는 공격입니다.  

공격 흐름:  
1. 외부에서 kubelet 포트 접근  
2. 인증 없이 API 호출  
3. 컨테이너 실행 또는 접근  
4. Host 자원 접근 시도  

결과:  
노드 장악 및 해당 노드 내 모든 컨테이너 제어 가능  

<br>

### RBAC 권한 상승

잘못된 RBAC 설정을 이용하여 공격자가 더 높은 권한을 획득하는 공격입니다.  

공격 흐름:  
1. 제한된 권한 계정 확보  
2. Role/ClusterRoleBinding 확인  
3. 권한 확장 가능한 리소스 탐색  
4. cluster-admin 권한 획득  

결과:  
클러스터 전체 리소스 제어 가능  

<br>

### API Server 노출

인증 및 접근 통제가 미흡한 API Server를 통해 클러스터에 직접 접근하는 공격입니다.  

공격 흐름:  
1. 외부에서 API Server 접근  
2. 인증 우회 또는 취약점 이용  
3. 리소스 조회 및 변경  
4. 악성 Pod 생성  

결과:  
클러스터 전체 제어 및 서비스 변조  

<br>

### Container Escape

컨테이너 내부에서 커널 취약점을 이용하여 호스트로 탈출하는 공격입니다.  

공격 흐름:  
1. 컨테이너 내부 침투  
2. 권한 상승  
3. 커널 취약점 exploit  
4. Host OS 접근  

결과:  
노드 전체 및 모든 컨테이너 장악  

<br>

### Secret 유출 (etcd / env / log)

클러스터 내 저장된 민감 정보가 외부로 노출되는 공격입니다.  

공격 흐름:  
1. etcd 또는 Pod 내부 접근  
2. Secret 데이터 추출  
3. 외부로 전송  

결과:  
DB 계정, API Key 등 민감 정보 유출  

<br>

### 내부 API 악용 (Lateral Movement)

내부 서비스 간 API를 이용하여 공격 범위를 확장하는 시나리오입니다.  

공격 흐름:  
1. 취약한 서비스 침투  
2. 내부 API 호출  
3. 인증 우회 또는 토큰 재사용  
4. 다른 서비스 접근  

결과:  
클러스터 내부 전체로 공격 확산  

<br>

## Kubernetes 보안 운영 전략 (Detection & Response Strategy)

컨테이너 및 Kubernetes 환경에서의 보안은 단순한 취약점 대응이 아닌, 공격 흐름 기반의 탐지 및 대응 전략으로 접근해야 합니다. 특히 MITRE ATT&CK for Kubernetes를 기반으로 공격을 구조화하면, 각 단계별로 필요한 보안 통제를 명확하게 정의할 수 있습니다.  

<br>

### ATT&CK 기반 전술 매핑

각 공격 벡터는 ATT&CK 전술과 매핑될 수 있으며, 이를 통해 공격 흐름을 체계적으로 이해할 수 있습니다.  

- Initial Access → 취약한 컨테이너, API 노출  
- Execution → 컨테이너 내부 명령 실행  
- Privilege Escalation → RBAC 오설정, privileged 컨테이너  
- Lateral Movement → Pod 간 네트워크 이동  
- Persistence → 악성 Pod 재생성  
- Exfiltration → Secret 및 데이터 유출  

이러한 매핑을 통해 단일 이벤트가 아닌, 공격 체인 단위로 분석이 가능합니다.  

<br>

### 방어 우선순위 설정 (Defense Prioritization)

모든 위협을 동일하게 대응하는 것은 비효율적이며, 공격 성공 시 영향도가 높은 영역부터 우선적으로 보호해야 합니다.  

우선순위는 다음 기준으로 설정할 수 있습니다.  

1. Kubernetes Control Plane (API Server, etcd)  
2. IAM 및 인증 체계 (ServiceAccount, Access Token)  
3. Container Runtime 및 Host 접근  
4. 네트워크 정책 및 내부 통신 제어  
5. 이미지 및 CI/CD 공급망  

특히 Control Plane과 IAM 영역은 침해 시 전체 클러스터 장악으로 이어지기 때문에 최우선 보호 대상입니다.  

<br>

### 탐지 및 대응 전략 (Detection & Response)

정적 분석(SAST, SCA)만으로는 런타임 공격을 탐지하기 어렵기 때문에, 행위 기반 탐지가 필수적입니다.  

주요 탐지 전략은 다음과 같습니다.  

- 비정상 프로세스 실행 (reverse shell, crypto miner)  
- Container Escape 시도 (권한 상승, 커널 호출)  
- 비정상 API 호출 패턴 (kubelet, API Server)  
- ServiceAccount Token 사용 이상 징후  
- 네트워크 이상 트래픽 (C2 통신, lateral movement)  

이를 위해 eBPF 기반 런타임 보안 도구(Falco 등)와 로그 수집 시스템을 연계하여 실시간 탐지 체계를 구축할 수 있습니다.  

<br>

### 로그 수집 및 분석 전략 (Log Collection & Analysis)

효과적인 탐지를 위해서는 다양한 계층에서 로그를 수집하고 통합 분석하는 것이 중요합니다.  

수집 대상은 다음과 같습니다.  

- Kubernetes Audit Log  
- Container Runtime Log  
- Application Log  
- CloudTrail / IAM Log  
- Network Flow Log  

이러한 로그를 중앙 저장소(SIEM 또는 DevSecOps Hub)로 수집하여, 공격 패턴 기반 분석 및 이상 탐지를 수행할 수 있습니다.  

<br>
<br>


## 출처 

- https://saas.or.kr/technology/1449
- https://kubernetes.io/ko/docs/concepts/security/overview/
- https://www.redhat.com/ko/topics/security/container-security
- https://www.sentinelone.com/ko/cybersecurity-101/cloud-security/container-security-vulnerabilities/
- https://www.boannews.com/media/view.asp?idx=126448&direct=mobile
- https://www.redhat.com/ko/topics/containers/kubernetes-security
- https://www.linkedin.com/pulse/mitre-attck-meaning-benefits-attack-framework-ec-council/
- https://www.paloaltonetworks.co.kr/cyberpedia/what-is-container-security
- https://velog.io/@yukyung_16/Falco%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EB%9F%B0%ED%83%80%EC%9E%84-%EB%B3%B4%EC%95%88-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81
- https://kubernetes.io/ko/docs/concepts/security/rbac-good-practices/