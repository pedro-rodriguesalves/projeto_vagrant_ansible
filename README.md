# Projeto DevOps com Vagrant e Ansible

**AUTORIDADES MÁXIMAS:** Pedro Henrique Rodrigues Alves - (20241380026) e Felipe da Silva Oliveira - (20241380003).

**DISCIPLINA:** Administração de Sistemas Abertos.

**PROFESSOR, ORIENTADOR E AUTORIDADE MÁXIMA:** Leonidas Francisco de Lima Júnior.

## Introdução

<p align="justify">
Este projeto tem como objetivo automatizar o provisionamento e a configuração de uma infraestrutura composta por quatro máquinas virtuais Linux (Debian), utilizando Vagrant e Ansible. O ambiente simula um cenário real de DevOps, integrando serviços como SSH, NFS, LVM, DHCP, Apache, MariaDB e um servidor DNS para resolução interna de nomes. Toda a documentação, bem como o Vagrantfile e os playbooks Ansible, está organizada neste repositório para facilitar reprodução e manutenção do ambiente.
</p>

## Vagrantfile

O projeto inclui um Vagrantfile que define a criação de quatro máquinas virtuais:

- Servidor de Arquivos (arq): hostname arq.pedro.felipe.devops, IP estático 192.168.56.126, 512 MB de RAM, três discos adicionais de 10 GB cada.

- Servidor de Banco de Dados (db): hostname db.pedro.felipe.devops, IP via DHCP, MAC: "0800271A0026", 512 MB de RAM.

- Servidor de Aplicação (app): hostname app.pedro.felipe.devops, IP via DHCP, MAC: "0800272B2600", 512 MB de RAM.

- Cliente (cli): hostname cli.pedro.felipe.devops, IP via DHCP, 1024 MB de RAM.
<p align="justify">  
Todas as máquinas usam a box debian/bookworm64, com linked_clone habilitado, guest additions desabilitado e sem geração de novas chaves SSH.
</p>

## Playbooks 

Os playbooks foram organizados de acordo com as responsabilidades de cada máquina:

- **`geral.yml`**  
  Aplica configuração comum a todas as VMs:  
  - Atualização do sistema.
  - Instalação e configuração do Chrony (NTP).  
  - Definição do timezone para America/Recife.
  - Criação do grupo `ifpb`.
  - Criação dos usuários `pedro` e `felipe`.  
  - Criação de diretórios .ssh e geração/configuração de chaves SSH.
  - Configuração de banner e regras de segurança do SSH.
  - Bloqueio de root e desativação de autenticação por senha.
  - Instalação do cliente NFS.
  - Permissão de sudo para o grupo `ifpb`.

- **`arq-dns-dhcp.yml`**  
  Configura o servidor DNS e DHCP na VM `arq`:  
  - Instalação do servidor DHCP.
  - Configuração do arquivo /etc/dhcp/dhcpd.conf com faixa `192.168.56.50–192.168.56.100`.
  - Criação de reservas para as VMs db e app por MAC address.
  - Definição da interface eth1 para o DHCP.
  - Reinicialização do serviço DHCP.
  - Instalação do servidor DNS BIND9.
  - Cópia das opções do DNS `(named.conf.options)`.
  - Cópia da configuração de zonas `(named.conf.internal-zones)`.
  - Cópia da zona direta do domínio `(pedro.felipe.devops.db)`.
  - Cópia da zona reversa da rede `(56.168.192.db)`.
  - Habilitação e inicialização do BIND9.

- **`arq-lvm-nfs.yml`**  
  Configura LVM e NFS na VM `arq`:  
  - Instalação dos pacotes LVM2 e parted.
  - Criação de partições em três discos.
  - Criação dos volumes físicos (PVs).
  - Criação do Volume Group `dados`.
  - Criação do Logical Volume `ifpb` com 15 GB.
  - Formatação do LV em `ext4`.
  - Montagem automática em `/dados`.
  - Instalação do serviço `nfs-kernel-server`.
  - Criação do usuário `nfs-ifpb` sem shell.
  - Criação do diretório compartilhado `/dados/nfs`.
  - Exportação do diretório para a rede `192.168.56.0/24`.
  - Uso de all_squash mapeando todos os acessos para o usuário nfs-ifpb.
  - Forçamento de gravação imediata no disco (sync).
  - Permissão de escrita apenas para o usuário nfs-ifpb.

