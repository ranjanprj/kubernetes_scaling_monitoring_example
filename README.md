This project is a simple demonstration of how to use Kubernetes Horizontal Autoscaler for Pods
and monitor nodes resources using Grafana and Prometheus.

Tested on Ubuntu 18.04
===============


# 1.Docker for Ubuntu.(Optional)
    Only if you wish to create your own docker container repository, otherwise you can use 
    ranjanprj/helloautoscaler

### 1.1 Update repositories
sudo apt-get update

### 1.2 Install docker
- sudo apt-get install docker.io
- Setup permissions to use Docker without sudo
    - sudo groupadd docker
    - sudo usermod -aG docker $USER
    - sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
    - sudo chmod g+rwx "/home/$USER/.docker" -R

### 1.3 Create Docker build
- clone this project and cd into it, then change 
Dockerfile and src folder with your favorite programming language.
- `docker build . -t helloautoscaler`
- Get the ID from running `docker images` and tag     your image
- `docker tag YOUR_DOCKER_IMAGE_ID YOUR_DOCKERHUB    USERNAME/helloautoscaler:latest`
- Login to your docker hub account with `docker      login`
- and then `docker push YOUR_DOCKERHUB_USERNAME/     helloautoscaler:latest`


# 2. Minikube

    - Minikube helps you run Kubernetes experiments on your laptop. Ofcourse not for production use.



### 2.1. Install VirtualBox

     Install VirtualBox( Kubernetes Default Driver)
- `sudo apt install -y virtualbox virtualbox-ext-pack`

### 2.2. Install Minikube and Kubectl

    Download latest minikube and make it executable
- `curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube`

-   Copy minikube to bin for direct invocation


- `sudo cp minikube /usr/local/bin && rm minikube`

-   Install Kubectl

-   `curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list`
- `sudo apt update`
- `sudo apt install kubectl`

- Setup permissions to minikube and kubectl to be used without sudo
    -   sudo chown -R $USER $HOME/.kube
    -   sudo chgrp -R $USER $HOME/.kube
    -   sudo chown -R $USER $HOME/.minikube
    -   sudo chgrp -R $USER $HOME/.minikube


# 3. Steps to run the sample HPA after installing required tools.


- Setup Minikube

    - Run `minikube start`
    - Enable metrics-server by `minikube addons enable metrics-server`      

        
- Git Clone this project
    - cd to this project directory and run following command ,  
    `kubectl apply -f autoscaler`
    - on new command tab and run `minikube dashboard` to open Kubernetes dashboard
    - Run `minikube service helloautoscaler` to check the app is working in the browser.
    - To see HPA kick in open terminal(T1) with following command `watch -n 1 kubectl get pods`
    - Install Apache Bench `sudo apt install -y apache2-utils`
    - In a new terminal run and flood the pod with requests every 10 seconds
    ` watch -n 10 ab -c 1000 -n  100000000000 -t 1000  $(minikube service helloautoscaler --url)/
`
`
    - Monitor first terminal(T1) to see autoscaling of pods
    - Since golang can be fast run ab command several times if you don't see autoscaling kick in


# 4. Monitoring tools
    Install Helm Charts Grafana and Prometheus charts
    
- `sudo snap install helm --classic`
- Init helm `helm init --history-max 200`
- Wait for few minutes for Tiller Pod to get ready
- Run `helm install stable/prometheus-operator --name prometheus-operator --namespace monitoring`
- Check everything is ok `kubectl get pods  -n monitoring`
- Port forward(Optional) `kubectl port-forward -n monitoring prometheus-prometheus-operator-prometheus-0 9090`
- Open http://localhost:9090 for Prometheus
- For Grafana port forward using 
`kubectl port-forward $(kubectl get  pods --selector=app=grafana -n  monitoring --output=jsonpath="{.items..metadata.name}") -n monitoring  3000`

-  Open http://localhost:3000 for Grafana
- At top-left select Home - > Kubernetes/Nodes to see all the metrics of the Node including network traffic

 - To purge all helm charts
  `helm ls --all --short | xargs -L1 helm delete --purge`
