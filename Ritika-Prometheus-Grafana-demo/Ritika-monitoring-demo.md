### Utilizing Kind cluster - Monitoring with Prometheus and visualizing with Grafana

step to step guide

- Update and install docker

    ```
    sudo apt-get update

    sudo apt-get install docker.io -y
    docker ps
    sudo usrmod -aG docker $USER && newgrp docker
    ```

    ```
    git clone repo
    ```

    cd kind-cluster/
   
- Install kind
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

cd k8s-specifications/

- deploy all manifest files from this       current   directory

    ```
    kubectl apply -f .
    kubectl get all
    ```

- Port forward both apps vote and result:

    ```
    kubectl port-forward service/vote 5000:5000 --address=0.0.0.0 &
    kubectl port-forward service/result 5001:5001 --address=0.0.0.0 &
    ```
Edit inbound rules and add port range 5000-5001 in SG


- Install helm

    ```
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
    chmod +x get_helm.sh
    ./get_helm.sh
    ```

    ```
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

    helm repo list 

    helm repo add stable https://charts.helm.sh/stable

    helm repo update

    kubectl create namespace monitoring

    helm install kind-prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --set prometheus.service.nodePort=30000 --set prometheus.service.type=NodePort --set grafana.service.nodePort=31000 --set grafana.service.type=NodePort --set alertmanager.service.nodePort=32000 --set alertmanager.service.type=NodePort --set prometheus-node-exporter.service.nodePort=32001 --set prometheus-node-exporter.service.type=NodePort

    kubectl get pods -n monitoring

    kubectl get namespace
    ```

- Port-forward Prometheus and Grafana

    ```
    kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &

    kubectl port-forward svc/kind-prometheus-grafana -n monitoring 3000:80 --address=0.0.0.0 &
    ```
No need to setup in EKS - port forwarding

Edit inbound rule and add open ports 9090 and 3000

- Access prometheus UI server:

    ```
    http://public-ip:9090/metrics
    ```

- Access Grafana UI:

    ```
    http://public-ip:3000

    ```
Grafana Credentials: admin, prom-operator
add data sources and create dashboards

Add users --> administration --> users and access

you can change role - give 'viewer' access

Add connections --> already added


prometheus UI - status --> targets go to - graph - promql

for example promql:
    ```
        sum (rate (container_cpu_usage_seconds_total{namespace="default"}[1m])) / sum (machine_cpu_cores) * 100
    ```



