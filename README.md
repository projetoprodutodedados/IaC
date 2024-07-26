Passo 6: Acessar os Serviços
Para acessar os serviços, você pode usar o kubectl port-forward ou configurar um Ingress Controller.

Passo 7: Configurar Persistência (Opcional)
Para persistência de dados, você pode configurar PersistentVolumeClaims (PVCs) para o PostgreSQL e MinIO.

Passo 4: Deploy dos Charts
Para fazer o deploy dos seus charts, você pode usar o comando helm install.

helm install dashboard-backend ./mychart

Passo 5: Configurar o Prometheus para Monitorar seus Serviços
Você pode configurar o Prometheus para monitorar seus serviços adicionando anotações nos seus deployments para que o Prometheus saiba onde encontrar as métricas.

Por exemplo, adicione as seguintes anotações no seu deployment.yaml:

metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "5000"

    Passo 6: Verificar os Recursos
Verifique se todos os recursos foram criados corretamente:

kubectl get pods
kubectl get services
Passo 7: Acessar o Prometheus
Para acessar o Prometheus, você pode usar o kubectl port-forward:

kubectl port-forward svc/prometheus-server 9090:80
Agora você pode acessar o Prometheus em http://localhost:9090.

Aplique o ClusterIssuer:

Passo 4: Configurar o Ingress para HTTPS
Aqui está um exemplo de configuração de Ingress para uma aplicação com HTTPS:

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
Aplique o Ingress:

kubectl apply -f ingress.yaml


kubectl apply -f cluster-issuer.yaml

Passo 2: Configurar o Airflow
Você pode configurar o Airflow através do values.yaml para ajustar as configurações de acordo com suas necessidades.

Exemplo de values.yaml para Airflow

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
Aplique o values.yaml:

helm install airflow apache-airflow/airflow -f values.yaml --namespace airflow
Seguindo esses passos, você conseguirá configurar HTTPS para suas aplicações e deployar uma aplicação open source para ETL usando o Helm. Se precisar de mais alguma coisa, estou à disposição!

helm repo add kubeflow https://kubeflow.github.io/manifests >>>> deu problema!!

Podemos resolver por aqui: https://github.com/kubeflow/kubeflow/issues/3173

Passo 4: Verificar a Instalação
Verifique se todos os pods estão em execução:




kubectl get pods -n kubeflow
Passo 5: Acessar o Kubeflow Dashboard
Para acessar o dashboard do Kubeflow, você pode usar o kubectl port-forward:




kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
Agora você pode acessar o Kubeflow Dashboard em http://localhost:8080.

Passo 6: Configurar o Kubeflow
Você pode configurar o Kubeflow de acordo com suas necessidades específicas. O Kubeflow oferece uma variedade de componentes para diferentes etapas do ciclo de vida do Machine Learning, como Jupyter Notebooks, Pipelines, Katib (para hyperparameter tuning), e KFServing (para servir modelos).

Exemplo de Configuração de Pipeline
Aqui está um exemplo básico de como criar e executar um pipeline no Kubeflow:

6.1. Criar um Pipeline
Você pode criar um pipeline usando o SDK do Kubeflow Pipelines. Primeiro, instale o SDK:




pip install kfp
6.2. Exemplo de Código para Criar um Pipeline



import kfp
from kfp import dsl

def echo_op(text):
    return dsl.ContainerOp(
        name='echo',
        image='alpine:3.6',
        command=['echo', text],
    )

@dsl.pipeline(
    name='Echo Pipeline',
    description='A simple pipeline that echoes a message.'
)
def echo_pipeline(text='Hello, Kubeflow!'):
    echo_task = echo_op(text)

if __name__ == '__main__':
    kfp.compiler.Compiler().compile(echo_pipeline, 'echo_pipeline.yaml')
6.3. Subir o Pipeline para o Kubeflow



import kfp
client = kfp.Client()
client.create_run_from_pipeline_func(echo_pipeline, arguments={'text': 'Hello, Kubeflow!'})
Passo 7: Monitorar e Gerenciar Experimentos
Você pode monitorar e gerenciar seus experimentos e pipelines diretamente pelo dashboard do Kubeflow.

Seguindo esses passos, você conseguirá instalar e configurar o Kubeflow para suas necessidades de Machine Learning utilizando o Helm. Se precisar de mais alguma coisa, estou à disposição!

Configurações Adicionais
PostgreSQL
Você pode personalizar ainda mais a instalação do PostgreSQL usando um arquivo values.yaml:




postgresqlUsername: myUser
postgresqlPassword: myPassword
postgresqlDatabase: myDatabase
persistence:
  enabled: true
  size: 8Gi
E então instalar o PostgreSQL com o arquivo de valores:




helm install my-postgresql bitnami/postgresql -f values.yaml
MinIO
Da mesma forma, você pode personalizar a instalação do MinIO:




accessKey: minio
secretKey: minio123
persistence:
  enabled: true
  size: 10Gi
E então instalar o MinIO com o arquivo de valores:




helm install my-minio minio/minio -f values.yaml

Passo 3: Configurar o Grafana
Você pode personalizar a instalação do Grafana usando um arquivo values.yaml. Aqui está um exemplo de como você pode configurar o values.yaml para o Grafana:




adminUser: admin
adminPassword: admin
service:
  type: LoadBalancer
persistence:
  enabled: true
  size: 10Gi
Passo 4: Instalar o Grafana com o arquivo de valores
Use o arquivo values.yaml para instalar o Grafana:




helm install my-grafana grafana/grafana -f values.yaml
Exemplo Completo
Aqui está um exemplo completo de um arquivo values.yaml e o comando para instalar o Grafana:

values.yaml



adminUser: admin
adminPassword: admin
service:
  type: LoadBalancer
persistence:
  enabled: true
  size: 10Gi
Comando de instalação



helm install my-grafana grafana/grafana -f values.yaml

Acessar o Grafana
Para acessar o Grafana, você pode usar o kubectl port-forward ou acessar diretamente o serviço se estiver usando um LoadBalancer:




kubectl port-forward svc/my-grafana 3000:80
Agora você pode acessar o Grafana em http://localhost:3000 com as credenciais definidas no values.yaml.

Seguindo esses passos, você conseguirá instalar e configurar o Grafana usando o Helm. Se precisar de mais alguma coisa, estou à disposição!