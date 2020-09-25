Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-18.04"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2408"
    vb.cpus = "2"
  end 

  config.vm.synced_folder "/Personal/", "/mnt/personal"

  config.vm.network "public_network"

	config.vm.network "forwarded_port", guest: 4040, host: 4040
	config.vm.network "forwarded_port", guest: 9090, host: 9090
	config.vm.network "forwarded_port", guest: 8080, host: 8080

  # Setting up Kubernetes
  
  config.vm.provision "shell", inline: <<-SHELL
  sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  sudo add-apt-repository \
     "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) \
     stable"
  sudo apt-get install docker-ce docker-ce-cli containerd.io

  # Verification
  docker --version
  # Version 19.03
  systemctl status docker
  # Ensure that the status is "active (running)"
  # You may have to press q to quit

  systemctl stop docker
  systemctl start docker
  systemctl restart docker

  # Install Kubernetes
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
  sudo apt-get update
  sudo apt-get install -y kubelet kubeadm kubectl
  sudo apt-mark hold kubelet kubeadm kubectl
  SHELL
end