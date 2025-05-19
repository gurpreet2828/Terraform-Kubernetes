
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

![Image3](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/9bc9affe2c6baf0846cd729b516f49e255c59c1e/Images/Image3.png)

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

![Image12](https://github.com/gurpreet2828/Terraform-Kubernetes/blob/ccb2abe80634611295238d8963ef24d7be384e10/Images/Image12.png)

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

## **Step 5: Connect to K8s Master (Control Plane) Node**  {#step-5-connect-to-k8s-master-control-plane-node}

Using the public IP address provided in the Terraform output, connect to the EC2 instance by executing the following command in your terminal:

ssh -i /root/.ssh/docker ec2-user@34.201.56.12

![A screenshot of a computer program AI-generated content may be
incorrect.](media/image14.png){width="7.5in"
height="3.3819444444444446in"}

## **Step 6: Install and Configure Kubernetes Master (Control Plane) Node**

**Follow these steps to set up the Kubernetes Control Plane node
effectively:**

**Update the System and Install Dependencies**

Run the following commands to update the system and install essential
packages:

sudo yum update -y

sudo yum install -y curl wget git

**Disable Swap**

Kubernetes requires swap to be disabled. Execute:

sudo swapoff -a

sudo sed -i \'/ swap / s/\^\\.\*\\\$/#\1/g\' /etc/fstab

**Load Modules for containerd:**

sudo modprobe overlay

sudo modprobe br_netfilter

**Set Up sysctl Parameters for Kubernetes Networking**

cat \<\<EOF \| sudo tee /etc/sysctl.d/99-kubernetes-cri.conf

net.bridge.bridge-nf-call-iptables = 1

net.ipv4.ip_forward = 1

net.bridge.bridge-nf-call-ip6tables = 1

EOF

sudo sysctl \--system

![A screen shot of a computer AI-generated content may be
incorrect.](media/image15.png){width="7.5in" height="4.21875in"}

**CRI-Tools**

cri-tools is a set of command-line utilities for working with container
runtimes that implement the Container Runtime Interface (CRI) in
Kubernetes.

**Download and Install the Latest cri-tools RPM:**

cd \~

curl -LO
https://download.opensuse.org/repositories/isv:/kubernetes:/core:/stable:/v1.30/rpm/x86_64/cri-tools-1.30.0-150500.1.1.x86_64.rpm

sudo yum localinstall -y cri-tools-1.30.0-150500.1.1.x86_64.rpm

sudo sysctl --system

**Install containerd**

**Update the system:**

sudo yum update -y

**Install containerd:**

sudo yum install -y containerd

![A screenshot of a computer AI-generated content may be
incorrect.](media/image16.png){width="7.5in" height="4.21875in"}

**Create the configuration directory:**

sudo mkdir -p /etc/containerd

**Generate containerd configuration:**

containerd config default \| sudo tee /etc/containerd/config.toml

![A computer screen shot of a black screen AI-generated content may be
incorrect.](media/image17.png){width="7.5in" height="4.21875in"}

**Restart containerd:**

sudo systemctl restart containerd

**Verify containerd status:**

sudo systemctl status containerd

![A computer screen shot of a black screen AI-generated content may be
incorrect.](media/image18.png){width="7.5in" height="4.21875in"}

**Install Kubernetes Components (kubeadm, kubelet, kubectl**

**Add Kubernetes repository:**

cat \<\<EOF \| sudo tee /etc/yum.repos.d/kubernetes.repo

\[kubernetes\]

name=Kubernetes

baseurl=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/

enabled=1

gpgcheck=1

gpgkey=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/repodata/repomd.xml.key

exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni

EOF

![A black screen with white text AI-generated content may be
incorrect.](media/image19.png){width="7.5in"
height="2.6798611111111112in"}

**Set SELinux in Permissive Mode**

sudo setenforce 0

sudo sed -i \'s/\^SELINUX=enforcing\$/SELINUX=permissive/\'
/etc/selinux/config

![](media/image20.png){width="7.5in" height="0.83125in"}

**Install kubeadm, kubelet, and kubectl**

sudo yum install -y kubelet kubeadm kubectl
\--disableexcludes=kubernetes

![A screen shot of a computer screen AI-generated content may be
incorrect.](media/image21.png){width="7.5in" height="4.21875in"}

**Enable and Start kubelet**

sudo systemctl enable \--now kubelet

![](media/image22.png){width="7.5in" height="0.3888888888888889in"}

## **Step 7: Initialize the Kubernetes Cluster and Install Calico Network**

1.  **Create Kubeadm Config File:**

vi kube-config.yml

**Insert the following content in above yml file**

apiVersion: kubeadm.k8s.io/v1beta3

kubernetesVersion: 1.32.0

\# This is a configuration file for kubeadm to set up a Kubernetes
cluster.

kind: ClusterConfiguration

networking:

  podSubnet: 192.168.0.0/16

apiServer:

  extraArgs:

   service-node-port-range: 1024-1233

2.  **Initialize Kubernetes Cluster:**

sudo kubeadm init \--config kube-config.yml
\--ignore-preflight-errors=all

**Note:** The purpose of \--ignore-preflight-errors=all flag is to
ignore the K8s HW requirements

![A computer screen with many small white and blue text AI-generated
content may be incorrect.](media/image23.png){width="7.5in"
height="4.21875in"}

3.  **Set Up Kubernetes CLI Access:**

mkdir -p \$HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf \$HOME/.kube/config

sudo chown \$(id -u):\$(id -g) \$HOME/.kube/config

4.  **Check Node Status:**

kubectl get nodes

![A black background with white text AI-generated content may be
incorrect.](media/image24.png){width="7.5in"
height="0.9319444444444445in"}

5.  **Install Calico Network Plugin:**

kubectl apply -f <https://docs.projectcalico.org/manifests/calico.yaml>

![A screen shot of a computer screen AI-generated content may be
incorrect.](media/image25.png){width="7.5in"
height="4.447916666666667in"}

**Wait a few minutes, then verify the node status:** Run the following
command

kubectl get nodes

## ![A black screen with white text AI-generated content may be incorrect.](media/image26.png){width="7.5in" height="1.2409722222222221in"} {#a-black-screen-with-white-text-ai-generated-content-may-be-incorrect.}

## 

## **Step 8: Connect to K8s Worker Node**

1.  **Connect to Worker Node**

- Use the same steps as the Master node, using the worker node\'s public
  IP.

2.  **Install Kubernetes on Worker Node**

- Follow the same installation steps as for the Master Node to install
  containerd, kubeadm, kubelet, and kubectl.

## **Step 9: Join the Work Node to the Kubernetes Cluster**  {#step-9-join-the-work-node-to-the-kubernetes-cluster}

**Get Join Command from Master Node**

- On the master node, generate the join command by running the following
  command

kubeadm token create \--print-join-command

you will see like following

sudo kubeadm join 10.0.1.194:6443 \--token oqxtns.kjiljprgiczfv2dv
\--discovery-token-ca-cert-hash
sha256:313c228d0a8ca5d96c2323c93ef2d9ec2e052308204cd2f5c18d368e247e395b
\--ignore-preflight-errors=all

copy the above command and paste to your worker node

In the Master (Control Plane) Node, check the cluster status (It could
take few moments until the node become ready)

kubectl get nodes

![A screenshot of a computer program AI-generated content may be
incorrect.](media/image27.png){width="7.5in"
height="0.9930555555555556in"}

## 

## **Step 10: Deploy React Application**

1.  **Create React Application YAML (react-app-pod.yml)**

> **Run the following command**
>
> vi react-app-pod.yml

apiVersion: v1

kind: Service

metadata:

name: react-app

spec:

type: NodePort

ports:

\- port: 80

targetPort: 80

nodePort: 1233

selector:

app: react-app

\-\--

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

\- name: react-app

image: \<your-docker-hub-image\>

ports:

\- containerPort: 80

2.  **Apply the React App YAML**

kubectl create -f react-app-pod.yml

> ![A black screen with white text AI-generated content may be
> incorrect.](media/image28.png){width="6.833333333333333in"
> height="0.8826388888888889in"}

3.  **Verify Pods and Services**

> kubectl get pods
>
> ![A screenshot of a computer program AI-generated content may be
> incorrect.](media/image29.png){width="7.5in"
> height="1.0881944444444445in"}
>
> kubectl get services
>
> ![A screen shot of a computer AI-generated content may be
> incorrect.](media/image30.png){width="6.65in"
> height="1.2513888888888889in"}

**Verify that the pod is up and running**

kubectl get pods -o wide

![](media/image31.png){width="7.5in" height="0.6076388888888888in"}

**Check communication with react-app pod**

curl \< react-app IP address\>

ex: curl 192.168.203.72

![A black screen with many small colored lines AI-generated content may
be incorrect.](media/image32.png){width="7.5in"
height="2.154861111111111in"}

**Verify that the deployment complete**

kubectl get deployment

![A screen shot of a computer AI-generated content may be
incorrect.](media/image33.png){width="7.5in"
height="1.3590277777777777in"}

Go to the pubic IP of your Master server and port 1233 \<Public
IP\>:1233. The sample react application should be running.

Ex: http://44.211.189.148:1233/

![A screenshot of a computer AI-generated content may be
incorrect.](media/image34.png){width="7.5in" height="4.21875in"}

Go to the pubic IP of your Worker server and port 1233 \<Public
IP\>:1233. The sample react application should be running.

Ex: http://54.161.213.12:1233/

![A screenshot of a computer AI-generated content may be
incorrect.](media/image35.png){width="7.5in"
height="3.933333333333333in"}

## **Step 11: Clean up Resources**

**Destroy the Infrastructure by running the following command**

Terraform destroy
