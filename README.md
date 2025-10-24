# PROJETO CI/CD – GITHUB ACTIONS, DOCKER HUB E ARGOCD

## 1. OBJETIVO
Implementar um pipeline **CI/CD** usando **GitHub Actions** (build e push de imagem), **Docker Hub** (registro de imagens) e **ArgoCD** (deploy contínuo em **Kubernetes K3s** via **Rancher Desktop**).

---

## 2. DESCRIÇÃO
A aplicação é uma **API Python (FastAPI)** que retorna um JSON simples.  
O pipeline:
1) faz **build** da imagem e **push** para o **Docker Hub**;  
2) atualiza automaticamente o repositório de **manifests** (`hello-manifests`);  
3) o **ArgoCD** detecta e aplica a nova versão no cluster **K3s** (**Rancher Desktop**).

---

## 3. TECNOLOGIAS
- Python 3.12 (FastAPI)  
- Docker / Docker Hub  
- GitHub Actions  
- Kubernetes (K3s – Rancher Desktop)  
- ArgoCD  
- YAML / Git

---

## 4. ESTRUTURA DOS REPOSITÓRIOS

### 4.1 `hello-app` (código e pipeline)
```
hello-app/
├── main.py
├── requirements.txt
├── Dockerfile
└── .github/
    └── workflows/
        └── cicd.yml
```

### 4.2 `hello-manifests` (manifests do Kubernetes)
```
hello-manifests/
└── k8s/
    ├── deployment.yaml
    ├── service.yaml
    └── application.yaml
```

---

## 5. PIPELINE (VISÃO GERAL)
1. **Push** no branch `main` do `hello-app`.  
2. **GitHub Actions**: build e **push** da imagem no Docker Hub.  
3. Pipeline atualiza o `hello-manifests` com a **nova tag**.  
4. **ArgoCD** sincroniza e aplica no cluster **K3s**.  
5. A nova versão sobe em produção (cluster local).

---

## 6. CONFIGURAÇÕES NECESSÁRIAS

### 6.1 Secrets – `hello-app` → *Settings → Secrets and variables → Actions*
- `DOCKER_USERNAME` – usuário Docker Hub  
- `DOCKER_PASSWORD` – token Docker Hub  
- `SSH_PRIVATE_KEY` – chave privada (sem passphrase) correspondente à Deploy Key

### 6.2 Deploy Key – `hello-manifests` → *Settings → Deploy Keys*
- Adicionar a **chave pública** correspondente à `SSH_PRIVATE_KEY`  
- Habilitar **Allow write access**

---

## 7. REGISTRO DA APLICAÇÃO NO ARGOCD
Arquivo `k8s/application.yaml` (no repo `hello-manifests`):
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: hello-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: "https://github.com/Sam-Melo/hello-manifests"
    targetRevision: main
    path: k8s
  destination:
    server: "https://kubernetes.default.svc"
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Aplicado com:
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/Sam-Melo/hello-manifests/main/k8s/application.yaml
```

---

## 8. TESTE DA APLICAÇÃO

Verifique os recursos:
```bash
kubectl get deploy,po,svc -l app=hello-app -n default
```

Port-forward:
```bash
kubectl port-forward -n default svc/hello-app 8080:80
```

Acesse:  
[http://localhost:8080](http://localhost:8080)

Saída esperada:
```json
{"message": "Hello from CI/CD!"}
```

---

## 9. RESULTADOS
- ✅ Build e push automatizados no Docker Hub  
- ✅ Atualização automática do manifesto com a nova tag  
- ✅ ArgoCD sincronizando e aplicando no cluster  
- ✅ Aplicação acessível via Service (port-forward)  

---

## 10. EVIDÊNCIAS
> Coloque os prints na pasta `docs/` do repositório **hello-app** e mantenha os nomes abaixo.  
> (Para trocar o caminho, basta ajustar o link da imagem.)

**Figura 1 – Verificação de virtualização/WSL2 habilitados (Windows)**
![Figura 1](docs/01-wsl-hyperv-ok.png)

**Figura 2 – Rancher Desktop instalado e Kubernetes habilitado**
![Figura 2](docs/02-rancher-desktop-k8s-on.png)

**Figura 3 – `kubectl get nodes` com nó `Ready` (K3s)**
![Figura 3](docs/03-kubectl-get-nodes.png)

**Figura 4 – ArgoCD instalado (`argocd` namespace pronto / pods `Running`)**
![Figura 4](docs/04-argocd-pods-running.png)

**Figura 5 – Secrets do GitHub configurados (DOCKER_USERNAME, DOCKER_PASSWORD, SSH_PRIVATE_KEY)**
![Figura 5](docs/05-github-secrets.png)

**Figura 6 – Deploy Key com write access no repositório `hello-manifests`**
![Figura 6](docs/06-deploy-key-hello-manifests.png)

**Figura 7 – Execução “Build and push Docker image” no GitHub Actions (sucesso)**
![Figura 7](docs/07-actions-build-push.png)

**Figura 8 – Imagem publicada no Docker Hub com tag do commit**
![Figura 8](docs/08-dockerhub-tags.png)

**Figura 9 – Commit automático no `hello-manifests` atualizando `deployment.yaml`**
![Figura 9](docs/09-commit-manifests-update.png)

**Figura 10 – ArgoCD Application `hello-app` em estado Synced/Healthy**
![Figura 10](docs/10-argocd-synced-healthy.png)
