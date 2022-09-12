# Kubernetes

## Introdução ao Kubernetes

É um produto Open Source utilizado para automatizar a implantação, o dimensionamento e o gerenciamento de aplicativos em contêiner.

### Da onde veio?

Google

1. Borg
2. Omega
3. Kubernetes

### Pontos Importantes

- Kubernetes é disponibilizado através de um conjunto de APIs
- Normalmente acessamos a API usando a CLI: kubectl
- Tudo é baseado em estado. Você configura o estado de cada objeto
- Kubernetes Master
-- Kube-apiserver
-- Kube-controller-manager
-- Kube-scheduler
- Outros Nodes:
-- Kubelet
-- Kubeproxy

### Dinâmica "superficial"

Cluster: Conjunto de máquinas (Nodes)
Pods: Unidade que contém os containers provisionados
O Pod representa os processos rodando no cluster

### Deployment

Tem o objetivo de provisionar os Pods

ReplicaSets: quem define quando provisionamento serão disponibilizados.

B = Backend => 3 réplicas
F = Frontend => 2 réplicas

----------------
-- Deployment --
----------------
----B--B--B-----
----F--F--------
----------------

### Instalando Kind

Instale Kind e Kubectl

https://kind.sigs.k8s.io/docs/user/quick-start/#creating-a-cluster

### Criando primeiro cluster com Kind

Criando cluster

```
kind create cluster
```

Rodando o Kubernetes

```
kubectl cluster-info --context kind-kind
```

Exibindo os Nodes

```
kubectl get nodes
```

Deletando cluster

```
kind delete clusters kind
```

### Criando cluster multi node

Crie o arquivo kind.yaml na pasta k8s

```
kind  create cluster --config=k8s/kind.yaml
```

https://kind.sigs.k8s.io/docs/user/configuration/#nodes

### Mudança de context oe extensão do VSCod

Listando os clusters configurados

```
kubectl config get-clusters
```

Troca o contexto (cluster)

```
kubectl config use-context {kind-fullcycle}
```

Instalar a extensão do VSCod

https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools

## Primeiros passos na prática

### Criando aplicação exemplo e imagem

Criar imagem

```
docker build -t lkosaka/hello-go .
```

Testando a imagem

```
docker run --rm -p 80:80 lkosaka/hello-go
```

Subir a imagem no docker hub

```
dockedr push lkosaka/hello-go
```

### Trabalhando com Pods

Criar arquivo k8s/pod.yaml

Criando o primeiro Pod

```
kubectl apply -f k8s/pod.yaml 
```

Excluir o Pod

```
kubectl delete pod goserver
```

### Criando primeira ReplicaSet

Criar arquivo k8s/replicaset.yaml

```
kubectl apply -f k8s/replicaset.yaml
```

### O problema do ReplicaSet

Quando vc sobe o ReplicaSet com docker X e depois altera para docker Y, a alteracão só ocorre quando mata o pod.

```
kubectl describe pod {nome do pod}
```

### Implementando Deployment

O Deployment é responsável por matar a ReplicaSet para subir uma nova.

```
kubectl delete replicaset goserver
kubectl get pods
```

```
kubectl apply -f k8s/deployment.yaml
kubectl get deployments
kubectl get replicasets
```

### Rollout e Revisões

Como voltar a versão anterior da replicaset?

```
kubectl rollout history deployment goserver
```

Volta para última versão

```
kubectl rollout undo deployment goserver
```

Escolhendo a versão que queria retornar

```
kubectl rollout undo deployment goserver --to-revision={numero da revisao}
```

## Services

### Entendendo o conceito de services

Quando temos as maquinas rodando, não sigifica que ele esteja disponível para acessar.

O services é a porta de entrada para aplicação, responsável por disponibilizar a aplicação para acessar externamente.

### Utilizando ClusterIP

Responsável por definir a porta do cluster

```
kubectl apply -f k8s/service.yaml
kubectl get service
```

