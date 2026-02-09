# shopping_site_EKS_along_AI-ML_integration

## Overview on a High Level as to what are going to do:

1. Create EKS Cluster
2. Deploy Microservices
3. Execute Python code which invokes Bedrock - which will query the K8s cluster

Brought up my machine with sufficient memory and storage to (whatever 8 GB machine covered under the FREE Tier 😅) </br>
<img width="1622" height="804" alt="Image" src="https://github.com/user-attachments/assets/14380845-ae15-4bbc-8df0-1c2c13292286" />

---

Then tried connecting to it using my **WSL** terminal but it was not connecting.
<img width="899" height="41" alt="Image" src="https://github.com/user-attachments/assets/a1151e0d-aa6e-49ec-a17c-962997143282" />

---

Then I thought maybe there was some issue with the security group as I had taken the default SG, and yes that was the issue:
<img width="1814" height="271" alt="Image" src="https://github.com/user-attachments/assets/5fe4b432-c4d9-4504-adbd-5d7b806da00e" />

---

So temporarily I gave it SSH permission from anywhere:
<img width="1814" height="522" alt="Image" src="https://github.com/user-attachments/assets/15157627-0409-4b37-b260-34d035021c51" />

---

And it worked but there was some issue with the permission of the pem file as general downloading of the pem file through AWS makes permission too open by default as we can see in the error message.
<img width="1053" height="269" alt="Image" src="https://github.com/user-attachments/assets/3b2386f7-66d0-4a8f-8174-06d361a666ec" />


---

So I changed the permission and tried reconnecting and finally it worked
<img width="1063" height="277" alt="Image" src="https://github.com/user-attachments/assets/f2c338b2-365d-4fdd-b7c8-78b83f7fd54a" />

---

Now comes creation of a user with a few permissions: one for EKS cluster creation (AdministratorAccess), one for Bedrock Full Access as we are going to run Bedrock and then one inline policy so that Bedrock can query the EKS Cluster.

<img width="960" height="675" alt="Image" src="https://github.com/user-attachments/assets/3ab2968c-01ce-4597-863c-2d8098255b7e" />

And then we create an Access Key for us to enable the AWS CLI to access our AWS account. We would have needed to install AWS CLI too if our AMI were not Amazon's AMI - here it's pre installed, we just need to give the create the necessary Access Key and we are done.

<img width="1474" height="738" alt="Image" src="https://github.com/user-attachments/assets/63810939-dd9d-4206-9933-5f84f396689e" />

<img width="1104" height="141" alt="Image" src="https://github.com/user-attachments/assets/94fd68f4-13b0-44fc-b82a-364cd1109886" />

---

Now we follow a few instructions for setting up an Amazon EKS cluster with EC2 instances as worker nodes, all using the AWS CLI.

Perform this step only if you are using AMI other than Amazon AMI
```bash
# Download and install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt-get install -y unzip
unzip awscliv2.zip
sudo ./aws/install

# Verify installation
aws --version

# Configure AWS CLI
aws configure
# Enter your AWS Access Key ID, Secret Access Key, region (us-east-1), and output format (json)
```

```bash
# Download kubectl binary from Amazon EKS (amd64 architecture)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Validate the binary (optional) - Download the kubectl checksum file:
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

'''Note:
To download a specific version, replace the $(curl -L -s https://dl.k8s.io/release/stable.txt) portion of the command with the specific version.
For example, to download version 1.35.0 on Linux x86-64, type:'''
curl -LO https://dl.k8s.io/release/v1.35.0/bin/linux/amd64/kubectl
curl -LO https://dl.k8s.io/release/v1.35.0/bin/linux/amd64/kubectl.sha256
```

<img width="1087" height="447" alt="Image" src="https://github.com/user-attachments/assets/0396a93d-b038-404c-8921-640e3f117f43" />

```basg
# Make kubectl executable
chmod +x ./kubectl

# Move kubectl to bin directory and add to PATH
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH

# Add to bashrc for persistence
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc

# Verify installation
kubectl version --client
```
---

```bash
# Downloading eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    
sudo mv /tmp/eksctl /usr/local/bin

eksctl version

# Creating cluster without the nodegroup to save us from the possiblity of cluster creation time out
eksctl create cluster --name=eksshopping --region=us-east-1 --zones=us-east-1a,us-east-1b,us-east-1c --without-nodegroup
```

<img width="1192" height="652" alt="Image" src="https://github.com/user-attachments/assets/0049e89a-6236-453c-a8bc-d57b6e0a7b1d" />

It creates CloudFormation too:

<img width="1908" height="692" alt="Image" src="https://github.com/user-attachments/assets/10b5984a-e70d-4812-af61-857fac468565" />

<img width="1911" height="694" alt="Image" src="https://github.com/user-attachments/assets/fcfa8ba1-0fa5-4c04-8f00-f6fb56a88b90" />

Cluster is Active and CloudFormation stack is also created:

<img width="1914" height="568" alt="Image" src="https://github.com/user-attachments/assets/a39c9187-4d94-45b0-990e-b3cbe7901f5f" />

<img width="1761" height="604" alt="Image" src="https://github.com/user-attachments/assets/f8b67392-ebc2-4379-875c-14caa00348c0" />

---

```bash
# Creating the nodegroup with the specification for worker nodes (we can change the machine configuration as per our need)
eksctl create nodegroup --cluster=eksshopping --region=us-east-1 --name=eksshopping-ng-public1 --node-type=c7i-flex.large --nodes=2 --nodes-min=1 --nodes-max=3 --node-volume-size=30 --ssh-access --ssh-public-key=shopping-site-key --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access --spot
```

Creation has started:
<img width="1873" height="420" alt="Image" src="https://github.com/user-attachments/assets/a95728df-81aa-414c-98fe-5d00c7dd498e" />

It has also called for another CloudFormation Stack:
<img width="1872" height="432" alt="Image" src="https://github.com/user-attachments/assets/4a03911a-37a2-463b-88bd-2320b7ee47d4" />

Finally, both are done:
<img width="798" height="479" alt="Image" src="https://github.com/user-attachments/assets/83f22e4d-15c8-4c38-a110-f6f1c97392ae" />

<img width="1919" height="796" alt="Image" src="https://github.com/user-attachments/assets/cb0643d5-d22e-4847-acf0-6e264dd8e030" />




























