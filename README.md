# StudyMate - 개인 학습 관리 서비스

## 1. 컨테이너 소개

### 1.1 서비스 이름

**StudyMate**

### 1.2 한 줄 설명

개인의 학습 목표와 진행 상황을 관리하고 뽀모도로 타이머를 활용해 효율적인 학습을 돕는 학습플래너 서비스입니다.

### 1.3 주제 선정 이유

학습 목표를 지속적으로 관리하고 실제 진행 상황을 바로 확인할 수 있도록  학습플래너 서비스를 기획했습니다.

사용자가 하루의 학습 목표를 직접 등록하고 완료 여부를 관리할 수 있도록 구성하였고 뽀모도로 타이머를 통해 집중 학습과 휴식 시간을 관리할 수 있도록 했습니다.

### 1.4 주요 기능

- **학습 목표 관리**
  - 새로운 학습 목표 추가
  - 목표 완료 및 미완료 상태 변경
  - 등록된 목표 삭제
  - 전체 / 진행 중 / 완료 항목 필터링

- **학습 진행률 확인**
  - 전체 학습 목표 수 표시
  - 완료한 목표 수 표시
  - 학습 목표 완료 비율 표시

- **Pomodoro 타이머**
  - 25분 집중 시간 설정
  - 5분 휴식 시간 설정
  - 타이머 시작, 일시정지, 초기화

- **오늘의 문장 저장**
  - 사용자가 원하는 문장 입력 및 저장
  - 브라우저 LocalStorage를 이용한 저장

※ 학습 목표와 문장은 브라우저 LocalStorage에 저장됩니다. 서버 데이터베이스를 사용하는 방식은 아니므로 다른 브라우저나 기기와 데이터가 공유되지는 않습니다.

---

## 2. 아키텍처 구성도

### 2.1 전체 시스템 구성

사용자가 웹 브라우저에서 StudyMate에 접속하면 OpenShift Route를 통해 요청이 전달됩니다.

Route는 Service로 요청을 전달하고, Service는 Deployment가 관리하는 Pod의 Nginx 컨테이너로 트래픽을 전달합니다. Nginx는 ConfigMap에 저장된 웹 페이지를 사용자에게 제공합니다.

### 2.2 아키텍처 구성도

<img width="1162" height="1034" alt="Image" src="https://github.com/user-attachments/assets/b4e6b3c8-3838-4b83-9fa7-004241451e10" />

### 2.3 사용한 Kubernetes Object 목록 및 역할

| Object | 파일명 | 역할 |
|---|---|---|
| ConfigMap | configmap.yaml | 웹 페이지 HTML 및 설정 데이터 관리 |
| Secret | secret.yaml | 민감한 설정값 관리 |
| Deployment | deployment.yaml | 컨테이너 실행 및 Pod 상태 관리 |
| Service | service.yaml | Pod에 접근하기 위한 내부 네트워크 제공 |
| Route | route.yaml | 외부 사용자가 웹 서비스에 접근할 수 있도록 연결 |

---

## 3. 사용 이미지

### 3.1 컨테이너 이미지 정보

| 항목 | 내용 |
|---|---|
| 이미지 주소 | `registry.access.redhat.com/ubi9/nginx-124` |
| 이미지 태그 | `latest` |
| 전체 이미지 | `registry.access.redhat.com/ubi9/nginx-124:latest` |
| 웹 서버 | Nginx |
| 노출 포트 | 8080/TCP |

### 3.2 이미지 선택 이유

StudyMate는 정적인 HTML, CSS, JavaScript로 구성된 웹 어플리케이션으로, 별도의 백엔드 서버 없이 웹 서버를 통해 제공할 수 있습니다. 따라서 Nginx가 포함된 Red Hat UBI 이미지를 사용하였습니다.

### 3.3 포트 구성

- 컨테이너 포트: 8080
- Service 포트: 8080
- Route를 통한 외부 HTTP/HTTPS 접속

---

## 4. 배포 방법

### 4.1 사전 준비

- OpenShift 계정 및 프로젝트
- OpenShift CLI (`oc`)
- Kubernetes YAML 파일
- Git

### 4.2 OpenShift 로그인

OpenShift 콘솔에서 제공하는 로그인 명령어를 사용합니다.

```bash
oc login <OpenShift API URL> --token=<TOKEN>
```

### 4.3 프로젝트 확인

