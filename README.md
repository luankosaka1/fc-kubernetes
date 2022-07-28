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

## Instalando Kind

Instale Kind e Kubectl

https://kind.sigs.k8s.io/docs/user/quick-start/#creating-a-cluster

## Criando primeiro cluster com Kind

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

## Criando cluster multi node

Crie o arquivo kind.yaml na pasta k8s

```
kind  create cluster --config=k8s/kind.yaml
```

https://kind.sigs.k8s.io/docs/user/configuration/#nodes

## Mudança de context oe extensão do VSCod

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