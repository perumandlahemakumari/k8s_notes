# Kubernetes on AWS with KOPS - DevOps Project Documentation

## 📦 Prerequisites

- AWS CLI configured: `aws configure`
- S3 Bucket created for KOPS state store
- SSH key generated: `ssh-keygen`

---

## ☁️ Environment Setup

```bash
export KOPS_STATE_STORE=s3://hema.kops.v1
```

---

## 🚀 Cluster Creation

```bash
kops create cluster   --name hema.k8s.local   --zones ap-south-1a   --master-size t2.medium   --node-size t2.micro

kops update cluster --name hema.k8s.local --yes --admin
```

---

## 🔐 Secrets

```bash
kubectl create secret generic firstsecret   --from-literal=username=hemakumariperumandla   --from-literal=password=Hema@1234

kubectl create secret docker-registry regcred   --docker-server=https://index.docker.io/v1/   --docker-username=hemakumariperumandla   --docker-password=Hema@1234
```

---

## 📄 Installing Kops & Kubectl

**`kops.sh`**
```sh
curl -LO "https://dl.k8s.io/release/$(curl -L -shttps://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
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

---

## 📄 Persistent Volume (PV) & Persistent Volume Claim (PVC)

**`pv.yml`**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mypv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  awsElasticBlockStore:
    volumeID: vol-0aa8d2c064e24d7d5
```

**`pvc.yml`**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

---

## 🐳 Pod Using Secret

**`pod.yml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
    - name: cont3
      image: suneel09/medical
      ports:
        - containerPort: 80
  imagePullSecrets:
    - name: regcreds
```

---

## 📋 Job Definition

**`job.yml`**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: testjob
spec:
  template:
    metadata:
      name: testjob
    spec:
      containers:
        - name: cont3
          image: ubuntu
          command: ["/bin/bash", "-c"]
          args:
            - |
              apt update -y
              touch kops.pdf
              mkdir hema
              sleep 60
      restartPolicy: Never
```

---

## 🛠️ Volume Types

### `emptyDir` (shared volume)

**`emptydir.yml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: suneel
spec:
  replicas: 1
  selector:
    matchLabels:
      app: swiggy
  template:
    metadata:
      labels:
        app: swiggy
    spec:
      containers:
        - name: cont1
          image: ubuntu
          command: ["/bin/bash", "-c", "while true; do echo Welcome to DevOps Classes; sleep 5 ; done"]
          volumeMounts:
            - name: myvolume
              mountPath: "/tmp/cont1"
        - name: cont2
          image: ubuntu
          command: ["/bin/bash", "-c", "while true; do echo Welcome to DevOps Classes; sleep 5 ; done"]
          volumeMounts:
            - name: myvolume
              mountPath: "/tmp/jenkins"
      volumes:
        - name: myvolume
          emptyDir: {}
```

### `hostPath`

**`hostpath.yml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
    - name: cont1
      image: ubuntu
      command: ["/bin/bash", "-c", "while true; do echo Welcome to DevOps Classes; sleep 5 ; done"]
      volumeMounts:
        - name: myvolume
          mountPath: "/tmp/cont1"
  volumes:
    - name: myvolume
      hostPath:
        path: "/tmp/data"
```

---

## 👨‍🔧 ConfigMaps

### From literals

```bash
kubectl create cm firstcm   --from-literal=Name=Hema   --from-literal=Course=Devops   --from-literal=Place=Hyd
```

### From file

**`aws.txt`**
```
cloud=AWS
Tech=Java
Os=Linux
```

```bash
kubectl create cm secondcm --from-file=aws.txt
```

### From environment file

**`Devops.env`**
```
SCM=Git
Build=Maven
CICD=Jenkins
```

```bash
kubectl create cm thridcm --from-env-file=Devops.env
```

### From folder

**`configmap/file1`**
```
CMMN=Ansible
Cont=Docker
Orchestratio=k8s
```

**`configmap/file2`**
```
IFAC=Terroform
monitoring=promentheus
operating=k8s
```

```bash
kubectl create cm fourthcm --from-file=configmap
```

### Manual config map

**`cm.yml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fifthcm
data:
  DATABASE_URL: "www.mysql.com"
  PORT: "3303"
