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
