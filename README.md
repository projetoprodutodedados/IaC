# Projeto de Arquitetura de Dados

>Este repositório contém a configuração necessária para montar a infraestrutura de um projeto de arquitetura de dados utilizando Kubernetes (K8s) com Kind e Helm. A infraestrutura inclui serviços como PostgreSQL, Airflow, bucket, ClusterIssuer, nginx, cert-manager, Grafana, Prometheus, Kubeflow, e aplicações front e backend.

## Pré-requisitos

- [Docker](https://www.docker.com/get-started)
- [Kind](https://kind.sigs.k8s.io/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Helm](https://helm.sh/docs/intro/install/)

## Passo a Passo

### 1. Configuração do Kind

1.1. Crie um cluster Kind:
```bash
kind create cluster --name meu-cluster
```

1.2. Verifique se o cluster foi criado corretamente:
```bash
kubectl cluster-info --context kind-meu-cluster
```

### 2. Configuração do Helm
2.1. Adicione os repositórios Helm necessários:
```bash
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add kubeflow https://kubeflow.github.io/manifests
helm repo update
```

### 3. Deploy dos Serviços

3.1. Deploy do PostgreSQL:
```bash
helm install meu-postgres bitnami/postgresql
```

3.2. Deploy do Airflow:
```bash
helm install meu-airflow stable/airflow
```

3.3. Deploy do Nginx:
```bash
helm install meu-nginx stable/nginx-ingress
```

3.4. Deploy do Cert-Manager:
```bash
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.crds.yaml
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace
```

3.5. Deploy do Grafana:
```bash
helm install meu-grafana stable/grafana
```

3.6. Deploy do Prometheus:
```bash
helm install meu-prometheus stable/prometheus
```

3.7. Deploy do Kubeflow:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubeflow/manifests/master/kfdef/kfctl_k8s_istio.v1.2.0.yaml
```


### 4. Deploy das Aplicações
4.1. Deploy do Frontend do Dashboard:
```bash
kubectl apply -f k8s/dashboard-frontend-deployment.yaml
```

4.2. Deploy do Backend do Dashboard:
```bash
kubectl apply -f k8s/dashboard-backend-deployment.yaml
```


4.3. Deploy do Frontend do Formulário:
```bash
kubectl apply -f k8s/formulario-frontend-deployment.yaml
```

4.4. Deploy do Backend do Formulário:
```bash
kubectl apply -f k8s/formulario-backend-deployment.yaml
```

### 5. Configuração do Bucket
5.1. Adicionar repositório do LocalStack ao Helm:
```bash
helm repo add localstack https://helm.localstack.cloud
helm repo update
```

5.2. Deploy do LocalStack:
```bash
helm install localstack localstack/localstack
```

5.3. Configure as credenciais de acesso no Kubernetes:
```bash
kubectl create secret generic bucket-credentials 
--from-literal=access-key=test --from-literal=secret-key=test
```

5.4. Crie o bucket no LocalStack:
```bash
aws --endpoint-url=http://localhost:4566 s3 mb s3://meu-bucket
```

### 6. Configuração do ETL
6.1. Deploy do ETL:
```bash
kubectl apply -f k8s/etl-deployment.yaml
```

## Referências
- [Documentação do Kind](https://kind.sigs.k8s.io/)
- [Documentação do Helm](https://helm.sh/docs/)
- [Documentação do Kubernetes](https://kubernetes.io/docs/home/)
- [Documentação do Kubeflow](https://www.kubeflow.org/docs/)

## Contribuição
Sinta-se à vontade para abrir issues e `pull requests` para melhorias e correções.
