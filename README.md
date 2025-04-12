# Kubernetes Project: End-to-End Deployment on EKS Using eksctl

## Project Overview

This project focuses on deploying the **2048 game** application on an **Amazon EKS (Elastic Kubernetes Service)** cluster. The application runs as a pod within the cluster and is initially accessible only internally via **ClusterIP**.

To enable external access, we use a **Kubernetes Ingress Controller** (like ALB or NGINX Ingress Controller) to route external traffic to the application. The ingress resource defines URL routing rules, while the ingress controller manages the load balancer configuration, ensuring cost-effective and secure access.

This setup provides a scalable, secure, and efficient way to expose the game to external users without the high costs of multiple load balancers.

# Tools and Technologies Used

In this project, the following tools and technologies were utilized to deploy the 2048 game on Amazon EKS:

- **kubectl**: Used for interacting with and managing the Kubernetes cluster.
- **eksctl**: A command-line tool for easily creating and managing EKS clusters.
- **AWS CLI**: Configured for interacting with AWS services like EKS and IAM.
- **IAM**: Used for configuring necessary permissions and roles to ensure secure access to AWS resources.
- **Helm**: A package manager for Kubernetes, used to deploy the AWS Load Balancer Controller.
- **Ingress Controller**: Manages traffic routing within the cluster; we used the **ALB Ingress Controller** for creating and managing Application Load Balancers.

These tools provided a streamlined, automated, and secure deployment of the 2048 game on EKS.

# Detailed Overview

### 1. Install and Configure Tools

Before we begin, make sure you have the following tools installed on your local machine:
- **kubectl**: Kubernetes command-line tool to interact with your Kubernetes cluster.
- **eksctl**: A command-line utility to create and manage EKS clusters.
- **AWS CLI**: AWS Command Line Interface to interact with AWS services.

## Installing `kubectl` on Linux Using `curl`

The Kubernetes command-line tool, `kubectl`, allows you to run commands against Kubernetes clusters. Here's a step-by-step guide to installing it manually using `curl`.

### Step 1: Download the Latest Release

Choose the appropriate command based on your system architecture:

- **For x86_64 (AMD64):**
```shell
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

- **For ARM64:**
```shell
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/arm64/kubectl
```

### 📌 Downloading a Specific Version

To fetch a particular version, replace the `$(...)` with the desired version tag:

```shell
curl -LO https://dl.k8s.io/release/v1.32.0/bin/linux/amd64/kubectl
```

Or for ARM64:
```shell
curl -LO https://dl.k8s.io/release/v1.32.0/bin/linux/arm64/kubectl
```

---

### Step 2: (Optional) Verify Binary Integrity

Download the SHA256 checksum:
```shell
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```

Then validate it:
```shell
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```

Successful output:
```
kubectl: OK
```

---

### Step 3: Install the Binary

Move the binary into your system's executable path:
```shell
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### 🔒 Without Root Access

If you lack root privileges:
```shell
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
export PATH=$PATH:~/.local/bin  # Add this to your shell profile
```

---

### Step 4: Verify Installation

Check the version to confirm installation:
```shell
kubectl version --client
```

For more detailed output:
```shell
kubectl version --client --output=yaml
```

## Installing `eksctl`

To install `eksctl`, the official command-line tool for Amazon Elastic Kubernetes Service (EKS), on your Linux system, follow these steps:

### 1. Determine Your System Architecture

Identify your system's architecture to download the appropriate version of `eksctl`. Open your terminal and run:

```shell
uname -m
```

- **x86_64 (AMD64):** Output is `x86_64`.
- **ARM-based Systems:** Output is `aarch64` (for `arm64`), `armv6l`, or `armv7l`.

Based on the output, set the `ARCH` variable:

For `x86_64` systems:
```shell
ARCH=amd64
```

For ARM-based systems:
```shell
ARCH=arm64  # or armv6 or armv7, depending on your system
```

### 2. Download the `eksctl` Binary

Construct the download URL using your system's architecture and platform, then download and extract the `eksctl` binary:

```shell
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
```

This will extract the `eksctl` binary to the `/tmp` directory.

### 3. Install the Binary

To make `eksctl` accessible system-wide, move it to a directory included in your system's `PATH`, such as `/usr/local/bin`:

```shell
sudo mv /tmp/eksctl /usr/local/bin
```

If you don't have root access, you can install `eksctl` to a directory within your user space:

```shell
mkdir -p ~/.local/bin
mv /tmp/eksctl ~/.local/bin/eksctl
```

Ensure that `~/.local/bin` is included in your `PATH`. Add the following line to your shell's configuration file (e.g., `~/.bashrc` or `~/.zshrc`):

