# VirtualBox: Redes em Modo Bridge no Cluster

## Objetivo

Este tutorial explica como funciona o modo de rede `Bridged Adapter`
no VirtualBox e por que ele é útil para o cluster Kubernetes do
episódio 1.

## O que é uma VM no VirtualBox

Uma VM é uma máquina virtual executada sobre o sistema operacional
host. No laboratório, cada laptop hospeda VMs que representam nodes
do cluster, como `control plane` e `worker`.

Cada VM tem CPU, memória, disco e interfaces de rede virtuais. A
configuração de rede define como essa VM conversa com o host, com a
LAN e com outras máquinas.

## O que é o modo Bridge

No modo `bridge`, a placa de rede virtual da VM é conectada à rede
física do host como se fosse uma máquina independente na mesma LAN.

Na prática:

- a VM aparece na rede com seu próprio IP
- o roteador pode entregar IP via DHCP diretamente para a VM
- outras máquinas da rede conseguem acessar a VM sem NAT intermediário

## Como o tráfego funciona

O VirtualBox cria uma ponte entre a interface de rede do host e a
interface virtual da VM. Assim, os pacotes da VM entram e saem pela
rede local como se viessem de um equipamento separado.

Exemplo:

- `Laptop 1` está na rede `192.168.0.0/24`
- a VM `cp-01` recebe `192.168.0.101`
- a VM `worker-01` recebe `192.168.0.102`

Outros laptops e VMs da mesma rede podem alcançar esses endereços
diretamente.

## Diferença para NAT

No modo `NAT`, a VM sai para a rede usando o IP do host e uma camada
de tradução. Isso funciona bem para acesso à internet, mas dificulta
conexões iniciadas de fora para dentro da VM.

Para um cluster, isso costuma ser um problema porque:

- nodes precisam se comunicar entre si diretamente
- ferramentas de administração precisam alcançar os nodes
- serviços internos dependem de conectividade previsível

## Por que Bridge é útil no cluster

No laboratório do episódio 1, o objetivo é distribuir nodes entre
três laptops e simular um ambiente mais próximo de produção. O modo
`bridge` ajuda porque:

- cada node ganha identidade própria na rede
- control planes e workers podem trocar tráfego sem gambiarras de NAT
- SSH, `ping`, `kubectl` e instalação do cluster ficam mais simples
- a topologia fica mais próxima de um ambiente real

## Aplicabilidade no Kubernetes

Um cluster Kubernetes exige comunicação estável entre nodes e entre
componentes do control plane. Em um cenário com VMs em bridge:

- cada node pode ter IP próprio e previsível
- o bootstrap do cluster fica mais direto
- integrações com DNS local, roteador e reserva de DHCP ficam viáveis
- o troubleshooting de rede fica mais fácil

Isso é especialmente importante quando há múltiplos `control planes`
mantendo quorum e workers distribuídos entre máquinas diferentes.

## Cuidados e Limitações

O modo `bridge` depende da rede física disponível. Alguns pontos
práticos:

- redes Wi-Fi podem ter limitações dependendo do adaptador e driver
- mudanças de rede podem alterar IPs se o DHCP for dinâmico
- regras do roteador podem afetar descoberta e conectividade
- o ideal é usar IPs reservados ou DHCP estável para os nodes

## Quando usar NAT e quando usar Bridge

Use `NAT` quando a VM só precisa sair para a internet e não precisa
ser acessada diretamente.

Use `bridge` quando a VM precisa se comportar como um host real da
rede local, como acontece com nodes de um cluster Kubernetes.

## Recomendação para o laboratório

Para este projeto, o mais coerente é configurar os nodes do cluster
em `bridge`, garantindo que cada VM possa:

- receber IP próprio na LAN
- ser acessada por SSH a partir de outros laptops
- se comunicar diretamente com control planes e workers
- participar do cluster com menos complexidade de rede

## Resumo

- `VM`: máquina virtual executada no host
- `Bridge`: VM entra na LAN como máquina independente
- `NAT`: VM sai para a rede usando tradução pelo host
- `Bridge` é mais indicado para nodes do cluster
- conectividade direta simplifica instalação e operação do Kubernetes