- **`dhcpd.conf`**  
  Configura DHCP na VM `arq`:  
  - Definição do tempo padrão e máximo de lease.
  - Marcação como servidor DHCP autoritativo.
  - Configuração do domain-name.
  - Configuração do domain-name-servers apontando para `192.168.56.126`.
  - Definição da sub-rede `192.168.56.0/24`.
  - Configuração do range de IPs dinâmicos.
  - Definição do gateway padrão.
  - Criação da reserva estática do host `db`.
  - Criação da reserva estática do host `app`.
  - Criação da reserva estática do host `arq`.

- **`named.conf.options`**  
  Configura DNS na VM `arq`:  
  - Definição do diretório de cache do BIND.
  - Ativação de recursion.
  - Permissão de consultas para qualquer origem.
  - Permissão de recursion para a rede interna e localhost.
  - Configuração de forwarders (8.8.8.8 e 1.1.1.1).
  - Ativação do DNSSEC-validation automático.
  - Configuração de listen-on (IPv4).

- **`named.conf.internal-zones`**  
  Configura DNS na VM `arq`: 
  - Definição do arquivo da zona direta (pedro.felipe.devops.db).
  - Definição do arquivo da zona reversa (56.168.192.db).

- **`pedro.felipe.devops.db`**  
  Configura DNS na VM `arq`:  
  - Definição do TTL padrão da zona.
  - Configuração do SOA.
  - Configuração do servidor NS autoritativo.
  - Registro A para arq `(192.168.56.126)`.
  - Registro A para db `(192.168.56.103)`.
  - Registro A para app `(192.168.56.163)`.

- **`56.168.192.db`**  
  Configura DNS na VM `arq`:  
  - Definição do TTL padrão da zona.
  - Configuração do SOA.
  - Configuração do servidor NS autoritativo.
  - Registro PTR para `arq`.
  - Registro PTR para `db`.
  - Registro PTR para `app`.

- **`db.yml`**  
  Configura a VM `db`:  
  - Instalação do servidor MariaDB.
  - Inicialização e habilitação do serviço MariaDB.
  - Instalação dos pacotes autofs e nfs-common.
  - Criação do diretório local de montagem NFS.
  - Configuração do autofs no arquivo /etc/auto.master.
  - Criação do arquivo /etc/auto.nfs para montagem automática do share.
  - Configuração do autofs para montar o diretório via NFS.
  - Inicialização e habilitação do serviço autofs.
  - Geração de handler para reiniciar o autofs quando necessário.

- **`app.yml`**  
  Configura a VM `app`:  
  - Instalação do servidor web `Apache2`.
  - Inicialização e habilitação do serviço apache2.
  - Criação e envio do arquivo index.html personalizado.
  - Reinício automático do Apache após atualização do arquivo `(pedro.felipe.html)`.
  - Instalação do autofs e do cliente NFS.
  - Criação do diretório local de montagem `/var/nfs`.
  - Configuração do arquivo `/etc/auto.master` para incluir o ponto de montagem NFS.
  - Criação do arquivo `/etc/auto.nfs` com parâmetros de montagem para o compartilhamento NFS do servidor arq.
  - Reinício automático do serviço autofs.

- **`cli.yml`**  
  Configura a VM `cli`:  
  - Instalação dos pacotes `firefox-esr`, `xauth` e `x11-utils`.
  - Configuração do SSH para permitir encaminhamento X11.
  - Ajuste das diretivas X11DisplayOffset e X11UseLocalhost.
  - Instalação do autofs e do cliente NFS.
  - Criação do diretório local de montagem `/var/nfs`.
  - Configuração do arquivo `/etc/auto.master` para incluir o ponto de montagem NFS.
  - Criação do arquivo `/etc/auto.nfs` com os parâmetros de montagem do compartilhamento NFS.
  - Inicialização e habilitação do serviço autofs.


## Execução do Projeto:

### **Pré-requisitos**
- **VirtualBox** Instalado.  
- **Vagrant** Instalado.  
- **Ansible** Instalado no Host.  
  

### **Passos Necessários para a execução:**

1. **Clone este repositório:**
   ```bash
   git clone https://github.com/pedro-rodriguesalves/projeto_vagrant_ansible.git
   CD projeto_vagrant_ansible
   
2. **Suba as VMs:**
   ```bash
   vagrant up 
