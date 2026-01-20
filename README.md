# android-ssh-fullstack-server
Servidor PostgreSQL nativo e ambiente Ubuntu (PRoot) rodando em Android 12 via Termux, com suporte a VS Code Remote-SSH e otimização de Phantom Processes.

# 📱 Mobile-Server Architecture: A12 Ubuntu & PostgreSQL

## 📌 Sobre o Projeto
Este repositório documenta a transformação de um dispositivo *Samsung Galaxy A12* (hardware móvel subutilizado) em um ambiente de servidor Linux robusto. O objetivo é criar um laboratório de desenvolvimento, hospedando serviços de banco de dados e automações, perfeitamente integrado ao meu setup principal (Vision R15m no Ubuntu).


## 🛠️ Especificações do Ecossistema
* *Servidor (Host):* Samsung Galaxy A12 (Octa-core / 4GB RAM)
* *S.O. Base:* Android 11+
* *Ambiente Linux:* Termux + PRoot (Ubuntu 22.04 LTS)
* *Estação de Trabalho:* Vision R15m (Ryzen 5 / Ubuntu)
* *Armazenamento Externo:* HDD 320GB (via Adaptador SATA/USB)

---

## 🏗️ 1. Implementação da Infraestrutura

A base do servidor foi construída utilizando emulação de espaço de usuário (PRoot) para rodar uma distribuição Linux completa sem necessidade de acesso Root.

### Setup do Ambiente
1. *Instalação do Host:*
   ```bash
   pkg update && pkg upgrade
   pkg install proot-distro
   proot-distro install ubuntu

   ### 2. Camada de Compatibilidade (Glibc & VS Code)
Para viabilizar o uso do **VS Code Remote Server**, foi implementado um ambiente isolado (Ubuntu) para fornecer as bibliotecas `Glibc` e `libstdc++`, inexistentes no runtime nativo do Android (Bionic).

* **Configuração do SSH:**
  - Porta de escuta: `8023` (Bypass de restrição de portas baixas).
  - Inicialização: Caminho absoluto `/usr/sbin/sshd`.
  - Autenticação: Customização do `sshd_config` para desativar o módulo `PAM` e permitir `RootLogin`.

### 3. Persistência de Dados (PostgreSQL)
O banco de dados foi configurado nativamente no Termux para garantir a melhor performance de I/O, utilizando o armazenamento externo como volume principal.

---

## 🛠️ Desafios Superados (Troubleshooting)

* **Phantom Process Killer:** Estabilização do servidor via comandos ADB para impedir que o kernel do Android 12 encerre os binários do Postgres em background.
* **Missing Directory:** Correção do erro de inicialização do SSH através da criação manual do diretório `/run/sshd`.
* **Network Handshake:** Resolução de conflitos de porta entre o Host (Termux) e o Guest (Ubuntu PRoot).



## ⚠️ Comportamento de Sessão e Persistência

Durante o desenvolvimento, observei um comportamento específico na gestão de processos entre o Host (Termux) e o Guest (Ubuntu):

### 1. Persistência do Terminal
Ao utilizar o script `./start-server.sh`, a sessão do Ubuntu torna-se o processo principal. 
- **Ação:** O comando `exit` encerra o container PRoot.
- **Efeito:** Devido ao encadeamento de comandos, o encerramento do container pode fechar a sessão atual do Termux para garantir que não fiquem processos "zumbis" consumindo a RAM do Samsung A12.

### 2. Identificação de Ambiente via Neofetch
Embora o hardware reportado seja sempre o do dispositivo host (Kernel Android / Helio P35), a integridade do ambiente Ubuntu é confirmada pelo:
- **OS:** Ubuntu 25.10 (Oracular Oriole)
- **Package Manager:** Presença do `apt`.