Vinculando a porta local com porta do cluster

```
kubectl port-forward svc/goserver-service 8000:80
```

### Diferenças entre Port e targetPort

Port: a porta do service
TargetPort: a porta do container (porta da aplicação)


### Utilizando proxy para acessar API do Kubernetes

Acessando a API do Kubernetes

```
kubectl proxy --port=8080
```

### Utilizando NodePort

Gera uma porta e libera a porta de todos os nodes do cluster.
Geralmente utilizado para demonstração

### Trabalhando com LoadBalancer

Utilizando quando tem um cluster gerenciado
Gera automaticamente IP externo

```
kubectl apply -f k8s/loadbalance.yaml
```

Quando executar kubectl get svc, o campo EXTERNAL-IP será preenchido automaticamente quando utilizado na AWS/DigitalOcean...

## Objetos de configuração

Trabalhando com variáveis de ambiente

Atualizar a imagem

```
docker build -t lkosaka/hello-go:v1 .
docker push lkosaka/hello-go:v1
```

Adicionar os variaveis de ambiente

```
kubectl apply -f k8s/deployment.yaml
```

Liberar a porta para testar local

```
kubectl port-forward svc/goserver-service 8000:80
```

### Variáveis de ambiente com ConfigMap

Crie o arquivo k8s/configmap-env.yaml

Quando alterar o arquivo configmap-env.yaml é necessário regerar o deployment.yaml (arquivo que utilizar as configurações do configmap-env.yaml)

### Injetando ConfigMap na aplicação

Acessando o pod

```
kubectl exec -it goserver-bxzrt -- bash
```

Acessando o log do pod

```
kubectl logs {pod}
```

Quando o POD fica com o status pending e náo starta é recomendado utilziar o k3d

Instalando k3d

```
$ wget -q -O - https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
```

### Secrets e variáveis de ambiente

Tem como objetivo não deixar visível as varáveis de ambiente, porém é possível descriptografar (quando usa base 64)

```
echo "luan" | base64
echo $USER # valor da variável de ambiente
```

## Probes

### Entendendo health check

Precisamos tratar os pods que apresentarem problemas e garantir que os usuários não acessem mais o pod com problema

### Criando endpoint Healthz

Criado novo endpoint no server.go e versão v7 do lkosaka/hello-go

### Liveness na prática

o liveness reincia o processo quando der X erros (failureThreshold)

Existem três tipos: 
- Http (faz requisição)
- Command (executa comando no container)
- TCP (estabelece conexão)

Consulta frequentement os pods

```
watch -n1 kubectl get pods
```

### Entendendo readiness

O readiness tem como objetivo definir quando a aplicação esta pronto para tráfego e quando der erro irá interromper a aplicação.


Veja as informações (historico) dos status de um determinado pod

```
kubectl describe pod {nome do pod}
```

### Combinando Liveness e Readiness

Devemos tomar cuidado com as configurações para que um não impacte no outro e acabe deixando a aplicação fora!!!

### Trabalhando com startupProbe

A partir da versão 1.16 do kubernet foi adicionado startupProbe.

## Resources e HPA (Horizontal Pod Autoscaling)

### Instalando metrics-server

Coletor de métricas em tempo real

https://github.com/kubernetes-sigs/metrics-server

```
cd k8s
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
cp components.yaml metrics-server.yaml
```

Adicionar --kubelet-insecure-tls na linha 139

### Entendendo utilização de Resources

Definir os recursos que iremos utilizar

vCPU = 1000m (milicores)

### Aplicando deployment com resources

Verificando o consumo do POD

```
kubectl apply -f k8s/deployment-resources.yaml 
kubectl top pod goserver-6557db858c-thns2
```

### Criando e configurando HPA (Horizontal Pod Autoscaling)

Responsável por verificar as métricas e vai provisionar as réplicas

Não utiliza apenas o CPU para escalonar