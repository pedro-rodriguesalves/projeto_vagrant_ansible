# -*- mode: ruby -*-
# vi: set ft=ruby :

# Configuração do Vagrant para criar máquinas virtuais Debian com Ansible
Vagrant.configure("2") do |config|
  # Box base utilizada para todas as VMs
  config.vm.box = "debian/bookworm64"
  # Impede que o Vagrant gere novas chaves SSH
  config.ssh.insert_key = false

  # Configuração do plugin vagrant-vbguest (adiciona suporte a guest additions)
  if Vagrant.has_plugin?("vagrant-vbguest")
    # Desativa atualização automática do guest additions
    config.vbguest.auto_update = false
  end

  # Desativa a pasta sincronizada padrão
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Configurações específicas do provedor VirtualBox
  config.vm.provider :virtualbox do |v|
    v.memory = 512           # Memória RAM padrão para todas as VMs
    v.linked_clone = true    # Ativa linked clone para economizar espaço
    v.check_guest_additions = false  # Desabilita verificação de guest additions
  end

  # Trigger para parar o servidor DHCP interno do VirtualBox antes de provisionar
  config.trigger.before :"Vagrant::Action::Builtin::WaitForCommunicator", type: :action do |t|
    t.warn = "Iterrompe o servidor dhcp do virtualbox"
    t.run = {inline: "vboxmanage dhcpserver stop --interface vboxnet0"}
  end

  # Provisionamento global com Ansible
  config.vm.provision "ansible" do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "playbooks/geral.yml" # Playbook para configuração comum
    ansible.groups = {
      "all" => ["arq", "db", "app", "cli"] # Aplica a todas as VMs
    }
  end

  # ===============================================================
  # Servidor de Arquivos (arq)
  # ===============================================================
  config.vm.define "arq" do |arq|
    arq.vm.hostname = "arq.pedro.felipe.devops" # Hostname da VM
    arq.vm.network :private_network, ip: "192.168.56.126" # IP estático

    # Criação de três discos adicionais de 10GB cada
    (0..2).each do |x|
      arq.vm.disk :disk, size: "10GB", name: "disk-#{x}"
    end

    # Provisionamento Ansible para DHCP e DNS
    arq.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/arq-dns-dhcp.yml"
    end

    # Provisionamento Ansible para LVM e NFS
    arq.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/arq-lvm-nfs.yml"
    end
  end

  # ===============================================================
  # Servidor de Banco de Dados (db)
  # ===============================================================
  config.vm.define "db" do |db|
    db.vm.hostname = "db.pedro.felipe.devops" # Hostname
    db.vm.network :private_network, mac: "0800271A0026", type: "dhcp" # IP via DHCP com reserva por MAC

    # Provisionamento Ansible para configuração do banco
    db.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/db.yml"
    end
  end

  # ===============================================================
  # Servidor de Aplicação (app)
  # ===============================================================
  config.vm.define "app" do |app|
    app.vm.hostname = "app.pedro.felipe.devops" # Hostname
    app.vm.network :private_network, mac: "0800272B2600", type: "dhcp" # IP via DHCP com reserva por MAC

    # Provisionamento Ansible para configuração do Apache e NFS
    app.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/app.yml"
    end
  end

  # ===============================================================
  # Host Cliente (cli)
  # ===============================================================
  config.vm.define "cli" do |cli|
    cli.vm.hostname = "cli.pedro.felipe.devops" # Hostname
    cli.vm.network :private_network, type: "dhcp" # IP via DHCP

    # Memória maior para o cliente
    cli.vm.provider :virtualbox do |v|
      v.memory = 1024
    end

    # Provisionamento Ansible para instalar pacotes cliente e montar NFS
    cli.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/cli.yml"
    end
  end
end

