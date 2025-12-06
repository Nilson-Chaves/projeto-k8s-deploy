# Projeto 01 — Deploy de Aplicação Full Stack com Kubernetes
Aplicação Flask + PostgreSQL + React
Ambiente: KIND + Kubernetes + Docker

Autores:
- Nilson Vinícius Aurelio Chaves — Matrícula 20221380002
- José Wilquer Nascimento de Lima — Matrícula 20212380024

----------------------------------------------------------------------
1. VISÃO GERAL DO PROJETO
----------------------------------------------------------------------

Este projeto realiza o deploy completo de uma aplicação web full-stack utilizando Kubernetes, seguindo as boas práticas exigidas pelo professor.

A aplicação é composta por:

- Frontend: React + Vite
- Backend: Flask + SQLAlchemy
- Banco de dados: PostgreSQL (com persistência)
- Ingress Controller: NGINX
- Orquestração: KIND + Kubernetes

A arquitetura atende aos seguintes pontos:

- Deploys com alta disponibilidade (2 réplicas no frontend e backend)
- Separação por namespaces
- Persistência com PVC para o banco
- ConfigMaps e Secrets para variáveis
- Ingress para roteamento externo
- Backend resiliente a reinícios do banco
- Imagens personalizadas com Docker

----------------------------------------------------------------------
2. ARQUITETURA FINAL
----------------------------------------------------------------------

Namespaces:
- app: backend, frontend, ingress, configmaps, secrets
- db: banco PostgreSQL + pvc + secrets

Componentes:
- Frontend Deployment (2 réplicas)
- Backend Deployment (2 réplicas)
- PostgreSQL StatefulSet
- PVC para armazenamento persistente
- Ingress NGINX
- Services para comunicação interna

----------------------------------------------------------------------
3. ESTRUTURA FINAL DO PROJETO
----------------------------------------------------------------------

projeto-k8s-deploy/
├── backend/
│   ├── configmap.yaml
│   ├── deployment.yaml
│   └── secret.yaml
│
├── frontend/
│   └── deployment.yaml
│
├── database/
│   ├── pvc.yaml
│   ├── secret.yaml
│   └── statefulset.yaml
│
├── ingress/
│   └── ingress.yaml
│
├── namespace.yaml
└── README.md

----------------------------------------------------------------------
4. PREPARAÇÃO DO AMBIENTE
----------------------------------------------------------------------

4.1 Criar o cluster KIND

kind create cluster --name k8s-atividade --config kind-config.yaml

4.2 Construir as imagens do projeto

Backend:
cd Projeto01/backend
docker build -t nilsonchaves/backend-k8s:v3 .
kind load docker-image nilsonchaves/backend-k8s:v3 --name k8s-atividade

Frontend:
cd Projeto01/frontend
docker build -t nilsonchaves/frontend-k8s:v1 .
kind load docker-image nilsonchaves/frontend-k8s:v1 --name k8s-atividade

----------------------------------------------------------------------
5. DEPLOY NO KUBERNETES
----------------------------------------------------------------------

5.1 Namespaces:
kubectl apply -f namespace.yaml

5.2 Banco de Dados:
kubectl apply -f database/secret.yaml
kubectl apply -f database/pvc.yaml
kubectl apply -f database/statefulset.yaml

5.3 Backend:
kubectl apply -f backend/configmap.yaml
kubectl apply -f backend/secret.yaml
kubectl apply -f backend/deployment.yaml

5.4 Frontend:
kubectl apply -f frontend/deployment.yaml

5.5 Ingress:
kubectl apply -f ingress/ingress.yaml

----------------------------------------------------------------------
6. TESTANDO A APLICAÇÃO
----------------------------------------------------------------------

6.1 Testar backend via Ingress:
curl http://localhost/api/mensagens

6.2 Inserir mensagem:
curl -X POST http://localhost/api/mensagens \
  -H "Content-Type: application/json" \
  -d '{"texto":"Mensagem de teste"}'

----------------------------------------------------------------------
7. TESTE DE RESILIÊNCIA (REQUISITO)
----------------------------------------------------------------------

Reiniciar o banco:
kubectl delete pod postgres-0 -n db

Aguardar:
sleep 15

Testar novamente:
curl http://localhost/api/mensagens

O backend reconecta automaticamente ao banco e os dados persistem via PVC.

----------------------------------------------------------------------
8. VALIDAÇÃO DOS REQUISITOS
----------------------------------------------------------------------

- Deploy com Deployment (frontend e backend): OK
- 2 réplicas para alta disponibilidade: OK
- StatefulSet para PostgreSQL: OK
- Namespaces separados (app/db): OK
- Ingress Controller configurado: OK
- Rota "/" (frontend) e "/api" (backend): OK
- ConfigMaps para variáveis: OK
- Secrets para credenciais: OK
- PVC funcionando com persistência real: OK
- Backend funcional após reinício do banco: OK
- Testes manuais via curl funcionando: OK

----------------------------------------------------------------------
9. CONCLUSÃO
----------------------------------------------------------------------

O projeto cumpre 100% dos requisitos solicitados.
A arquitetura implementada segue boas práticas de Kubernetes, com:

- Alta disponibilidade
- Banco de dados persistente
- Segurança com Secrets
- Separação lógica via namespaces
- Tráfego externo gerenciado via Ingress
- Resiliência garantida mesmo após falha do banco
- Testes realizados e aprovados