```

---

## 🌐 Ingress Setup

```bash
git clone "https://github.com/suneelprojects/k8s.git"
cd k8s/ingress
kubectl create -f ingress.yml
kubectl get ing -o wide
```

---

## 📎 Helpful Commands

```bash
kubectl get po
kubectl describe cm <name>
kubectl get pv
kubectl get pvc
kubectl delete pod --all
kubectl get cm
```

---

## 📁 File Tree

```
├── aws.txt
├── Devops.env
├── configmap/
│   ├── file1
│   └── file2
├── pv.yml
├── pvc.yml
├── pod.yml
├── job.yml
├── cm.yml
├── emptydir.yml
├── hostpath.yml
├── hema.yml
└── ingress.yml
```

---

## ✅ Final Notes

- Make sure your AWS EBS volume (`volumeID`) exists and is in the correct region.
- Always confirm ConfigMap/Secret names match the references in your pods/deployments.
- For best practice, avoid hardcoding credentials—use AWS Secrets Manager or Vault.


# 🛠️ Kubernetes Setup & Configuration Guide

_Generated on 2025-05-30 14:09:12_

This document contains the detailed steps and Kubernetes manifests used to set up and manage a Kubernetes 
cluster using **kops**, including working with volumes, config maps, secrets, jobs, and ingress resources.

## 🧾 Command History

```bash
1  aws configure
2  export KOPS_STATE_STORE=s3://hema.kops.v1
3  ssh-keygen
6  vi kops.sh
7  sh kops.sh
8  kops create cluster --name hema.k8s.local --zones ap-south-1a --master-size t2.medium --node-size t2.micro
9  kops update cluster --name hema.k8s.local --yes --admin
10  vi emptydir.yml
11  kubectl create -f emptydir.yml
13  kubectl exec -it pod1 -c cont1 -- /bin/bash
    cd /tmp/cont1/
    touch git{1..5}
14  kubectl exec -it pod1 -c cont2 -- /bin/bash
    ll
    cd /tmp/cont2/
    touch jenkins{1..5}
15  kubectl exec -it pod1 -c cont1 -- /bin/bash
    ll
17  vi hostpath.yml
18  kubectl delete --all
19  kubectl delete pods --all
27  vi hostpath.yml
28  kubectl create -f hostpath.yml
29  vi hema.yml
30  kubectl create -f hema.yml
31  kubectl get deployment
32  kubectl get po
33  kubectl exec -it suneel-68f5c5bccc-hvrdv -c cont1 -- bin/bash
    cd /tmp/cont1/
    touch git{1..5}
34  kubectl exec -it suneel-68f5c5bccc-hvrdv -c cont2 -- bin/bash
    ll
    cd /tmp/cont2/
    touch jenkins{1..5}
35  kubectl delete pod --all
36  kubectl exec -it suneel-68f5c5bccc-hvrdv -c cont2 -- bin/bash
37  vi pv.yml
38  vi pvc.yml
39  kubectl create -f pv.yml
40  kubectl create -f pvc.yml
41  kubectl get pv
42  kubectl get pvc
43  kubectl delete deployment suneel
44  kubectl get pvc
45  kubetcl describe mypvc
46  kubectl describe mypvc
47  vi aws.txt
48  ll
52  kubectl create cm firstcm --from-literal=Name=Hema --from-literal=Course=Devops --from-literal=Place=Hyd
53  vi aws.txt
54  kubectl creae cm secondcm --from-file=aws.txt
55  kubectl create cm secondcm --from-file=aws.txt
56  kubectl get cm
57  kubectl describe secondcm
58  kubectl describe cm secondcm
59  vim Devops.env
60  kubectl create cm thridcm --from-env=file=Devops.env
61  kubectl create cm thridcm --from-env-file=Devops.env
62  kubectl describe thridcm
63  kubectl describe cm thridcm
64  mkdir configmap
65  cd configmap
66  touch file1 file2
67  vi file1
68  vi file2
69  kubectl create cm fourthcm --from-file=configmap
70  cd ..
71  kubectl create cm fourthcm --from-file=configmap
72  vi cm.yml
73  kubectl create -f cm.yml
74  kubectl api-resources
75  vi cm.yml
76  kubectl create -f cm.yml
77  vi cm.yml
78  kubectl create -f cm.yml
79  kubectl create secret generic firstsecret --from-literal=username=hemakumariperumandla --from-literal=password=Hema@1234
80  kubectl describe secret firstsecret
81  vi pod.yml
82  kubectl delete pod --all
83  kubectl delete pods --all
84  kubectl create -f pod.yml
85  kubectl create secret docker-registry --docker-server=https://index.docker.io.v1/ --docker-username=hemakumariperumandla --docketr-password=Hema@1234
86  kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=hemakumariperumandla --docker-password=Hema@1234
87  kubectl get secret
88  vi pod.yml
90  kubectl delete pod --all
91  kubectl create -f pod.yml
92  kubectl get po
93  vi job.yml
95  kubectl delete pods --all
96  kubectl create -f job.yml
97  kubectl get po
98  kubectl delete job testjob
99  vi job.yml
100  kubectl create -f job.yml
101  kubectl get po
102  yum install git -y
103  kubectl bget po
104  kubectl get po
105  git clone "https://github.com/suneelprojects/k8s.git"
106  cd ingress/
107  ll
108  cd k8s
109  ll
110  cd ingress/
111  vi ingress.yml
112  kubectl create -f ingress.yml
113  kubectl create -f .
114  kubectl get ing
115  kubectl get ing -o wide
116  kubectl get ing
117  kubectl create -f ingress.yml
118  kubectl get ing
119  cd ..
```
