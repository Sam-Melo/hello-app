# PROJETO CI/CD – GITHUB ACTIONS, DOCKER HUB E ARGOCD

## 1. OBJETIVO
Desenvolver um pipeline de integração e entrega contínua (CI/CD) utilizando o **GitHub Actions**, **Docker Hub** e o **ArgoCD**, aplicando práticas de automação de build, versionamento e deploy em um ambiente **Kubernetes (K3s)** via **Rancher Desktop**.

O projeto visa demonstrar, de forma prática, a automação de todo o ciclo de desenvolvimento, desde o push do código até o deploy automatizado da aplicação em um cluster Kubernetes.

---

## 2. DESCRIÇÃO DO PROJETO
A aplicação desenvolvida é uma **API simples em Python (FastAPI)**, responsável por retornar uma mensagem JSON de teste.

A imagem da aplicação é gerada e publicada automaticamente no **Docker Hub**, e o **ArgoCD** é responsável por sincronizar as alterações do repositório de manifests (`hello-manifests`) com o cluster Kubernetes local.

---

## 3. TECNOLOGIAS UTILIZADAS
- **Python 3.12 (FastAPI)**
- **Docker / Docker Hub**
- **GitHub Actions**
- **Kubernetes (K3s via Rancher Desktop)**
- **ArgoCD**
- **YAML / Git**

---

## 4. ESTRUTURA DO PROJETO

### Repositório `hello-app`
Contém o código-fonte da aplicação e o pipeline CI/CD.

hello-app/
├── main.py
├── requirements.txt
├── Dockerfile
└── .github/
└── workflows/
└── cicd.yml


### Repositório `hello-manifests`
Contém os manifests do Kubernetes:

hello-manifests/
└── k8s/
├── deployment.yaml
├── service.yaml
└── application.yaml


---

## 5. FUNCIONAMENTO DO PIPELINE

1. O desenvolvedor faz um **push no branch main** do repositório `hello-app`.  
2. O **GitHub Actions** inicia automaticamente o workflow (`cicd.yml`).  
3. O job realiza o **build da imagem Docker** e faz o **push para o Docker Hub**.  
4. O pipeline acessa o repositório `hello-manifests` via **SSH (Deploy Key)** e atualiza o `deployment.yaml` com a nova tag da imagem.  
5. O **ArgoCD**, configurado para monitorar o repositório `hello-manifests`, detecta a alteração e executa o **deploy automático** no cluster Kubernetes.  
6. O novo pod é criado com a imagem atualizada, completando o ciclo **CI/CD**.

---

## 6. CONFIGURAÇÕES DE AMBIENTE

### 6.1 Repositório `hello-app`
Secrets configurados em *Settings → Secrets and variables → Actions*:
- `DOCKER_USERNAME` → usuário do Docker Hub  
- `DOCKER_PASSWORD` → token do Docker Hub  
- `SSH_PRIVATE_KEY` → chave privada gerada localmente (sem senha)

### 6.2 Repositório `hello-manifests`
- Chave pública adicionada em *Settings → Deploy Keys*  
  - Título: `gh-actions`
  - **Allow write access** habilitado

---

## 7. ARGOCD – REGISTRO DA APLICAÇÃO

Manifesto `application.yaml`:

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
Aplicado com o comando:

kubectl apply -n argocd -f https://raw.githubusercontent.com/Sam-Melo/hello-manifests/main/k8s/application.yaml
8. TESTE DA APLICAÇÃO
Verifique os recursos criados:


kubectl get deploy,po,svc -l app=hello-app -n default
Faça o port-forward para acesso local:


kubectl port-forward svc/hello-app 8080:80
Acesse:
http://localhost:8080

Saída esperada:

json
Copiar código
{"message": "Hello from CI/CD!"}
9. RESULTADOS
✅ Pipeline automatizado com GitHub Actions e Docker Hub
✅ Deploy contínuo via ArgoCD


✅ Sincronização automática entre repositórios
✅ Ambientes versionados e reproduzíveis


