# test-gitops

# FluxCD 적용 방법

이 문서는 FluxCD를 Kubernetes 클러스터에 적용하고 GitOps 방식으로 클러스터를 관리하는 방법을 안내합니다. FluxCD는 Git 리포지토리를 지속적으로 모니터링하며 Kubernetes 클러스터 상태를 자동으로 동기화하는 도구입니다.

---

## 1. FluxCD 설치

### **Prerequisite**
- Kubernetes 클러스터 접근 권한이 있어야 합니다.
- `kubectl`이 설치되어 있어야 합니다.
- Git 리포지토리가 준비되어 있어야 합니다.

### **1.1 Flux CLI 설치**
FluxCD CLI를 설치하려면 다음 명령어를 실행합니다:

```bash
curl -s https://fluxcd.io/install.sh | sudo bash
```

설치가 완료되면 Flux CLI가 정상적으로 설치되었는지 확인합니다:

```bash
flux --version
```

---

### **1.2 FluxCD 설치**
FluxCD를 클러스터에 설치하려면 다음 명령어를 실행합니다:

```bash
flux install
```

이 명령은 다음을 수행합니다:
- FluxCD 컨트롤러를 `flux-system` 네임스페이스에 설치합니다.
- 클러스터에 필요한 기본 설정을 적용합니다.

네임스페이스를 변경하려면 `--namespace` 플래그를 사용합니다:

```bash
flux install --namespace my-namespace
```

---

## 2. Git 리포지토리 연결

### **2.1 GitRepository 리소스 생성**
Git 리포지토리를 모니터링하는 `GitRepository` 리소스를 생성합니다. 다음은 YAML 예제입니다:

```yaml
apiVersion: source.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: my-git-repo
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/your-username/your-repository.git
  ref:
    branch: main
```

위 YAML을 파일로 저장하고 적용합니다:

```bash
kubectl apply -f gitrepository.yaml
```

---

### **2.2 Kustomization 리소스 생성**
Git 리포지토리에서 Kubernetes 리소스를 클러스터에 적용하려면 `Kustomization` 리소스를 생성해야 합니다. 다음은 YAML 예제입니다:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta1
kind: Kustomization
metadata:
  name: my-kustomization
  namespace: flux-system
spec:
  sourceRef:
    kind: GitRepository
    name: my-git-repo
  path: ./manifests
  prune: true
  interval: 5m
```

위 YAML을 파일로 저장하고 적용합니다:

```bash
kubectl apply -f kustomization.yaml
```

---

## 3. FluxCD 동작 확인

### **3.1 리소스 상태 확인**
FluxCD 리소스가 정상적으로 설치 및 동작하는지 확인하려면 다음 명령어를 실행합니다:

```bash
flux get sources git
```

```bash
flux get kustomizations
```

### **3.2 GitOps 적용 확인**
Git 리포지토리에서 리소스를 수정하고 커밋/푸시한 후, FluxCD가 변경 사항을 자동으로 클러스터에 적용했는지 확인합니다:

```bash
kubectl get all -n flux-system
```

---

## 4. 추가 설정

### **4.1 HelmRelease 사용**
Helm 차트를 사용하는 경우 `HelmRelease` 리소스를 생성하여 적용할 수 있습니다. 예제는 아래와 같습니다:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: my-helm-release
  namespace: flux-system
spec:
  releaseName: my-app
  chart:
    spec:
      chart: ./charts/my-app
  interval: 5m
```

### **4.2 모니터링 설정**
FluxCD의 동작 상태를 Prometheus 및 Grafana를 통해 모니터링할 수 있습니다. 자세한 설정 방법은 FluxCD 공식 문서를 참고하세요.

---

## 5. 참고 자료
- **[FluxCD 공식 문서](https://fluxcd.io/docs/)**: FluxCD의 설치, 설정, 사용법, 문제 해결 방법 등을 다룬 공식 문서.
- **[GitOps 소개](https://www.gitops.tech/)**: GitOps의 개념과 원리를 설명하며 FluxCD와 같은 도구가 이를 어떻게 구현하는지 소개.
- **[FluxCD GitHub 저장소](https://github.com/fluxcd/flux2)**: FluxCD의 소스 코드 및 최신 릴리즈 정보 확인.
- **[Kubernetes 공식 문서](https://kubernetes.io/docs/)**: FluxCD와 Kubernetes 리소스 간의 상호작용을 이해하기 위한 자료.
- **[Helm 공식 문서](https://helm.sh/docs/)**: HelmRelease 리소스와 관련된 설정을 참고할 수 있는 자료.
- **[Prometheus 및 Grafana 공식 문서](https://prometheus.io/docs/)**: FluxCD 모니터링 구성과 관련된 설정 가이드.
