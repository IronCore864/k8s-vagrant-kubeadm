BOX_IMAGE = "hashicorp/bionic64"
NODE_COUNT = 2
MASTER_IP = "192.168.10.10"
NODE_IP_PREFIX = "192.168.10."
POD_NETWORK_CIDR = "10.244.0.0/16"
#Generate new using steps in README
KUBETOKEN = "b029ee.968a33e8d8e6bb0d"

$masterscript = <<MASTERSCRIPT
kubeadm reset -f
kubeadm init --apiserver-advertise-address=#{MASTER_IP} --pod-network-cidr=#{POD_NETWORK_CIDR} --token #{KUBETOKEN} --token-ttl 0
mkdir -p $HOME/.kube
sudo cp -Rf /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
MASTERSCRIPT

$nodescript = <<NODESCRIPT
kubeadm reset -f
kubeadm join --token #{KUBETOKEN} #{MASTER_IP}:6443
NODESCRIPT

Vagrant.configure(2) do |config|
    config.vm.box = BOX_IMAGE
    config.vm.provider "virtualbox" do |vb|
        vb.cpus = 1
        vb.memory = "1024"
    end
    config.vm.provision :shell, :path => "install.sh"

    config.vm.define "master" do |subconfig|
        subconfig.vm.hostname = "master"
        subconfig.vm.network "private_network", ip: MASTER_IP
        subconfig.vm.provider :virtualbox do |vb|
            vb.memory = 2048
            vb.cpus = 2
        end
        subconfig.vm.provision :shell, inline: $masterscript
    end
    
    (1..NODE_COUNT).each do |i|
        config.vm.define "node0#{i}" do |subconfig|
            subconfig.vm.hostname = "node0#{i}"
            subconfig.vm.network "private_network", ip: NODE_IP_PREFIX + "#{i + 10}"
            subconfig.vm.provision :shell, inline: $nodescript
        end
    end
end
