# S01 Home Datacenter > E01 Building the Lab

Nesse episódio vamos planejar um cluster Kubernetes em uma
infraestrutura doméstica.

Serão usados 3 laptops, e neles serão distribuídas VMs com
Oracle VirtualBox para provisionamento dos servidores que serão
os nodes do cluster.

## Sobre o Laboratório

Este laboratório tem como objetivo simular um pequeno datacenter
real utilizando hardware doméstico.

A ideia é aprender como sistemas distribuídos e infraestrutura
moderna funcionam na prática, construindo tudo do zero:

- Virtualização
- Kubernetes
- Redes
- Storage
- Observabilidade
- Automação
- Segurança
- Cloud

## Premissas

Conhecimento prévio dos conceitos de Kubernetes, Cluster, HA e Containers.

Material de apoio em
[docs/kubernetes_basic_concepts.md](../../../docs/kubernetes_basic_concepts.md).

## Objetivos da Season

Ao final desta season teremos:

- 5 VMs com Ubuntu Server instaladas
- Rede configurada com IP fixo e DNS local
- Acesso SSH entre os nodes
- Cluster Kubernetes inicializado
- 3 control-planes com quorum
- 2 workers
- Um workload nginx rodando
- Teste de alta disponibilidade desligando nodes control-plane

## Arquitetura do Cluster

O cluster Kubernetes será composto por:

- 3 nodes control-plane
- 2 nodes worker
- etcd rodando em modo stacked
- Rede local 192.168.1.0/24
- DNS local via /etc/hosts
- Virtualização com VirtualBox
- Ubuntu Server LTS

## Os Laptops

| Laptop   | Memória | CPUs | Storage | VMs                       |
| -------- | ------: | ---: | ------: | ------------------------- |
| Laptop 1 |    16Gi |   12 |    450G | 1 control-plane, 1 worker |
| Laptop 2 |     8Gi |    8 |    250G | 1 control-plane           |
| Laptop 3 |    16Gi |    8 |      1T | 1 control-plane, 1 worker |

O `Laptop 1` será usado como workstation, a base de operações
onde os videos serão gravados e os nodes serão acessados por
SSH. O SO é Ubuntu

## Os Nodes

Vamos provisionar os nodes com recursos mínimos para instalação do
servidor Linux e configuração do Kubernetes.

Será usado Ubuntu Server em sua última versão LTS.

**Provisionamento inicial**

| VM              | MEM | CPU | STORAGE |
| --------------- | --: | --: | ------- |
| control-plane-1 | 2Gi |   2 | 50G     |
| control-plane-2 | 2Gi |   2 | 50G     |
| control-plane-3 | 2Gi |   2 | 50G     |
| worker-1        | 2Gi |   2 | 50G     |
| worker-2        | 2Gi |   2 | 50G     |

À medida que forem sendo adicionados workloads, vamos
monitorando o consumo e aumentando os recursos se necessário até
o limite possível.

### IPs e DNS

A rede das VMs será configurada para operar em modo Bridge,
para que cada node tenha um IP simulando uma placa de rede real.

[Material de apoio sobre rede em modo Bridge](../../../docs/virtualbox_bridge_networking.md)

A rede bridge será configurada usando o Wi-Fi do roteador doméstico,
pois não há cabo de rede para todos os laptops.

Sempre que possível é mais indicado uso de conexão cabeada.

O range de IP usado para o cluster depende de disponibilade. Nessa rede doméstica usei os comandos
desse doc: [comandos para descobrir a configuração de rede](../../../docs/discovery_network_configs.md) -
para descobrir minha configuração de rede.

Escolhi os seguintes IPs e DNSs para os nodes:

| Tipo de Node  | IP              | DNS            |
| ------------- | --------------- | -------------- |
| control-plane | 192.168.1.50/24 | cp1.kube.dslab |
| control-plane | 192.168.1.51/24 | cp2.kube.dslab |
| control-plane | 192.168.1.52/24 | cp3.kube.dslab |
| worker        | 192.168.1.60/24 | w1.kube.dslab  |
| worker        | 192.168.1.61/24 | w2.kube.dslab  |

Como estaremos confinados em uma rede local, para usar DNS nos nodes,
os name servers serão configurados nos arquivos `/etc/hosts` de cada node.

```bash
# nano /etc/hosts
192.168.1.50    cp1.kube.dslab
192.168.1.51    cp1.kube.dslab
192.168.1.52    cp2.kube.dslab
192.168.1.60    w1.kube.dslab
192.168.1.61    w2.kube.dslab
```

### Credenciais de acesso aos nodes

As credenciais serão sempre as mesmas, configuradas na instalação do Ubuntu Server:

    user: dslab
    pass: dslab123
