<h1> Cluster Swarm local, utilizando máquinas virtuais (Arch linux e Vagrant)</h1>
🔸 <strong>1.</strong> Abra o terminal e atualize repositorios e sitema operacional (arch) :

```
sudo pacman -Syyuu 
```

🔸 <strong>2.</strong> Para instalar Vagrant  acesse o site https://www.vagrantup.com/ </em>

```
sudo pacman -S vagrant
```

🔸 <strong>3.</strong> Criar o arquivo vagrant :

```
vagrant init
```

🔸<strong>4.</strong> Basta editar aquivo com seu editor de texto preferido:
```
# -*- mode: ruby -*-
# vi: set ft=ruby  :

machines = {
  "master" => {"memory" => "1024", "cpu" => "1", "ip" => "100", "image" => "bento/ubuntu-22.04"},
  "node01" => {"memory" => "1024", "cpu" => "1", "ip" => "101", "image" => "bento/ubuntu-22.04"},
  "node02" => {"memory" => "1024", "cpu" => "1", "ip" => "102", "image" => "bento/ubuntu-22.04"},
  "node03" => {"memory" => "1024", "cpu" => "1", "ip" => "103", "image" => "bento/ubuntu-22.04"}
}

Vagrant.configure("2") do |config|

  machines.each do |name, conf|
    config.vm.define "#{name}" do |machine|
      machine.vm.box = "#{conf["image"]}"
      machine.vm.hostname = "#{name}"
      machine.vm.network "private_network", ip: "10.10.10.#{conf["ip"]}"
      machine.vm.provider "virtualbox" do |vb|
        vb.name = "#{name}"
        vb.memory = conf["memory"]
        vb.cpus = conf["cpu"]
        
      end
      machine.vm.provision "shell", path: "docker.sh"
      
      if "#{name}" == "master"
        machine.vm.provision "shell", path: "master.sh"
      else
        machine.vm.provision "shell", path: "worker.sh"
      end

    end
  end
end

```

🔸​<strong>4.1</strong> Criar script Docker (docker.sh)  :

```
#!/bin/bash
curl -fsSL https://get.docker.com | sudo bash
sudo curl -fsSL "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo usermod -aG docker vagrant

```

​🔸<strong>5</strong> Criar script docker swarm   (master.sh)

```
#!/bin/bash
sudo docker swarm init --advertise-addr=10.10.10.100
sudo docker swarm join-token worker | grep docker > /vagrant/worker.sh
```
​🔸<strong>6</strong> criar arquivo para armazenar comando para transformar as demais maquinas virtuais em trabalhadoras (worker.sh)

```
touch worker.sh
```
🔸​<strong>7</strong> Executar escript vagrant

```
vagrant up 
```
