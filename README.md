# kubernetes-setup

### Install Docker

 sudo apt-get remove docker docker-engine docker.io containerd runc  <br />
 sudo apt-get update -y <br />
 sudo apt-get install ca-certificates curl gnupg lsb-release -y <br />
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg  <br />
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null  <br />
 sudo apt-get update -y<br />
 sudo apt-get install docker-ce docker-ce-cli containerd.io -y <br />
sudo swapoff -a
sudo sysctl net.bridge.bridge-nf-call-iptables=1

sudo mkdir /etc/docker  <br />
```
cat<<EOF | sudo tee /etc/docker/daemon.json 
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```
<br />

sudo systemctl enable docker  <br />
sudo systemctl daemon-reload  <br />
sudo systemctl restart docker  <br />
 sudo docker run hello-world  <br />
 
sudo apt-get update -y <br />
sudo apt-get install -y apt-transport-https ca-certificates curl  <br />
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg  <br />
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list  <br />
sudo apt-get update -y <br />
sudo apt-get install -y kubelet kubeadm kubectl  <br />
sudo apt-mark hold kubelet kubeadm kubectl  <br />




## on master node
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors all<br />
mkdir -p $HOME/.kube <br />
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config <br />
sudo chown $(id -u):$(id -g) $HOME/.kube/config <br />

## joining the worker nodes
Use kubeadm to join the worker nodes to the master node (the command is provided after kubeadm init is run on the master): <br />

sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash <br />

## install flannel network add on all nodes
Enable net.bridge.bridge-nf-call-iptables on all nodes to ensure that the the iptables proxy functions correctly. <br />
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf <br />
sudo sysctl -p <br />
# Install flannel on the master:
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