```bash
oc project
```

현재 사용 중인 OpenShift 프로젝트를 확인합니다.

### 4.4 Kubernetes 리소스 배포

저장소의 루트 디렉터리에서 실행하는 경우:

```bash
oc apply -f students/202325195/k8s/
```

`students/202325195/` 디렉터리로 이동한 경우:

```bash
oc apply -f k8s/
```


### 4.5 배포 상태 확인

Pod 상태 확인:

```bash
oc get pods
```

Deployment 상태 확인:

```bash
oc get deployment studymate
```

Service 확인:

```bash
oc get service studymate
```

Route 확인:

```bash
oc get route studymate
```

전체 리소스 확인:

```bash
oc get all
```

* Pod가 `Running` 상태이고 Deployment의 `READY`가 `1/1`이면 정상 실행 중

### 4.6 접속 경로 및 예상 결과

Route 주소:

https://studymate-abbdd1909-dev.apps.rm1.0a51.p1.openshiftapps.com

브라우저에서 Route 주소에 접속하면 StudyMate 대시보드가 표시됩니다.

정상적으로 접속되면 다음 화면을 확인할 수 있습니다.

- StudyMate 대시보드
- 학습 목표 추가 및 삭제 기능
- 학습 목표 완료 처리 및 진행률
- Pomodoro 타이머
- 오늘의 문장 입력 및 저장 기능

---

## 5. 배포 결과


---

## 6. 어려웠던 점 및 해결 과정

### 6.1 컨테이너 실행 오류

**문제 상황**

초기 배포 과정에서 Pod가 `CrashLoopBackOff` 상태로 진입했습니다.

Pod 로그를 확인한 결과, 예상했던 Nginx 웹 서버 실행 대신 S2I 관련 초기화 메시지가 출력되는 것을 확인했습니다.

**해결 과정**

Deployment의 컨테이너 실행 명령을 다음과 같이 지정했습니다.

```yaml
command: ["nginx"]
args: ["-g", "daemon off;"]
```

이를 통해 Nginx가 포그라운드에서 실행되도록 설정했습니다.

수정된 YAML 파일을 다시 적용하고 Deployment를 재시작한 결과 Pod가 `Running` 상태로 정상 실행되었습니다.

### 6.2 ConfigMap 변경 사항 미반영

**문제 상황**

ConfigMap에 저장된 웹 페이지를 수정했지만 브라우저에서 변경된 화면이 바로 표시되지 않는 문제가 있었습니다.

**해결 과정**

Deployment에서 ConfigMap을 `subPath` 방식으로 마운트하고 있었기 때문에 ConfigMap 변경 후 컨테이너를 재시작하도록 했습니다.

```bash
oc apply -f students/202325195/k8s/configmap.yaml
oc rollout restart deployment/studymate
```

이후 브라우저에서 강력 새로고침(Ctrl + F5)을 실행하여 변경된 화면을 확인했습니다.

## 7. AI 및 참고자료 활용 내역

### 7.1 AI 활용 내역

| 도구 | 활용 목적 |
|---|---|
| ChatGPT | 프로젝트 아이디어 구체화 및 기능 구성 |
| ChatGPT | Kubernetes YAML 파일 작성 및 설정 이해 |
| ChatGPT | OpenShift 배포 과정에서 발생한 오류 분석 및 해결 방법 확인 |
| ChatGPT | README 문서 구성 및 기술 내용 정리 |

AI에게 Kubernetes 및 OpenShift의 개념과 배포 과정을 쉽게 설명해달라고 하여 개념과 배포 과정을 이해했습니다.
또한 실제 명령어를 실행하면서 배포 상태와 오류를 직접 확인하고 AI를 통해 오류를 수정하였습니다.

최종 YAML 파일 적용, 서비스 배포, 브라우저 접속 및 기능 테스트는 직접 수행했습니다.

### 7.2 참고자료

- [OpenShift 공식 문서](https://docs.openshift.com/)
- [Kubernetes 공식 문서](https://kubernetes.io/docs/)
- [Red Hat Universal Base Image 공식 문서](https://catalog.redhat.com/software/base-images)

---

**프로젝트명:** StudyMate  
**학번:** 202325195  
**배포 환경:** OpenShift Developer Sandbox  
**컨테이너 이미지:** Red Hat UBI 9 Nginx 1.24
