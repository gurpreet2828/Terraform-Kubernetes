
# Kubernetes Cluster Deployment on AWS using Terraform

This guide walks through deploying a Kubernetes cluster on AWS using Terraform and kubeadm, followed by deploying a React application on the cluster.

---

## Step 1: Transfer Files from Windows to Linux Machine

Use SCP to copy your Terraform and Kubernetes configuration files from your Windows machine to the Ubuntu Linux instance:

```bash
scp -r "C:\Users\Gurpreet\OneDrive\Desktop\York Univ\Assignments\Assignment-7-Kubernetes\Terraform-Kubernetes" administrator@<ubuntu-ip>:/home/administrator/
```

Set the correct ownership and permissions on the Linux machine:

```bash
sudo chown -R administrator:administrator /home/administrator/Terraform-Kubernetes
sudo chmod -R u+rwx /home/administrator/Terraform-Kubernetes
```

---

## Step 2: Install Terraform and AWS CLI on Ubuntu

Update your system and install dependencies:

```bash
sudo apt update && sudo apt install -y gnupg software-properties-common curl unzip
```

Add HashiCorp's GPG key and repository to install Terraform:

```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install terraform -y
terraform -v
```

Install AWS CLI:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

---

## Step 3: Configure AWS CLI

Run AWS CLI configuration and enter your AWS Access Key, Secret Key, default region, and output format:

```bash
aws configure
```

Verify credentials:

```bash
aws sts get-caller-identity
```

---

## Step 4: Provision AWS Infrastructure Using Terraform

Navigate to your Terraform directory:

```bash
cd /home/administrator/Terraform-Kubernetes
```

Initialize Terraform:

```bash
terraform init
```

Format Terraform files for consistency:

```bash
terraform fmt
```

Validate Terraform configuration:

```bash
terraform validate
```

Review infrastructure changes Terraform will apply:

```bash
terraform plan
```

Apply Terraform configuration to create resources:

```bash
terraform apply
```

---

## Step 5: Connect to Kubernetes Master Node

Use SSH to access the master node using the public IP output from Terraform:

```bash
ssh -i /path/to/key.pem ec2-user@<master-node-public-ip>
```

---

## Step 6: Setup Kubernetes Master Node

Disable swap (required by Kubernetes):

```bash
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
```

Update system and install dependencies:

```bash
sudo yum update -y
sudo yum install -y curl wget git
```

Load kernel modules for Kubernetes networking:

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

Configure sysctl parameters:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

Install containerd:

```bash
sudo yum install -y containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

Add Kubernetes repository and install kubeadm, kubelet, kubectl:

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
```

Set SELinux to permissive mode:

```bash
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

---

## Step 7: Initialize Kubernetes Cluster and Install Calico

Create kubeadm config file:

```bash
cat <<EOF > kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: 1.32.0
networking:
  podSubnet: 192.168.0.0/16
apiServer:
  extraArgs:
   service-node-port-range: 1024-1233
EOF
```

Initialize Kubernetes cluster:

```bash
sudo kubeadm init --config kubeadm-config.yaml --ignore-preflight-errors=all
```

Configure kubectl for the user:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Install Calico network plugin:

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Check node status:

```bash
kubectl get nodes
```

---

## Step 8: Setup Kubernetes Worker Node

SSH to worker node:

```bash
ssh -i /path/to/key.pem ec2-user@<worker-node-public-ip>
```

Repeat setup steps from Step 6 to install containerd, Kubernetes components, and disable swap.

---

## Step 9: Join Worker Node to Cluster

On master node, generate join command:

```bash
kubeadm token create --print-join-command
```

Run the output command on the worker node to join it to the cluster.

Verify nodes on master:

```bash
kubectl get nodes
```

---

## Step 10: Deploy React Application

Create `react-app-pod.yml` with this content (replace `<your-docker-hub-image>`):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: react-app
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 1233
  selector:
    app: react-app
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: react-app
  template:
    metadata:
      labels:
        app: react-app
    spec:
      containers:
      - name: react-app
        image: <your-docker-hub-image>
        ports:
        - containerPort: 80
```

Apply the deployment:

```bash
kubectl apply -f react-app-pod.yml
kubectl get pods
kubectl get svc
```

Access the app via:

```
http://<master-node-public-ip>:1233
```

or

```
http://<worker-node-public-ip>:1233
```

---

## Step 11: Cleanup Resources

Destroy all infrastructure created by Terraform:

```bash
terraform destroy
```

---

# End of Guide
