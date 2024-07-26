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
2.1. Instale o Helm:
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

2.2. Adicione os repositórios Helm necessários:
```bash
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add kubeflow https://kubeflow.github.io/manifests
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo add apache-airflow https://airflow.apache.org
helm repo update
```

### 3. Deploy dos Serviços

3.1. Deploy do PostgreSQL:

Você pode personalizar ainda mais a instalação do PostgreSQL usando um arquivo `values.yaml`:
```
postgresqlUsername: myUser
postgresqlPassword: myPassword
postgresqlDatabase: myDatabase
persistence:
  enabled: true
  size: 8Gi
```

E então instalar o PostgreSQL com o arquivo de valores:
```bash
helm install my-postgresql bitnami/postgresql -f values.yaml
```

3.2. Deploy do Airflow:

Você pode configurar o Airflow através do `values.yaml` para ajustar as configurações de acordo com suas necessidades.

Exemplo de `values.yaml` para Airflow:

```
executor: "CeleryExecutor"
airflow:
  image:
    repository: apache/airflow
    tag: "2.1.2"
  config:
    AIRFLOW__CORE__LOAD_EXAMPLES: "False"
    AIRFLOW__CORE__EXECUTOR: "CeleryExecutor"
    AIRFLOW__CELERY__BROKER_URL: "redis://redis:6379/0"
    AIRFLOW__CELERY__RESULT_BACKEND: "db+postgresql://airflow:airflow@postgresql/airflow"
redis:
  enabled: true
postgresql:
  enabled: true
```

Aplique o `values.yaml`:
```bash
helm install airflow apache-airflow/airflow -f values.yaml --namespace airflow
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

3.4.1. Configurar um Issuer ou ClusterIssuer:

Você precisa configurar um Issuer ou ClusterIssuer para gerenciar os certificados. Para isso, crie o arquivo `cluster-issuer.yaml`:

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

Aplique o ClusterIssuer:
```bash
kubectl apply -f cluster-issuer.yaml
```

3.4.2. Configurar o Ingress para HTTPS
Crie o aqruivo `ingress.yaml` para configuração de Ingress para uma aplicação com HTTPS:
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dashboard-backend-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - your-domain.com
    secretName: dashboard-backend-tls
  rules:
  - host: your-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: dashboard-backend
            port:
              number: 5000
```

Aplique o Ingress:
```bash
kubectl apply -f ingress.yaml
```

3.5. Deploy do Grafana:
```bash
helm install meu-grafana stable/grafana
```

3.6. Deploy do Prometheus:
```bash
helm install meu-prometheus stable/prometheus
```

Para acessar o Prometheus, você pode usar o kubectl port-forward:
```bash
kubectl port-forward svc/prometheus-server 9090:80
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

## Referências
- [Documentação do Kind](https://kind.sigs.k8s.io/)
- [Documentação do Helm](https://helm.sh/docs/)
- [Documentação do Kubernetes](https://kubernetes.io/docs/home/)
- [Documentação do Kubeflow](https://www.kubeflow.org/docs/)

## Contribuição
Sinta-se à vontade para abrir issues e `pull requests` para melhorias e correções.
