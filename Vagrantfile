# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.ssh.insert_key = false
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.provider :virtualbox do |v|
    v.memory = 512
    v.linked_clone = true
    v.check_guest_additions = false
  end
  config.trigger.before :"Vagrant::Action::Builtin::WaitForCommunicator", type: :action do |t|
    t.warn = "Iterrompe o servidor dhcp do virtualbox"
    t.run = {inline: "vboxmanage dhcpserver stop --interface vboxnet0"}
  end

  # Servidor de Arquivos (arq):
  config.vm.define "arq" do |arq|
    arq.vm.hostname = "arq.pedro.felipe.devops"
    arq.vm.network :private_network, ip: "192.168.56.126"
    (0..2).each do |x|
      arq.vm.disk :disk, size: "10GB", name: "disk-#{x}"
    end
  end

  # Servidor de Banco de Dados (db):
  config.vm.define "db" do |db|
    db.vm.hostname = "db.pedro.felipe.devops"
    db.vm.network :private_network, mac: "0800271A0026", type: "dhcp"
  end

  # Servidor de Aplicação (app):
  config.vm.define "app" do |app|
    app.vm.hostname = "app.pedro.felipe.devops"
    app.vm.network :private_network, mac: "0800272B2600", type: "dhcp"
  end

  # Host Cliente (cli):
  config.vm.define "cli" do |cli|
    cli.vm.hostname = "cli.pedro.felipe.devops"
    cli.vm.network :private_network, type: "dhcp"
    cli.vm.provider :virtualbox do |v|
      v.memory = 1024
    end
  end
end
