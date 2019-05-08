## On NGINX LB
```
yum update
yum install -y docker
mkdir /etc/nginx
vim /etc/nginx/nginx.conf
```
Add this to conf
```
stream {
    upstream stream_backend {
        # REPLACE WITH master0 IP
        server 10.160.0.9:6443;
        # REPLACE WITH master1 IP
        server 10.160.0.10:6443;
        # REPLACE WITH master2 IP
        server 10.160.0.11:6443;
    }

    server {
        listen        6443;
        proxy_pass    stream_backend;
        proxy_timeout 3s;
        proxy_connect_timeout 1s;
    }

}
```
Start the docker container with ngnix, copying the conf we added above to the container and exposing the 6443 ports
```
docker run --name proxy \
    -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
    -p 6443:6443 \
    -d nginx
```
Do these on all masters and node
```
Add Docker Repo

cat <<EOF > /etc/yum.repos.d/centos.repo
[centos]
name=CentOS-7
baseurl=http://ftp.heanet.ie/pub/centos/7/os/x86_64/
enabled=1
gpgcheck=1
gpgkey=http://ftp.heanet.ie/pub/centos/7/os/x86_64/RPM-GPG-KEY-CentOS-7
#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
baseurl=http://ftp.heanet.ie/pub/centos/7/extras/x86_64/
enabled=1
gpgcheck=0
EOF
```
Install and enable start docker
```
yum -y update 
yum -y install docker
systemctl enable docker
systemctl start docker
```
Add Kubernetes repo
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
Disable SELINUX
```
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
```
Install Kubeadm, Kubectl, Kubelet and enable them
```
yum -y install kubelet kubeadm kubectl
systemctl enable kubelet
systemctl start kubelet
```
Check connectivity
```
kubeadm config images pull --kubernetes-version=1.14.0
```
Disable Swap
```
swapoff -a
```
Network configs and disabling firewalld for conflicts between iptable rules
```
bash -c 'cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF'
echo 1 > /proc/sys/net/ipv4/ip_forward

sysctl --system

systemctl disable firewalld
systemctl stop firewalld
```
Do the above mentioned tasks on all master nodes and worker nodes.

SSH into the master-0 and initiate the kubeadm init with the following config

```
mkdir /etc/kubernetes/kubeadm
vim /etc/kubernetes/kubeadm/kubeadm-config.yaml
```
```
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: stable
#REPLACE with `loadbalancer` IP
controlPlaneEndpoint: "LBIP:6443"
networking:
  podSubnet: 192.168.0.0/18
```
```
kubeadm init \
    --config=/etc/kubernetes/kubeadm/kubeadm-config.yaml \
    --experimental-upload-certs
```

```
You can now join any number of the control-plane node running the following command on each as root:

kubeadm join 10.160.0.8:6443 --token nmiqmn.yls76lcyxg2wt36c \
--discovery-token-ca-cert-hash sha256:5efac16c86e5f2ed6b20c6dbcbf3a9daa5bf75aa604097dbf49fdc3d1fd5ff7d \
--experimental-control-plane --certificate-key 828fc83b950fca2c3bda129bcd0a4ffcd202cfb1a30b36abb901de1a3626a9df
```

Run these to get the kubectl access

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Examine the `kubeadm-cert` secret in the `kube-system` namespace.
```
kubectl get secrets -n kube-system kubeadm-certs -o yaml
```
```
kubeadm token list
```
Install `calico` CNI-plugin with a pod CIDR matching the `podSubnet` configured above.

```
curl https://docs.projectcalico.org/v3.7/manifests/calico.yaml -O
POD_CIDR="<your-pod-cidr>" \
sed -i -e "s?192.168.0.0/16?$POD_CIDR?g" calico.yaml
kubectl apply -f calico.yaml
```
Verify 1 node is  `Ready`.

```
kubectl get nodes
```
Verify  `kube-system`  pods are  `Running`.

```
kubectl get pods -n kube-system
```
SSH to the  `master1`  host.
    
Run the recorded join command from the previous section.

    kubeadm join 192.168.122.170:6443 --token nmiqmn.yls76lcyxg2wt36c \
    --discovery-token-ca-cert-hash sha256:5efac16c86e5f2ed6b20c6dbcbf3a9daa5bf75aa604097dbf49fdc3d1fd5ff7d \
    --experimental-control-plane \
    --certificate-key 828fc83b950fca2c3bda129bcd0a4ffcd202cfb1a30b36abb901de1a3626a9df
After completion, verify there are now 2 nodes.

```
kubectl get nodes
```
```
NAME       STATUS   ROLES    AGE   VERSION
master-0   Ready    master   12h   v1.14.1
master-1   Ready    master   12h   v1.14.1
```
Add the Join commend on the other worker node and the join command of the worker node on worker node
```
NAME       STATUS   ROLES    AGE   VERSION
master-0   Ready    master   12h   v1.14.1
master-1   Ready    master   12h   v1.14.1
master-2   Ready    master   11h   v1.14.1
worker-0   Ready    <none>   11h   v1.14.1
```