```shell
export PATH=$PATH:~/.local/bin
```

After editing the configuration file, apply the changes:

```shell
source ~/.bashrc  # or source ~/.zshrc
```

### 4. Verify the Installation

Confirm that `eksctl` is correctly installed by checking its version:

```shell
eksctl version
```

This command should display the installed version of `eksctl`.

**Optional: Verify the Downloaded Binary's Integrity**

To ensure the integrity of the downloaded `eksctl` binary, you can verify its checksum:

1. **Download the Checksums File:**
```shell
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" -o eksctl_checksums.txt
```

2. **Verify the Checksum:**
```shell
grep $PLATFORM eksctl_checksums.txt | sha256sum --check
```

If the checksum matches, the output will be:
```
eksctl_<platform>: OK
```

If the check fails, the output will indicate a mismatch.

By following these steps, you should have `eksctl` successfully installed on your Linux machine, ready to manage your Amazon EKS clusters. For more detailed information and advanced configurations, refer to the [official eksctl documentation](https://eksctl.io/installation/).

## Installing `AWS CLI`

### Installing AWS CLI v2 on Linux

To install or update the AWS Command Line Interface (CLI) version 2 on a Linux system, follow the steps below. These instructions are for 64-bit (x86_64) systems.

### Step 1: Download the Installer
```shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

### Step 2: Unzip the Package
```shell
unzip awscliv2.zip
```

### Step 3: Run the Installer
```shell
sudo ./aws/install
```

This installs the AWS CLI to `/usr/local/aws-cli` and creates a symbolic link at `/usr/local/bin/aws`.

### Step 4 (Optional): Update an Existing Installation

If you're updating from a previous version, use the `--update` flag:
```shell
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
```

### Step 5: Verify the Installation
```shell
aws --version
```

You should see output like:
```
aws-cli/2.x.x Python/X.X.X Linux/X.X.X botocore/X.X.X
```

---

For advanced verification (PGP signatures, integrity checks) or ARM-based systems, refer to the official documentation:

👉 [AWS CLI Installation Guide (Linux)](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)

**After installing the tools, configure AWS CLI:**

After successfully installing the AWS Command Line Interface (CLI) on your Linux system, it's essential to configure your AWS credentials to enable secure and authorized access to AWS services. Follow the steps below to set up your AWS credentials:

**1. Obtain AWS Access Keys:**

Before configuring the AWS CLI, ensure you have your AWS Access Key ID and Secret Access Key. These credentials are typically generated through the AWS Management Console under the IAM (Identity and Access Management) section. If you don't have them, follow [AWS's guide to creating access keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html).

**2. Configure AWS CLI Credentials:**

The AWS CLI provides a straightforward method to configure your credentials using the `aws configure` command.

```shell
aws configure
```

When prompted, enter your Access Key ID, Secret Access Key, preferred AWS Region, and desired output format:

```
AWS Access Key ID [None]: YOUR_ACCESS_KEY_ID
AWS Secret Access Key [None]: YOUR_SECRET_ACCESS_KEY
Default region name [None]: us-west-2  # Replace with your preferred region
Default output format [None]: json  # Options include json, yaml, text, etc.
```

This command updates two configuration files located in the `~/.aws/` directory:

- **`config`**: Stores configuration settings such as the default region and output format.
- **`credentials`**: Stores your AWS Access Key ID and Secret Access Key.

**3. Manual Configuration (Optional):**

If you prefer manual configuration or need to set up multiple profiles, you can directly edit the `config` and `credentials` files.

- **`~/.aws/config`**:
```
[default]
region = us-west-2  # Replace with your preferred region
output = json       # Options include json, yaml, text, etc.
```

- **`~/.aws/credentials`**:
```
[default]
aws_access_key_id = YOUR_ACCESS_KEY_ID
aws_secret_access_key = YOUR_SECRET_ACCESS_KEY
```

For additional details on configuring the AWS CLI, refer to [AWS's official documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html).

**4. Verify Configuration:**

To ensure your AWS CLI is correctly configured, execute a simple command such as listing your S3 buckets:

```shell
aws s3 ls
```

If configured correctly, this command will display a list of your S3 buckets.

**Security Best Practices:**

- **Avoid Hardcoding Credentials:** Do not hardcode your AWS credentials in application code. Instead, use environment variables or the AWS credentials file.
- **Use IAM Roles:** When operating within AWS services like EC2 or ECS, utilize IAM roles for enhanced security.
- **Rotate Credentials Regularly:** Periodically rotate your AWS access keys to maintain security.

By following these steps, you can securely configure your AWS CLI credentials, ensuring safe and authorized interactions with AWS services.

### 2. Create an EKS Cluster Using eksctl

Now that your tools are set up, let's create the EKS cluster using `eksctl`:

```shell
# Creating EKS Cluster
eksctl create cluster --name 2048-game-cluster --region ap-south-1 --fargate
```

**Explanation:**
- **`-name my-cluster`**: The name of your Kubernetes cluster.
- **`-region us-east-1`**: The AWS region where the cluster will be created.
- **`-fargate`**: Specifies that the cluster will use AWS Fargate for serverless compute.

This command will:
- Create the necessary VPC (Virtual Private Cloud).
- Set up public and private subnets.
- Create an IAM role for the cluster.
- Set up networking and security settings for the cluster.

**Note**: The cluster creation process may take 15-20 minutes.

### 3. Verify the Cluster Creation

After the cluster is created, go to the AWS Management Console, navigate to the **EKS** service, and check if the cluster has been created. You should see the cluster with its Kubernetes version and status.

### 4. Update `kubeconfig` to Access Your Cluster

To interact with the cluster using `kubectl`, update your `kubeconfig` file:

```shell
aws eks update-kubeconfig --name 2048-game-cluster --region ap-south-1
```

This command will allow `kubectl` to interact with your newly created EKS cluster.

### 5. Create a Fargate Profile

A **Fargate profile** tells the EKS cluster which pods should run on Fargate. We will create a Fargate profile for our application namespace:

```shell
eksctl create fargateprofile \
--cluster 2048-game-cluster \
--region ap-south-1 \
--name alb-sample-app \
--namespace 2048-game
```

**Explanation**:
- **`-name alb-sample-app`**: Name for the Fargate profile.
- **`-namespace game-2048`**: The Kubernetes namespace where the pods will be deployed.

### 6. Deploy the 2048 Application

We are now ready to deploy the 2048 application. We will apply the necessary Kubernetes resources for the game.

First, create the resources using the following command:

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

This command will create:
- A **namespace** (`game-2048`).
- A **pod**, **service**, **deployment**, and **ingress** for the game application.

### 7. Check the Resources Created

To verify that the resources are created properly:

- **Check Pods**:
```shell
kubectl get pods -n game-2048
```

- **Check Services**:
```shell
kubectl get svc -n game-2048
```

- **Check Ingress**:
```shell
kubectl get ingress -n game-2048
```

At this stage, the application is running, but it won't be accessible from the outside world until we set up an **Ingress Controller**.

### 8. Set Up an Ingress Controller

To make the application accessible from the outside world, we need to deploy an **Application Load Balancer (ALB) Ingress Controller**. This controller will automatically create a load balancer and configure it to route traffic to the correct Kubernetes services.

**Step 1: Associate IAM OIDC Provider**

Before we can use the ALB Ingress Controller, we need to associate an IAM OIDC (OpenID Connect) provider with the cluster to allow Kubernetes pods to access AWS services.

```shell
eksctl utils associate-iam-oidc-provider --cluster my-cluster --approve
```

**Step 2: Create IAM Policy**

Next, create an IAM policy that grants the ALB Ingress Controller the necessary permissions to manage AWS resources:

```shell
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
```

**Step 3: Create IAM Role and Attach Policy**

Now create an IAM service account for the ALB Ingress Controller and attach the IAM policy:

```shell
eksctl create iamserviceaccount \
--cluster=my-cluster \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--role-name AmazonEKSLoadBalancerControllerRole \
--attach-policy-arn=arn:aws:iam::<aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
--approve
```

**Step 4: Install ALB Ingress Controller Using Helm**

Helm is a package manager for Kubernetes. Use Helm to install the ALB Ingress Controller:

```shell
helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
-n kube-system \
--set clusterName=my-cluster \
--set serviceAccount.name=aws-load-balancer-controller \
--set region=us-east-1 \
--set vpcId=<vpc-id>
```

This will deploy the ALB Ingress Controller to the cluster and configure it to create load balancers in your VPC.

### 9. Verify the Ingress Controller

To verify the ALB Ingress Controller is running:

```shell
kubectl get deployment -n kube-system aws-load-balancer-controller
```

You should see two replicas running in two availability zones, continuously monitoring and managing ingress resources.

### 10. Verify the Ingress and Get Load Balancer URL

Now, check the ingress resource to get the load balancer's address:

```shell
kubectl get ingress -n game-2048
```

After a short time, you should see an external URL in the **ADDRESS** field.

### 11. Access the Application

Once the load balancer is active, copy the URL provided in the ingress and open it in your browser:

```
http://<load-balancer-url>
```

You should now be able to access the 2048 game running on EKS via the load balancer!
