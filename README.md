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


### 📦 1. Link do repositório com os manifests (`deployment.yaml`, `service.yaml`)  
[https://github.com/Sam-Melo/hello-manifests](https://github.com/Sam-Melo/hello-manifests)

---

### 🧱 2. Evidência de build e push da imagem no Docker Hub  
<img width="1856" height="975" alt="image" src="https://github.com/user-attachments/assets/7f5bc2f6-08ca-45ab-b66e-15ae970a0817" />


---

### 🔁 3. Evidência de atualização automática dos manifests com a nova tag da imagem  
<img width="1858" height="951" alt="image" src="https://github.com/user-attachments/assets/4c8d4f08-11ff-479c-a1a7-eb1111db3483" />


---

### 🧭 4. Captura de tela do ArgoCD com a aplicação sincronizada  
<img width="1919" height="1026" alt="image" src="https://github.com/user-attachments/assets/b3537866-41fe-4d1d-9f39-008785986068" />


---

### 🧩 5. Print do `kubectl get pods` com a aplicação rodando  
<img width="1115" height="624" alt="image" src="https://github.com/user-attachments/assets/e66b13b1-5d83-4739-b2d3-a0cc316ad7d4" />


---

### 🌐 6. Print da resposta da aplicação via navegador  
<img width="623" height="70" alt="image" src="https://github.com/user-attachments/assets/c990848a-51c0-44f9-9499-27cd88f88463" />
