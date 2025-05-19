
# **Deploying Kubernetes Cluster on AWS**

## **Summary**

This report outlines the deployment of a Kubernetes cluster on AWS using Terraform and Kubernetes tools, followed by the deployment of a sample React application. The infrastructure is provisioned with VPC, subnets,security groups, and EC2 instances designated as master and worker nodes. The cluster is initialized using kubeadm, and Calico is configured as the network plugin. The React application is deployed as a Kubernetes service using NodePort for external access. The provided troubleshooting section addresses common issues such as SSH connectivity, Kubernetes node joins, and service accessibility.

## **Main steps of the report, summarized:**

1. **Transfer Files to Linux Machine**: Use SCP to transfer Terraform and Docker files from a Windows machine to an Ubuntu instance and set the correct permissions.

2. **Install Terraform and AWS CLI**: Install the necessary dependencies, HashiCorp GPG key, and Terraform. Install and configure the AWS CLI with the required access keys.

3. **AWS Infrastructure**: Use Terraform to initialize, validate, plan, and apply the infrastructure configurations. This includes creating EC2 instances for master and worker nodes.

4. **Connect to Kubernetes Master Node**: SSH into the EC2 instance using the public IP address assigned during Terraform deployment.

5. **Set Up Kubernetes Master Node**: Install necessary dependencies (e.g., containerd, kubeadm, kubelet) and configure Kubernetes on the master node, including disabling swap and configuring networking settings.

6. **Initialize Kubernetes Cluster**: Use kubeadm init to initialize the cluster and set up Kubernetes CLI access.

7. **Install Calico Network Plugin**: Apply the Calico plugin for networking and check the status of the nodes.

8. **Connect to Kubernetes Worker Node**: SSH into the worker node and install the required Kubernetes components (containerd, kubeadm, kubelet, kubectl).

9. **Join Worker Node to Cluster**: Use the kubeadm join command from the master node to add the worker node to the Kubernetes cluster.

10. **Deploy React Application**: Create and apply a YAML file for the React app, then verify the pod and service status. Access the app via the public IP of the master or worker node.

11. **Clean Up Resources**: Destroy the infrastructure using terraform destroy after the deployment is complete.

## **Step 1: Transfer Files from Windows to Linux Machine**

Use `scp` to transfer your Terraform and Docker files from your local machine to your Ubuntu instance.  
**Note:** Run the following command in Command Prompt (CMD) to copy the Terraform and Kubernetes code to your Linux machine:

```shell
scp -r -v "C:\Users\Gurpreet\OneDrive\Desktop\York Univ\Assignments\Assignment-7-Kubernetes\Terraform-Kubernetes" administrator@10.0.0.83:/home/administrator/
```

![Image1](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/9bc9affe2c6baf0846cd729b516f49e255c59c1e/Images/Image1.png)

After entering the password, you will be logged into your Ubuntu Linux machine and will see the files in your home directory as shown below

![Image2](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/9bc9affe2c6baf0846cd729b516f49e255c59c1e/Images/Image2.png)

**Note:** To avoid permission issues, please run the following commands to ensure the appropriate permissions are set:

```bash
sudo chown -R administrator:administrator /home/administrator/Terraform-Kubernetes

sudo chmod -R u+rwx /home/administrator/Terraform-Kubernetes
```

These commands will assign ownership to the administrator user and grant the necessary read, write, and execute permissions for the Terraform-Kubernetes directory.

## **Step 2: Install Terraform and AWS Command Line Interface (CLI)**

### **1. Update and install dependencies**

```shell
sudo apt update && sudo apt install -y gnupg software-properties-common curl
```

### **2. Add the HashiCorp GPG key**

