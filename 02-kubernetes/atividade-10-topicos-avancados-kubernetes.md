# Atividade 10 — Tópicos avançados em Kubernetes

## 1. Arquitetura do cluster do curso

O cluster usado no curso é provisionado com **Vagrant** (VirtualBox), a partir do repositório `fbscarel/contorq-files` (pasta `s2`).

| Item | Onde é definido | Valor |
|---|---|---|
| Nº de VMs e faixa de IP | início do `Vagrantfile` | 1 master (`.19`) + 1 node (`.24`) em `192.168.68.x` |
| Imagem das VMs | `config.vm.box` | `bento/debian-11` |
| Recursos | `vb.memory` / `vb.cpus` | master: 4096 MB, 1 CPU; node: 2048 MB, 1 CPU |
| Scripts de laboratório | `file provisioner` | pasta `./scripts` copiada para `~/scripts` |
| Instalação do cluster | `shell provisioner` | `setup.sh`, com `master` ou `node` no `$1` |
| Versões | topo do `setup.sh` | containerd 1.5.10, Docker 20.10.13, Kubernetes 1.23.5 |
| Método de instalação | `setup.sh` | `kubeadm init` com `--pod-network-cidr=10.32.0.0/12` |

O `kubeadm init` usa o IP da interface `eth1` (`--apiserver-advertise-address`), e o pod network CIDR coincide com o `IPALLOC_RANGE` do Weave Net.

## 2. Gerenciamento gráfico do cluster

Opções populares: **Rancher**, **Red Hat OpenShift**, **Portainer** e **Kubernetes Dashboard**.

A mais interessante é o **Rancher**: open source, multi-cluster, integração com autenticação corporativa (LDAP/AD/SSO), RBAC por interface e monitoramento integrado. O OpenShift é muito adotado no meio corporativo, porém mais opinativo e geralmente com custo de licenciamento.

Vantagens sobre a configuração apenas por linha de comando:

- facilidade na configuração de autenticação complexa e RBAC;
- provisionamento de clusters e adição de nodes facilitados;
- monitoramento e observabilidade integrados;
- integração com pipelines de CI/CD e plataformas de logging/SIEM;
- menor barreira de entrada e menos erro operacional para equipes diversas, com visibilidade centralizada.

A CLI segue necessária para automação e GitOps; a interface complementa, não substitui.

## 3. Ferramentas de segurança (hardening)

Soluções populares: Falco, Open Policy Agent, The Update Framework, in-toto, Anchore, Checkov, Snyk, Aqua Security e Orca Security.

Mais fáceis de adotar (open source, sem aquisição):

- **Falco:** detecção em runtime (shell em container, acesso a arquivos sensíveis, escalada de privilégio).
- **OPA/Gatekeeper:** políticas no admission (bloquear containers privilegiados, exigir limits, permitir só registries confiáveis).
- **Checkov:** análise estática de manifestos, Helm e IaC, executável direto no pipeline de CI/CD antes do deploy.

Ganho: prevenção (políticas e varredura na esteira, *shift-left*) e detecção (visibilidade em runtime), reduzindo a superfície de ataque e facilitando auditoria e conformidade. Fonte para acompanhar novidades: CNCF Landscape (categoria *Security & Compliance*).

## 4. Entrega

Documento discursivo (`.docx`) com as respostas das seções 2 e 3 enviado ao AVA.
