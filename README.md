# Docker image : docker pull harshj2003/flaskapp

# **Two-Tier Application Deployment on AWS EKS**  
*A DevOps Guide to Deploying Flask + MySQL on Kubernetes*  

## **Project Overview**  
**Objective**:  
Deploy a two-tier Flask and MySQL application on **Amazon EKS** with:  
- **Frontend**: Flask app (exposed via `LoadBalancer`)  
- **Backend**: MySQL (`ClusterIP` service with persistent storage)  
- **Infrastructure**: 2-node EKS cluster  

### **Architecture**  
```plain
[Internet] → [ELB:80] → [Flask Pods] → [MySQL Service] → [MySQL Pod]  
                     ↑               ↑  
                     │               └── Persistent Volume  
                     └── EKS Worker Nodes  
```

---

## **Infrastructure Setup**  

### **1. IAM Configuration**  
Create an IAM user `eks-admin` with `AdministratorAccess` and generate **Access Keys** for CLI authentication.  

### **2. Launch EC2 **  
- **Region**: `ap-south-1` (as per your requirements)  
- **AMI**: Ubuntu  / Amazon Linux / RedHat (as per your requirements)
- **Purpose**: Manage EKS deployments  

#### **SSH & Tool Installation**  
```bash
ssh -i "your-key.pem" ec2-user@<EC2-Public-IP>
```

#### **Install AWS CLI v2**  
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure  # Set Access Key, Secret Key, and Region (us-west-2)
```

#### **Install Docker**  
```bash
sudo apt-get update
sudo apt install docker.io
sudo usermod -aG docker $USER
sudo chown $USER /var/run/docker.sock
```

#### **Install kubectl**  
```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

#### **Install eksctl**  
```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### **3. Create EKS Cluster**  
```bash
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes
```

---

## **Application Deployment**  

### **1. Clone the Repository**  
```bash
git clone https://github.com/Harshwardhanjadhav/two-tier-flask-eks.git
cd two-tier-flask-eks
```

### **2. Deploy MySQL Components**  
```bash
kubectl apply -f mysql-secrets.yml       # Database credentials (base64)
kubectl apply -f mysql-configmap.yml    # DB configuration
kubectl apply -f mysql-deployment.yml   # MySQL pod
kubectl apply -f mysql-svc.yml          # ClusterIP service
```

### **3. Deploy Flask App**  
```bash
kubectl apply -f two-tier-app-deployment.yml
kubectl apply -f two-tier-app-svc.yml   # Creates a LoadBalancer
```

### **4. Verify Deployment**  
```bash
kubectl get pods,svc
```
Wait for the `EXTERNAL-IP` of the Flask app to populate, then access it at:  
`http://<ELB-DNS>`

---

## **Security Best Practices**  
- **IAM**: Restrict permissions using `AmazonEKSClusterPolicy` (avoid `AdministratorAccess`).  
- **Secrets**: Use **AWS Secrets Manager** instead of hardcoding in YAML.  
- **Networking**: Implement **Network Policies** to restrict pod communication.  

---

## **Verification & Testing**  
| Command | Description |  
|---------|------------|  
| `kubectl get pods -o wide` | Check pod status and IPs |  
| `kubectl logs <pod-name>` | Debug container logs |  
| `kubectl exec -it <mysql-pod> -- mysql -u root -p` | Test MySQL connection |  
| `curl http://<ELB-DNS>` | Test Flask app |  

---

## **Appendix: Cheat Sheet**  
```bash
# Cluster Management
eksctl delete cluster --name three-tier-cluster --region us-west-2  # Cleanup

# Debugging
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## **Conclusion**  
✅ **Automated EKS provisioning** with `eksctl`  
✅ **Deployed a scalable two-tier app** (Flask + MySQL)  
✅ **Implemented Kubernetes best practices**  

**Next Steps**:  
- Automate with **Terraform/Helm**  
- Set up **CI/CD (GitHub Actions/ArgoCD)**  
- Add **monitoring (Prometheus/Grafana)**  

🚀 **Happy DevOps-ing!** 
