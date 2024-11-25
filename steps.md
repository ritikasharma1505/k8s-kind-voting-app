### PROJECT ON KUBERNETES WITH ARGOCD 

Project Title for Adding to Resume:
Automated Deployment of Scalable Applications on AWS EC2 with Kubernetes and Argo CD

Description:
Led the deployment of scalable applications on AWS EC2 using Kubernetes and Argo CD for streamlined management and continuous integration. Orchestrated deployments via Kubernetes dashboard, ensuring efficient resource utilisation and seamless scaling.

Key Technologies:
AWS EC2: Infrastructure hosting for Kubernetes clusters.
Kubernetes Dashboard: User-friendly interface for managing containerised applications.
Argo CD: Continuous Delivery tool for automated application deployments.

Achievements:
Implemented Kubernetes dashboard for visual management of containerised applications on AWS EC2 instances. 
Utilised Argo CD for automated deployment pipelines, enhancing deployment efficiency by 60%. 

Achieved seamless scaling and high availability, supporting 99.9% uptime for critical applications. 

This project description emphasises role in leveraging AWS EC2, Kubernetes, and Argo CD to optimise application deployment and management processes effectively.

---------------------------------------------------------------

Using Kind - Kubernetes in Docker

**step to step guide**

- Update and install docker

```
sudo apt-get update

sudo apt-get install docker.io -y

sudo usrmod -aG docker $USER && newgrp docker
```

NOTE: Use kind-cluster doc -from k8s-kind-voting-app repo

- create install_kind.sh 

```
chmod +x install_kind.sh
./install_kind.sh
```

- create config.yml

```
kind create cluster --config=config.yml --name=<clustername>
```

- Install kubectl

```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

```
kubectl get nodes
```

- Setup ARGOCD

```
kubectl get namespace
docker ps
```

```
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl get all -n argocd

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'

kubectl port-forward -n argocd service/argocd-server 8443:443 --address=0.0.0.0 &

```
open port 8443 on EC2's SG

- Login to ArgoCD server

```
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```
- create new app
name - lower case - voting-app
project name - default
sync policy - automatic, prune resources, self heal
repository url -
branch - main
path - k8s specifications
cluster url - 
namespace - default
click on create

```
kubectl get pods
kubectl get deployments
kubectl get svc
```


```
kubectl port-forward svc/vote 5000:5000 --address=0.0.0.0 &
kubectl port-forward svc/result 5001:5001 --address=0.0.0.0 &
```
Edit inbound rules - allow port range '5000-5001' in SG

-  Installing Kubernetes Dashboard

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

```

- create dashboard-adminuser.yml

```
kubectl create namespace kubernetes-dashboard
kubectl apply -f dashboard-adminuser.yml
```

- Port-forward kubernetes dashboard

```
kubectl port-forward svc/kubernetes-dashboard -n kubernetes-dashboard 8080:443 --address=0.0.0.0 &
```
NOTE: Edit inbound rule open port, 8080 for kubernetes dashboard


- Create a token for dashboard access:

```
kubectl -n kubernetes-dashboard create token admin-user

```
NOTE: did not work for me above command, kubectl version issue
It looks like the admin-user service account has been created, but it doesn't have any associated secrets, and no token is automatically created for it. This is why you're not seeing any token under the secrets list.

To generate a token for the admin-user service account, you'll need to create it manually, as Kubernetes doesn't create a token by default for service accounts unless explicitly configured.

**Other way around:**

- create  'service-account-token.yml'

```
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  annotations:
    kubernetes.io/service-account.name: admin-user
type: kubernetes.io/service-account-token

```

```
kubectl apply -f service-account-token.yml -n kubernetes-dashboard
```

- manually retrieve the token from the secret associated with the service account.
  Find the secret associated with the service account:

  ```
  kubectl get serviceaccount admin-user -n kubernetes-dashboard -o jsonpath='{.secrets[0].name}'
  ```
- Get the token from the secret:

  ```
  kubectl get secret <secret-name> -n kubernetes-dashboard -o jsonpath='{.data.token}' | base64 --decode
  ```

NOTE: copy token and paste on kubernetes dashboard's token 


- Access on browser

```
https://public-ip:8443 (argocd server)
https://public-ip:8080 (kubernetes dashboard)
http://public-ip:5000 (vote app)
http://public-ip:5001 (result app)

```


Happy Learning !!


*Troubleshoot steps related to token* 

- Verify the service account exists

```
kubectl get serviceaccount admin-user -n kubernetes-dashboard
```

- Check for secrets in the namespace

```
kubectl get secrets -n kubernetes-dashboard
```

- Verify the ClusterRoleBinding:

```
kubectl get clusterrolebinding admin-user
```