```shell
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

### **3. Add the HashiCorp repo**

```shell
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
```

```shell
sudo tee /etc/apt/sources.list.d/hashicorp.list
```

### **4. Update and install Terraform**

```shell
sudo apt update
```

```shell
sudo apt install terraform -y
```

### **5. Verify installation**

```shell
terraform -v
```

![Image3](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/32e431dae7f7ac294d82c533e71d836a17b8b7c9/Images/Image3.png)

## AWSCLI Install

To install the AWS CLI, run the following command

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Run the following command to check if AWS CLI is installed correctly:

```shell
aws –version
```

You see the following output

![Image4](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/9bc9affe2c6baf0846cd729b516f49e255c59c1e/Images/Image4.png)

## **Create AWS account**

After Creating

Click on account name - Select Security Credentials

![Image50](https://github.com/gurpreet2828/jenkins-cicd-terraform-aws/blob/51d4f14a4001b6679c4c4977191cf7a04ea76768/Images/Image50.png)

Click **Create access key**.

![Image51](https://github.com/gurpreet2828/Jenkins-CICD/blob/47b28cca86aff817a0d18ae3a7d99cb69b7591f3/Images/Image51.png)

**Note:** Download the key file or copy the Access Key ID & Secret Access Key (Secret Key is shown only once!).

After install and creating AWS account configure the AWS

Configure AWS CLI with the New Access Key

```shell
aws configure
```

It will prompt you for:

**1. AWS Access Key ID**: Your access key from AWS IAM.

**2. AWS Secret Access Key**: Your secret key from AWS IAM.

**3. Default region name**: (e.g., us-east-1, us-west-2).

**4. Default output format**: (json, table, text --- default is json).

***Enter access key and secret key which you will get from aws account***

**Check credentials added to aws configure correctly**:

```shell
aws sts get-caller-identity
```

If your AWS CLI is properly configured, you\'ll see a response like this:

![Image9](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/9bc9affe2c6baf0846cd729b516f49e255c59c1e/Images/Image9.png)

## **Step 4: Provisioning AWS Infrastructure using Terraform**

**1.** `Terraform init`

- prepares your environment and configures everything Terraform needs to interact with your infrastructure.

![Image10](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/ccb2abe80634611295238d8963ef24d7be384e10/Images/Image10.png)

**2.** `terraform fmt`

- used to **automatically format** your Terraform configuration files to a standard style. It ensures that your code is consistently formatted, making it easier to read and maintain.

**3.** `Terraform validate`

- used to **check the syntax and validity** of your Terraform configuration files. It helps you catch errors in the configuration
  before you attempt to run other Terraform commands, like terraform plan or terraform apply.

![Image11](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/ccb2abe80634611295238d8963ef24d7be384e10/Images/Image11.png)

**4.** `terraform plan`

- used to **preview the changes** Terraform will make to your infrastructure based on the current configuration and the existing state. It shows what actions will be taken (such as creating, modifying, or deleting resources) when you apply the configuration

- Before running terraform apply to check exactly what changes Terraform will make.

> ***Before Running Terraform Plan must update the location of public and private ssh keys under modules -compute - variables.tf***
>
> ***As shown in following image***

![Image12](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/a3e907b060d5c68931297d338630ad10c392b774/Images/Image12.png)

**After applying the Terraform plan, you will see the following output:**

![Image13](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/ccb2abe80634611295238d8963ef24d7be384e10/Images/Image13.png)

**5.** `Terraform apply`

Provision terraform managed infrastructure. You must confirm by trying **yes** if you would like to continue and perform the actions described to provision your infrastructure resources

After successfully applying the Terraform configuration, you will see the public IP addresses assigned to your Kubernetes master and node instances as output.

![Image14](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/ccb2abe80634611295238d8963ef24d7be384e10/Images/Image14.png)

**k8s-master-Public-IP**: The public IP address assigned to the Kubernetes master node.

**k8s-node-Public-IP**: A list of public IP addresses assigned to the Kubernetes worker nodes.

**You can log in to your AWS account to view the infrastructure resources that have been provisioned.**

![Image15](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/ccb2abe80634611295238d8963ef24d7be384e10/Images/Image15.png)

## **Step 5: Connect to K8s Master (Control Plane) Node**

Using the public IP address provided in the Terraform output, connect to the EC2 instance by executing the following command in your terminal:

```shell
ssh -i /root/.ssh/docker ec2-user@34.201.56.12
```

![Image16](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/9ee57c26065324975be7567bbab76783d6b7b4ef/Images/Image16.png)

## **Step 6: Install and Configure Kubernetes Master (Control Plane) Node**

### **Follow these steps to set up the Kubernetes Control Plane node effectively:**

**Update the System and Install Dependencies**

Run the following commands to update the system and install essential packages:

```bash
sudo yum update -y

sudo yum install -y curl wget git
```

**Disable Swap**:

Kubernetes requires swap to be disabled. Execute:

```bash
sudo swapoff -a
sudo sed -i \'/ swap / s/\^\\.\*\\\$/#\1/g\' /etc/fstab
```

**Load Modules for containerd:**

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

**Set Up sysctl Parameters for Kubernetes Networking**:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

```shell
sudo sysctl --system
```

![Image17](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/9ee57c26065324975be7567bbab76783d6b7b4ef/Images/Image17.png)

**CRI-Tools**:

cri-tools is a set of command-line utilities for working with container runtimes that implement the Container Runtime Interface (CRI) in Kubernetes.

**Download and Install the Latest cri-tools RPM:**

```shell
cd \~
```

```shell
curl -LO
https://download.opensuse.org/repositories/isv:/kubernetes:/core:/stable:/v1.30/rpm/x86_64/cri-tools-1.30.0-150500.1.1.x86_64.rpm
```

```shell
sudo yum localinstall -y cri-tools-1.30.0-150500.1.1.x86_64.rpm
```

```shell
sudo sysctl --system
```

### **Install containerd**

**Update the system:**

```shell
sudo yum update -y
```

**Install containerd:**

```shell
sudo yum install -y containerd
```

![Image18](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/9ee57c26065324975be7567bbab76783d6b7b4ef/Images/Image18.png)

**Create the configuration directory:**

```shell
sudo mkdir -p /etc/containerd
```

**Generate containerd configuration:**

```shell
containerd config default | sudo tee /etc/containerd/config.toml
```

![Image19](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/main/Images/Image19.png)

**Restart containerd:**

```shell
sudo systemctl restart containerd
```

**Verify containerd status:**

```shell
sudo systemctl status containerd
```

![Image20](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/32e431dae7f7ac294d82c533e71d836a17b8b7c9/Images/Image20.png)

### **Install Kubernetes Components (kubeadm, kubelet, kubectl)**

#### **Add Kubernetes repository:**

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
```

