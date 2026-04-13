# IaC (Infrastructure as Code) 개념과 보안

## 목차  

1. IaC 개요  
   1.1 IaC란 무엇인가  
   1.2 선언형 vs 절차형 모델  

2. IaC가 필요성과 장점  
   2.1 배경  
   2.2 IaC 장점  

3. IaC 보안 (IaC Security)  
   3.1 주요 보안 위협  
   3.2 Misconfiguration의 위험성  

4. IaC 보안 대응 전략  
   4.1 SAST for IaC  
   4.2 Policy as Code  
   4.3 DevSecOps 통합  

5. 실무 관점의 IaC  

6. 결론  

7. 출처  


<br>

## IaC 개요

### IaC란 무엇인가

IaC는 서버, 네트워크, 스토리지, IAM 정책과 같은 인프라를 코드로 선언하고, 이를 기반으로 자동으로 환경을 구성하는 기술입니다.  

기존 방식에서는 관리자가 콘솔이나 CLI를 통해 수동으로 설정을 진행했습니다.  
이 과정은 반복 작업이 많고, 환경 간 설정 차이가 발생하기 쉬웠습니다.  

IaC는 이러한 문제를 해결합니다.  

- 동일한 코드를 실행하면 동일한 환경이 생성됩니다.  
- 설정이 코드로 남기 때문에 추적이 가능합니다.  
- 자동화를 통해 운영 효율성이 크게 향상됩니다.  


<br>

### 선언형 vs 절차형 모델

#### 선언형(Declarative)

원하는 최종 상태(State)를 정의합니다.  
도구가 현재 상태를 분석하고 목표 상태로 맞춥니다.  

- Terraform  
- AWS CloudFormation  

장점: 간결하고 유지보수가 용이합니다.  

<br>

#### 절차형(Imperative)

인프라를 어떻게 구성할지 절차를 정의합니다.  

- Ansible  
- Shell Script 기반 구성  

장점: 세밀한 제어가 가능합니다.  

<br>

## IaC가 필요성과 장점

IaC는 클라우드 환경의 구조적 변화 때문에 등장한 필수 기술입니다.  

### 배경

- 클라우드의 On-demand 리소스 생성  
- 마이크로서비스 기반 아키텍처 확산  
- DevOps / CI-CD 환경 정착  
- 빠른 배포 요구 증가  

이러한 환경에서는 수동 운영을 사용하는 것은 구조적으로 매우 어렵습니다.

<br>

### IaC 장점

#### 자동화 (Automation)

IaC는 인프라 생성 과정을 완전히 자동화합니다.  

- 반복 작업 제거  
- 배포 속도 향상  
- 운영 비용 절감  

CI/CD와 결합하면 코드 변경 시 인프라도 함께 배포됩니다.  

<br>

#### 일관성 (Consistency)

IaC의 가장 중요한 장점은 환경 일관성 보장입니다.  

| 구분 | 기존 방식 | IaC |
|------|----------|-----|
| 환경 차이 | 발생 | 없음 |
| 설정 재현 | 어려움 | 가능 |
| 오류 발생 | 높음 | 낮음 |


<br>

#### 확장성과 재사용성

IaC는 코드이기 때문에 재사용이 가능합니다.  

- 모듈화된 인프라 구성  
- 템플릿 기반 배포  
- 멀티 환경 적용  

특히 대규모 인프라에서 효과가 극대화됩니다.  

<br>

#### 코드리뷰 & 감사추적 강화

git 기반으로 형상관리를 할 수 있으며, 인프라 변경 사항을 동료와 검토하고 품질과 보안성을 검증할 수 있습니다.  

- 모든 변경 기록 추적  
- 롤백 가능  
- 코드 리뷰 기반 검증  

이는 누가 언제 무엇을 변경했는지 커밋 로그를 통해, 투명하게 확인이 가능하므로 컴플라이언스 대응에 용이합니다.  

<br>

## IaC 보안 (IaC Security)

IaC는 강력한 도구이지만, 동시에 대규모 보안 리스크를 내포합니다.  

잘못된 설정이 코드에 포함되면  
해당 취약점이 자동으로 전체 환경에 배포됩니다.  

<br>

### 주요 보안 위협

#### Misconfiguration (설정 오류)

가장 흔하고 위험한 문제입니다.  

- Public S3 Bucket  
- 0.0.0.0/0 오픈  
- 과도한 IAM 권한  
- 암호화 미적용  

<br>

#### Secret 노출

IaC 코드에 민감 정보가 포함될 수 있습니다.  

- API Key  
- DB Password  
- Access Token  

<br>

#### 공급망 공격 (Supply Chain Attack)

외부 모듈 사용 시 발생합니다.  

- Terraform module  
- 오픈소스 템플릿  

<br>

### Misconfiguration의 위험성

IaC에서는 설정 오류가 확산됩니다.  

- 단일 실수가 전체 인프라에 적용됨  
- 자동화로 인해 빠르게 배포됨  
- 탐지 전까지 지속적으로 유지됨  

즉, IaC는 보안 사고의 증폭기가 될 수 있습니다.  

<br>

## IaC 보안 대응 전략

### SAST for IaC

IaC 코드도 정적 분석 대상입니다.  

- Checkov  
- tfsec  
- Terrascan  

CI 단계에서 자동 실행하는 것이 핵심입니다.  

<br>

### Policy as Code

보안 정책을 코드로 정의합니다.  

- OPA (Open Policy Agent)  
- Sentinel  

정책 위반 시 배포를 차단할 수 있습니다.  

<br>

### DevSecOps 통합

IaC 보안은 CI/CD 파이프라인에 포함되어야 합니다.  

1. 코드 Commit  
2. IaC 정적 분석  
3. 정책 검증  
4. 승인 후 배포  

SAST / SCA / DAST와 함께 통합 운영됩니다.  

<br>

## 실무 관점의 IaC

실제 환경에서는 다음 구조가 일반적입니다.  

- Terraform 기반 인프라 정의  
- GitHub Actions CI/CD  
- AWS OIDC 인증  
- S3 기반 결과 저장  

이 구조는 보안성과 확장성을 동시에 확보합니다.  

<br>

## 결론

IaC는,   

- 운영 효율성  
- 인프라 일관성  
- 보안 및 거버넌스  

이 세 가지를 동시에 해결하는 핵심 기술입니다.  

하지만 잘못 설계할 경우  
대규모 보안 사고로 이어질 수 있습니다.  

따라서 IaC는 반드시 다음과 함께 사용되어야 합니다.  

- 자동화된 보안 검증  
- 정책 기반 통제  
- DevSecOps 통합  

<br>
<br>


## 출처  

- https://wikidocs.net/341069
- https://aws.amazon.com/ko/what-is/iac/
- https://yozm.wishket.com/magazine/detail/2464/
- https://www.redhat.com/en/topics/automation/what-is-infrastructure-as-code-iac