# Kubernetes Setup and Deployment Commands

This README provides a step-by-step shell command reference to create and manage a Kubernetes cluster using KOPS, deploy pods and services, and use Helm for application deployment.

## ⚙️ AWS & KOPS Setup

```bash
1.  aws configure
    # Configure AWS CLI with your credentials

2.  vi kops.sh
    # Open kops.sh script for editing

3.  sh kops.sh
    # Run the kops.sh script

4.  export KOPS_STATE_STORE=s3://hema.kops.v1
    # Set S3 bucket for KOPS cluster state

5.  kops create cluster --name hema.k8s.local --zones ap-south-1a --master-size t2.medium --node-size t2.micro
    # Create a Kubernetes cluster

6.  kops update cluster --name hema.k8s.local --yes --admin
    # Apply the cluster configuration
```

## 📦 Pod & Service Deployment

```bash
7.  vi pod.yml
    # Define a pod specification

8.  vi clusterip.yml
    # Define a ClusterIP service

9.  kubectl create -f pod.yml
    # Create pod

12. kubectl create -f pod.yml
14. kubectl create -f pod.yml
15. kubectl create -f clusterip.yml
```

## 🔍 Service & Pod Info

```bash
16. kubetcl get po   # Typo (should be kubectl)
17. kubectl get po   # Get pods
18. kubectl get svc  # Get services
19. kubectl describe svc hema
20. kubectl delete svc hema
```

## 🌐 NodePort Service

```bash
21. cp clusterip.yml np.yml
22. vi np.yml        # Convert to NodePort
23. kubectl create -f np.yml
24. kubectl get svc
```

## 🚀 Application Deployment

```bash
25. vi hema.yml
27. kubectl create -f hema.yml
29. kubectl create -f hema.yml
30. kubectl create -f np.yml
34. kubectl create -f np.yml
```

## 🔁 LoadBalancer Service

```bash
35. kubectl delete svc --all
36. cp np.yml Loadbal.yml
37. vi Loadbal.yml   # Define LoadBalancer service
38. kubectl create -f Loadbal.yml
39. kubectl get svc
```

## 🔧 Git & StatefulSet

```bash
40. yum install git -y
41. git clone "https://github.com/suneelprojects/k8s-statefullset.git"
43. cd k8s-statefullset
45. kubectl create -f .
46. kubectl get ns
47. kubectl get po -n medilab-db-ns
48. kubectl delete pod medilab-db-deployment-2 -n medilab-db-ns
```

## 🛠️ Helm Setup

```bash
49. curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
50. chmod 700 get_helm.sh
51. ./get_helm.sh
52. helm version
54. helm create devops
57. cd templates/
59. vi deployment.yaml
62. vi values.yaml
63. helm install hema .
64. helm list
```

## 🧹 Cleanup

```bash
66. kubectl delete statefulset --all -n medilab-db-ns
67. kubectl delete pod --all -n medilab-db-ns
71. kops delete cluster --name hema.k8s.local --yes
```

## 📝 Notes

- Ensure you have IAM permissions for KOPS and S3.
- Use correct Kubernetes YAML definitions for resources.
- Helm 3+ is recommended for Helm commands.

---

## 📁 YAML & Script References

### 📄 clusterip.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hema
spec:
  type: ClusterIP
  selector:
    env: dev
  ports:
    - port: 80
      targetPort: 80
```

### 📄 hema.yml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    app: swiggy
spec:
  containers:
    - name: cont3
      image: suneel09/app
      ports:
        - containerPort: 80
```

### ⚙️ kops.sh

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
wget https://github.com/kubernetes/kops/releases/download/v1.24.1/kops-linux-amd64
chmod +x kops-linux-amd64 kubectl
mv kubectl /usr/local/bin/kubectl
mv kops-linux-amd64 /usr/local/bin/kops

sudo curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
sudo echo "$(cat kubectl.sha256) kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

kubectl version
kops version
```

### 📄 Loadbal.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: devops
spec:
  type: LoadBalancer
  selector:
    app: swiggy
  ports:
    - port: 80
      targetPort: 80
```

### 📄 np.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: devops
spec:
  type: NodePort
  selector:
    app: swiggy
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

### 📄 pod.yml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    env: dev
spec:
  containers:
    - name: cont3
      image: nginx
      ports:
        - containerPort: 80
```
