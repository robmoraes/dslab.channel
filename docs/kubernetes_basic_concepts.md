# Conceitos Básicos de Kubernetes

## Objetivo

Esta página serve como preparação para o episódio 1 da temporada
Home Datacenter. A ideia é alinhar os conceitos mínimos antes de
provisionar os nodes do laboratório.

## O que é Kubernetes

Kubernetes é uma plataforma de orquestração de containers. Ele
coordena onde aplicações rodam, como são distribuídas entre máquinas
e como continuam disponíveis mesmo com falhas.

## Cluster

Um cluster Kubernetes é o conjunto de máquinas que executa a
plataforma. Essas máquinas podem ser físicas ou virtuais. No contexto
do episódio, o cluster será montado sobre VMs distribuídas entre
três laptops.

## Node

Node é uma máquina do cluster. Cada node oferece CPU, memória,
rede e armazenamento para executar workloads.

Os dois tipos principais são:

- `Control Plane`: mantém o estado do cluster e toma decisões.
- `Worker`: executa as aplicações empacotadas em containers.

## Control Plane

O control plane é a parte de gerenciamento do Kubernetes. Ele decide
em qual node um workload será executado, acompanha o estado desejado
do cluster e reage a falhas.

No episódio 1, a proposta é usar 3 nodes de control plane consolidar 
um quorum mínimo para a alta disponibilidade.

## Worker Node

Worker nodes são responsáveis por rodar os Pods. Eles recebem as
instruções do control plane e executam as cargas de trabalho do
cluster.

## Pod

Pod é a menor unidade de execução do Kubernetes. Normalmente ele
contém um ou mais containers que precisam compartilhar rede e ciclo
de vida.

Na prática, quando uma aplicação roda no Kubernetes, ela roda dentro
de um Pod.

## Deployments e Estado Desejado

Kubernetes trabalha com estado desejado. Em vez de subir containers
manualmente, você declara o que quer, por exemplo:

- 3 réplicas de uma aplicação
- imagem de container específica
- portas e variáveis de ambiente

O cluster tenta continuamente manter esse estado.

## Service

Service é a forma de expor Pods de maneira estável na rede. Como Pods
podem ser recriados e mudar de IP, o Service fornece um ponto fixo de
acesso dentro ou fora do cluster.

## Alta Disponibilidade

Alta disponibilidade, ou HA, significa manter o ambiente operando
mesmo quando uma máquina falha. No contexto deste laboratório:

- distribuir control planes entre laptops reduz pontos únicos de falha
- múltiplos workers permitem continuar executando workloads
- quorum entre control planes ajuda a preservar a saúde do cluster

## Relação com o Episódio 1

No episódio 1, o foco não é ainda subir aplicações complexas, mas
preparar a base correta:

- criar os nodes
- separar papéis entre control plane e worker
- distribuir recursos entre os laptops
- garantir base para um cluster com HA

## Resumo Rápido

- `Cluster`: conjunto de máquinas do Kubernetes
- `Node`: cada máquina do cluster
- `Control Plane`: camada de gerenciamento
- `Worker`: máquina que executa workloads
- `Pod`: menor unidade de execução
- `Service`: forma estável de acesso na rede
- `HA`: capacidade de continuar operando diante de falhas