![Image21](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image21.png)

#### **Set SELinux in Permissive Mode:**

```bash
sudo setenforce 0
```

```bash
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

![Image22](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image22.png)

#### **Install kubeadm, kubelet, and kubectl**

```shell
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

![Image23](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image23.png)

#### **Enable and Start kubelet**

```bash
sudo systemctl enable --now kubelet
```

![Image24](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image24.png)

## **Step 7: Initialize the Kubernetes Cluster and Install Calico Network**

### 1. **Create Kubeadm Config File:**

```bash
vi kube-config.yml
```

#### **Insert the following content in above yml file**

```yml
apiVersion: kubeadm.k8s.io/v1beta3
kubernetesVersion: 1.32.0
# This is a configuration file for kubeadm to set up a Kubernetes cluster.
kind: ClusterConfiguration
networking:
  podSubnet: 192.168.0.0/16
apiServer:
  extraArgs:
   service-node-port-range: 1024-1233
```

### 2. **Initialize Kubernetes Cluster:**

```bash
sudo kubeadm init --config kube-config.yml --ignore-preflight-errors=all
```

**Note:** The purpose of --ignore-preflight-errors=all flag is to ignore the K8s HW requirements

![Image25](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image25.png)

### 3. **Set Up Kubernetes CLI Access:**

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 4. **Check Node Status:**

```bash
kubectl get nodes
```

![Image26](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image26.png)

### 5. **Install Calico Network Plugin:**

```bash
kubectl apply -f <https://docs.projectcalico.org/manifests/calico.yaml>
```

![Image27](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image27.png)

**Wait a few minutes, then verify the node status:** Run the following command

```bash
kubectl get nodes
```

![Image28](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image28.png)

## **Step 8: Connect to K8s Worker Node**

### 1. **Connect to Worker Node**

- Use the same steps as the Master node, using the worker node's public IP.

### 2. **Install Kubernetes on Worker Node**

- Follow the same installation steps as for the Master Node to install containerd, kubeadm, kubelet, and kubectl.

## **Step 9: Join the Work Node to the Kubernetes Cluster**

**Get Join Command from Master Node**:

- On the master node, generate the join command by running the following command

```bash
kubeadm token create --print-join-command
```

you will see like following

sudo kubeadm join 10.0.1.194:6443 --token oqxtns.kjiljprgiczfv2dv --discovery-token-ca-cert-hash sha256:313c228d0a8ca5d96c2323c93ef2d9ec2e052308204cd2f5c18d368e247e395b --ignore-preflight-errors=all

copy the above command and paste to your worker node

In the Master (Control Plane) Node, check the cluster status (It could take few moments until the node become ready)

```bash
kubectl get nodes
```

![Image29](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image29.png)

## **Step 10: Deploy React Application**

### 1. **Create React Application YAML (react-app-pod.yml)**

> **Run the following command**

```shell
 vi react-app-pod.yml
```

insert following yml code

```yml
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

### 2. **Apply the React App YAML**

```shell
kubectl create -f react-app-pod.yml
```

![Image30](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image30.png)

### 3. **Verify Pods and Services**

kubectl get pods

![Image31](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image31.png)

kubectl get services

![Image32](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image32.png)

#### **Verify that the pod is up and running**

kubectl get pods -o wide

![Image33](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image33.png)

#### **Check communication with react-app pod**

curl < react-app IP address>

Example:

```bash
curl 192.168.203.72
```

![Image34](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image34.png)

#### **Verify that the deployment complete**

kubectl get deployment

![Image35](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image35.png)

Go to the pubic IP of your Master server and port 1233. The sample react application should be running.

[http://publicip:port](http://publicip:port)

Example
`http://44.211.189.148:1233/`

![Image36](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image36.png)

Go to the pubic IP of your Worker server and port 1233 \<PublicIP\>:1233. The sample react application should be running.

Example
`http://54.161.213.12:1233/`

![Image37](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/58dbf88a3c25d088ebf725eae03719df41a520d2/Images/Image37.png)

## **Step 11: Clean up Resources**

### **Destroy the Infrastructure by running the following command**

```bash
Terraform destroy
```
