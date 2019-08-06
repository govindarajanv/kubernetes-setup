# kubernetes-setup

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - <br />
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" <br />
sudo apt-get update <br />
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu <br />
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - <br />
sudo bash -c 'cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list  <br/>
deb https://apt.kubernetes.io/ kubernetes-xenial main  <br/>
EOF' <br/>
sudo apt-get update <br />
sudo apt-get install -y kubelet=1.12.7-00 kubeadm=1.12.7-00 kubectl=1.12.7-00 <br />
sudo apt-mark hold kubelet kubeadm kubectl <br />

# on master node
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 <br />
mkdir -p $HOME/.kube <br />
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config <br />
sudo chown $(id -u):$(id -g) $HOME/.kube/config <br />

# joining the worker nodes
Use kubeadm to join the worker nodes to the master node (the command is provided after kubeadm init is run on the master): <br />

sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash <br />

# install flannel network add on all nodes
Enable net.bridge.bridge-nf-call-iptables on all nodes to ensure that the the iptables proxy functions correctly. <br />
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf <br />
sudo sysctl -p <br />
# Install flannel on the master:
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
