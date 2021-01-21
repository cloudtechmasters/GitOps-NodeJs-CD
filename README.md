# Git Ops NodeJS CD

# Pre-requisites:
    - Install GIT
    - EKS Cluster
    - Install Docker
    - Install fluxctl
    - Install FluxCD or Flux Operator
# Install GIT:
    yum install git -y
# Install Docker:
    yum install docker -y
    service docker start
# Clone code from github:
    git clone https://github.com/Naresh240/GitOps-NodeJs-CD.git
    cd GitOps-NodeJs-CD
# Build Docker image for Springboot Application
    docker build -t naresh240/nodejsdeploy:latest .
# Docker login
    docker login
# Push docker image to dockerhub
    docker push naresh240/nodejsdeploy:latest
# EKS Cluster Setup:
  [EKS Cluster Setup](https://github.com/Naresh240/eks-cluster-setup/blob/main/README.md)
# Install fluxctl:
    wget https://github.com/fluxcd/flux/releases/download/1.21.0/fluxctl_linux_amd64
    mv fluxctl_linux_amd64 fluxctl
    chmod +x fluxctl
    mv fluxctl /usr/bin/
    fluxctl version
# Install FluxCD or Flux Operator:
  Create the flux namespace:
    
    kubectl create ns nodejsdeploy
  
  Then, install Flux in your cluster (replace YOURUSER with your GitHub username):
  Syntax:    
    
    export GHUSER="YOURUSER"
        fluxctl install \
        --git-user=${GHUSER} \
        --git-email=${GHUSER}@users.noreply.github.com \
        --git-url=git@github.com:${GHUSER}/flux-get-started \		--> Replace URL with our Git Repo URL
        --git-path=namespaces,workloads \				            --> Replace with folder inside Git Repo
        --namespace=flux | kubectl apply -f -
  
  Changed as:
    
    export GHUSER="naresh240"
    fluxctl install \
    --git-user=${GHUSER} \
    --git-email=${GHUSER}@users.noreply.github.com \
    --git-url=git@github.com:${GHUSER}/GitOps-NodeJs-CD \
    --git-path=deploy \
    --namespace=nodejsdeploy | kubectl apply -f -

  Wait for Flux to start:
    
    kubectl -n flux rollout status deployment/flux
  
  Check if Flux deployment is successful or not:
  
    kubectl get deploy -n nodejsdeploy
  
  ![image](https://user-images.githubusercontent.com/58024415/101141082-cb7dff00-3639-11eb-9d15-f6a1ea351391.png)
    
  Authorise Flux CD to Connect to Your Git Repository:
  
  We now need to allow the Flux CD operator to interact with the Git repository, and therefore, we need to add its public SSH key to the repo.
  
  Get the public SSH key using fluxctl:
    
    fluxctl identity --k8s-fwd-ns nodejsdeploy
    
  In order to sync your cluster state with git you need to copy the public key and create a deploy key with write access on your GitHub repository.
  Open GitHub Repo --> settings --> Deploy Keys
  Add public key inside Deploy Keys
  
  ![image](https://user-images.githubusercontent.com/58024415/101140750-632f1d80-3639-11eb-81cb-62df35a35db0.png)

  Click on Allow write access check box and then click on Add Key
  
  ![image](https://user-images.githubusercontent.com/58024415/101140439-fa47a580-3638-11eb-9905-797908298fc2.png)

  By default, Flux git pull frequency is set to 5 minutes. You can tell Flux to sync the changes immediately with:
  
    fluxctl sync --k8s-fwd-ns nodejsdeploy
   
  ![image](https://user-images.githubusercontent.com/58024415/101141524-6b3b8d00-363a-11eb-9745-8cdc10927991.png)
  
  Now check wether deployment done in cluster or not:
  
  ![image](https://user-images.githubusercontent.com/58024415/101142457-9ecae700-363b-11eb-8c5e-d14a680aa7aa.png)
