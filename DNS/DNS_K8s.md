# DNS Interno no K8s
A utilização de DNS interno é altamente recomendada para a gestão de IPs de Nodes, Pods, e outros recursos no Kubernetes. O Kubernetes já possui um serviço de DNS interno integrado, geralmente implementado com o `CoreDNS`, que facilita a resolução de nomes de domínio dentro do cluster. Aqui estão algumas razões para utilizar DNS interno no Kubernetes:

### Benefícios do DNS Interno no Kubernetes
* Resolução de Nomes Automática:

O Kubernetes automaticamente cria registros DNS para serviços e pods, permitindo que você se refira a eles por nome em vez de IP, o que simplifica a comunicação entre componentes.

* Descoberta de Serviços:

Facilita a descoberta de serviços dentro do cluster. Por exemplo, um serviço chamado my-service no namespace default pode ser acessado via `my-service.default.svc.cluster.local`.

* Escalabilidade:

Permite que os serviços escalem horizontalmente sem a necessidade de atualizar manualmente as configurações de rede, pois os registros DNS são atualizados automaticamente.

* Isolamento e Segurança:

Mantém a resolução de nomes dentro do cluster, reduzindo a exposição a ameaças externas e melhorando a segurança.

* Gestão Centralizada:

Facilita a gestão centralizada de nomes de domínio e endereços IP dentro do cluster, permitindo uma administração mais eficiente e organizada.

### Exemplo de Configuração de DNS Interno no Kubernetes

O `CoreDNS` é geralmente configurado por padrão em clusters Kubernetes. Aqui está um exemplo básico de como você pode verificar e configurar o `CoreDNS`:

#### Verificar o Deployment do CoreDNS:
```bash
kubectl get deployments -n kube-system
```

#### Configurar o CoreDNS:
O arquivo de configuração do `CoreDNS` geralmente está localizado em um ConfigMap chamado coredns no namespace kube-system.
```bash
kubectl edit configmap coredns -n kube-system
```

Um exemplo de configuração do `CoreDNS` pode ser:
``` 
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
```

#### Aplicar as Configurações:

Após editar o `ConfigMap`, o `CoreDNS` automaticamente recarrega as configurações.

#### Testar a Resolução de Nomes
Você pode testar a resolução de nomes dentro do cluster usando um pod de teste:

```bash
kubectl run -i --tty --rm debug --image=busybox --restart=Never -- sh
```
Dentro do pod de teste, você pode usar o comando `nslookup` ou `dig` para verificar a resolução de nomes:

```bash
nslookup my-service.default.svc.cluster.local
```

## Conclusão
Utilizar DNS interno no Kubernetes é uma prática recomendada que simplifica a comunicação entre componentes, melhora a segurança e facilita a gestão de serviços e pods dentro do cluster. O `CoreDNS`, que é a solução DNS padrão no Kubernetes, oferece uma configuração flexível e poderosa para atender a essas necessidades.



