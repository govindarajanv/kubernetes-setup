# kubernetes-setup

### Install Docker

 sudo apt-get remove docker docker-engine docker.io containerd runc  <br />
 sudo apt-get update -y <br />
 sudo apt-get install ca-certificates curl gnupg lsb-release -y <br />
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg  <br />
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null  <br />
 sudo apt-get update -y<br />
 sudo apt-get install docker-ce docker-ce-cli containerd.io -y <br />


sudo mkdir /etc/docker  <br />
cat <<EOF | sudo tee /etc/docker/daemon.json 
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF  <br />

sudo systemctl enable docker  <br />
sudo systemctl daemon-reload  <br />
sudo systemctl restart docker  <br />
 sudo docker run hello-world  <br />
 
sudo apt-get update  <br />
sudo apt-get install -y apt-transport-https ca-certificates curl  <br />
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg  <br />
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list  <br />
sudo apt-get update  <br />
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

## for Xenial use sudo apt-get install -y docker-ce
sudo add-apt-repository \
>    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
>    $(lsb_release -cs) \
>    stable"
sudo apt-get install docker-ce
sudo apt-get update
sudo apt-get install docker-ce
sudo apt-get update

# Suse

# become root

$ sudo -s

# install docker

$ zypper refresh
$ zypper install docker

# configure sysctl for Kubernetes

$ cat <<EOF >> /etc/sysctl.conf
net.ipv4.ip_forward=1
net.ipv4.conf.all.forwarding=1
net.bridge.bridge-nf-call-iptables=1
EOF

# add Google repository for installing Kubernetes packages
#$ zypper addrepo --type yum --gpgcheck-strict --refresh https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64 google-k8s

#or

$ cat <<EOF > /etc/zypp/repos.d/google-k8s.repo
[google-k8s]
name=google-k8s
enabled=1
autorefresh=1
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
type=rpm-md
gpgcheck=1
repo_gpgcheck=1
pkg_gpgcheck=1
EOF

# import Google repository keys

$ rpm --import https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
$ rpm --import https://packages.cloud.google.com/yum/doc/yum-key.gpg
$ rpm -q gpg-pubkey --qf '%{name}-%{version}-%{release} --> %{summary}\n'

# the following repository was needed only for GCP image
# other images was able successfully install conntrack-tools using existing repository

$ zypper addrepo https://download.opensuse.org/repositories/security:netfilter/SLE_12/security:netfilter.repo conntrack
$ zypper refresh conntrack

# conntrack presence is checked during kubeadm pre-flight checks 
# but zypper unable to find appropriate dependency for kubelet, 
# so let's install it manually

$ zypper install conntrack-tools

# refresh Google repository cache and check if we see several versions of Kubernetes packages to choose from

$ zypper refresh google-k8s
$ zypper packages --repo google-k8s

# install latest available kubelet package
# ignore conntrack dependency and install kubelet (Solution 2 in my case)

$ zypper install kubelet

# install kubeadm package. kubectl and cri-tools are installed as kubeadm dependency

$ zypper install kubeadm

# force docker to use systemd cgroup driver and overlay2 storage driver. 
# Check the links in the end of the answer for details. 
# BTW, kubelet would work even with default content of the file.

$ cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

# Not sure if it's necessary it was taken from the Kubernetes documentation

$ mkdir -p /etc/systemd/system/docker.service.d

# lets start and enable docker and kubelet services

$ systemctl start docker.service
$ systemctl enable docker.service
$ systemctl enable kubelet.service

# apply configured earlier sysctl settings. 
# net.bridge.bridge-nf-call-iptables becomes available after successfully starting
# Docker service 

$ sysctl -p

# Now it's time to initialize Kubernetes master node. 
# Ignore pre-flight checks for Vagrant box.

$ kubeadm init --pod-network-cidr=10.244.0.0/16

# prepare kubectl configuration to connect the cluster

$  mkdir -p $HOME/.kube
$  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$  sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Check if api-server responds to our requests. 
# At this moment it's fine to see master node in NotReady state.

$ kubectl get nodes

# Deploy Flannel network addon

$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# remove taint from the master node. 
# It allows master node to run application pods. 
# At least one worker node is required if this step is skipped.

$ kubectl taint nodes --all node-role.kubernetes.io/master-

# run test pod to check if everything works fine

$ kubectl run nginx1 --image=nginx

# after some time... ~ 3-5 minutes

# check the pods' state 

$ kubectl get pods -A -o wide
NAMESPACE     NAME                                READY   STATUS    RESTARTS   AGE     IP           NODE        NOMINATED NODE   READINESS GATES
default       nginx1                              1/1     Running   0          74s     10.244.0.4   suse-test   <none>           <none>
kube-system   coredns-66bff467f8-vc2x4            1/1     Running   0          2m26s   10.244.0.2   suse-test   <none>           <none>
kube-system   coredns-66bff467f8-w4jvq            1/1     Running   0          2m26s   10.244.0.3   suse-test   <none>           <none>
kube-system   etcd-suse-test                      1/1     Running   0          2m41s   10.4.0.4     suse-test   <none>           <none>
kube-system   kube-apiserver-suse-test            1/1     Running   0          2m41s   10.4.0.4     suse-test   <none>           <none>
kube-system   kube-controller-manager-suse-test   1/1     Running   0          2m41s   10.4.0.4     suse-test   <none>           <none>
kube-system   kube-flannel-ds-amd64-mbfxp         1/1     Running   0          2m12s   10.4.0.4     suse-test   <none>           <none>
kube-system   kube-proxy-cw5xm                    1/1     Running   0          2m26s   10.4.0.4     suse-test   <none>           <none>
kube-system   kube-scheduler-suse-test            1/1     Running   0          2m41s   10.4.0.4     suse-test   <none>           <none>

# check if the test pod is working fine

# curl 10.244.0.4
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...skipped...

# basic Kubernetes installation is don


# Fedora
# RedHat

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

## Set SELinux in permissive mode (effectively disabling it)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
