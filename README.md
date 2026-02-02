 # android-ssh-fullstack-server
Servidor PostgreSQL nativo e ambiente Ubuntu (PRoot) rodando em Android 12 via Termux, com suporte a VS Code Remote-SSH, otimização de login e Banco de Dados PostgreSQL.


Servidor reorganizado, lembrete de update aqui !


# 📱 Mobile-Server Architecture: A12 Ubuntu & PostgreSQL

## 📌 Sobre o Projeto ##
 documentando a transformação de um celular *Samsung Galaxy A12* (hardware móvel) em um ambiente de servidor Linux robusto. O objetivo era criar um laboratório de desenvolvimento, hospedando serviços de banco de dados e automações, integrado ao meu setup principal (Vision R15m no Ubuntu).

__________________________

## 🛠️ Especificações do Ecossistema ##
- *Servidor (Host):* Samsung Galaxy A12 (Octa-core / 4GB RAM)
- *S.O. Base:* Android 11+
- *Ambiente Linux:* Termux + PRoot (Ubuntu 25.10 LTS)
- *Estação de Trabalho:* Vision R15m (Ryzen 5 / Ubuntu)
- *Armazenamento Externo:* HDD 320GB (via Adaptador SATA/USB)

__________________________


    
## 🏗️ 1. Implementação da Infraestrutura ##

A base do servidor foi construída utilizando emulação de espaço de usuário (PRoot) para rodar uma distribuição Linux completa.
### Setup do Ambiente ###
1. *Instalação do Host:*
   ```bash
   pkg update && pkg upgrade
   pkg install proot-distro
   proot-distro install ubuntu

##2 . Banco de Dados (PostgreSQL)##
O banco de dados foi configurado nativamente no Termux para garantir a melhor performance de I/O, utilizando o armazenamento externo como volume principal.
```
pkg update && pkg upgrade -y
pkg install postgresql -y
mkdir -p $PREFIX/var/lib/postgresql
initdb $PREFIX/var/lib/postgresql
pg_ctl -D $PREFIX/var/lib/postgresql start
createuser --superuser --pwprompt seu_usuario
psql nome_banco_de_dados
```
Este banco é o meu repositório central de estudos. Nele, registo todos os exercícios de cursos e as tecnologias que estou a explorar, mantendo um histórico estruturado do meu progresso.

## forma de acesso: ##

-após a instalação, digite no terminal: psql postresql (ou o nome que você escolher a ele) e digite a senha.

**🗄️Gerenciamento de Dados**
 consulte o arquivo:
👉 [setup.sql](./setup.sql)

__________________________________

## 🛠️ Desafios Superados (Troubleshooting)##

* **Phantom Process Killer:** Estabilização do servidor via comandos ADB para impedir que o kernel do Android 12 encerre os binários do Postgres em background.
* **Missing Directory:** Correção do erro de inicialização do SSH através da criação manual do diretório `/run/sshd`.
* **Network Handshake:** Conflitos de porta entre o Host (Termux) e o Guest (Ubuntu PRoot).

__________________________________

## ⚠️ Comportamento de Sessão e Persistência ##

Durante o desenvolvimento, observei um comportamento específico na gestão de processos entre o Host (Termux) e o Guest (Ubuntu):

### 1. Persistência do Terminal
Ao utilizar o script `./start-server.sh`, a sessão do Ubuntu torna-se o processo principal. 
- **Ação:** O comando `exit` encerra o container PRoot.
- **Efeito:** Devido ao encadeamento de comandos, o encerramento do container pode fechar a sessão atual do Termux para garantir que não fiquem processos "zumbis" consumindo a RAM do Samsung A12.

### 2. Identificação de Ambiente via Neofetch ###
Embora o hardware reportado seja sempre o do dispositivo host (Kernel Android / Helio P35), a integridade do ambiente Ubuntu é confirmada pelo:
- **OS:** Ubuntu 25.10 (Oracular Oriole)
- **Package Manager:** Presença do `apt`.

___________________________________

***VScode***

Para viabilizar o uso do **VSCode Remote Server**, foi implementado um ambiente isolado (Ubuntu) para fornecer as bibliotecas `Glibc` e `libstdc++`, inexistentes no runtime nativo do Android (Bionic).

* **Configuração do SSH:**
  - Porta de escuta: 8023 (Bypass de restrição de portas baixas).
  - Inicialização: Caminho absoluto `/usr/sbin/sshd`.
  - Autenticação: Customização do `sshd_config` para desativar o módulo `PAM` e permitir `RootLogin`.
    ---
    Cuidado, configuração para uso pessoal e Servidores locais, para produção em nuvem é recomendado uso de chave SSH.
    ---
   
Erro ❌: Conflito entre os Servidores.

### conflito resolvido
-O S.O não permitia abrir o servidor Ubuntu, e ao abrir o servidor Termux e fazer o login direto, era fechado automaticamente, sendo necessário uma solução para abrir dentro do prórprio ubuntu
-usando proot-distro login ubuntu, para acessar o Sistema dentro do server Termux, usava-se os seguintes comandos: mkdir -p /run/sshd; /usr/sbin/sshd -p 8023 2>/dev/null; ssh root@IP -p 8023

**Dessa forma é manual e extensa.**

##**automatização:**##
-dentro do ubuntu (proot-distro login ubuntu), abri o servidor manualmemnte mkdir -p /run/sshd;    /usr/sbin/sshd -p 8023 2>/dev/null;    ssh root@IP -p 8023. 
-dentro do servidor, abri no cd ~ o nano ./bashrc , e adicionei nas informações do servidor as duas primeiras etapas, bastando apenas digitar (ssh root@ip -p 8023)

### novo modo de abertura ###
-entrar no server primario (termux) 
-abrir o ubuntu - proot-distro login ubuntu.
-dentro do ubuntu - ssh root@IP -p 8023 -> server aberto

### para entrar no VScode pelo servidor ###
-instalar a extenção Remote SSH
-adicionar o endereço IP (ssh root@IP -p 8023)
-coloque a senha do Servidor
-conectado


##Reetruturação do Sql dentro da nova camada



apt update && apt install postgresql postgresql-contrib -y  === para instalar no ubuntu via vscode

 chown postgres:postgres /run/postgresql ===  Porteiro que define o dono

  su - postgres === entrar no postgres

ls /usr/lib/postgresql/  === ver a versão do postgresql


mkdir -p /var/lib/postgresql/17/main   === criar um diretório para o postgresql

/usr/lib/postgresql/17/bin/initdb /var/lib/postgresql/17/main  === usado para direcionar os dados 


 /usr/lib/postgresql/17/bin/pg_ctl -D /var/lib/postgresql/17/main -l logfile start      ==== iniciar o Servidor Postgresql


Automação


nano ~/.bashrc  

cole no final

alias ligar='su - postgres -c "/usr/lib/postgresql/17/bin/pg_ctl -D /var/lib/postgresql/17/main -l logfile start"'

comando == digite (ligar)para iniciar o banco de dados
su - postgres == para entrar como dono
psql === entre no console do banco

source~/.bashrc === atualiza o terminal caso necessário
