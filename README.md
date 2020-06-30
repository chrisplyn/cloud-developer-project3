# Refactor Udagram App into Microservices and Deploy

# Running locally
## Install docker-compose
    curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > docker-compose
    chmod +x docker-compose
    sudo mv docker-compose /usr/local/bin
## Build and run docker containers locally
    docker-compose -f udagram-deployment/docker/docker-compose-build.yaml build --parallel
    docker-compose -f udagram-deployment/docker/docker-compose-build.yaml push
    docker-compose up

## Verify application
Visit `localhost:8100`

# Deploy to AWS EKS cluster
## Install kubectl
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
    sudo rm /usr/local/bin/docker-compose
## Install AWS CLI
    curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
    unzip awscli-bundle.zip
    sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
## Configure AWS EKS cluster
    aws eks --region ${AWS_REGION} update-kubeconfig --name ${AWS_EKS_CLUSTER}

`AWS_REGION` and `AWS_EKS_CLUSTER` are environment variables, also `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` must be set.

## Create pods and services on EKS cluster
    kubectl apply -f udagram-deployment/k8s/aws-secret.yaml
    kubectl apply -f udagram-deployment/k8s/env-secret.yaml
    kubectl apply -f udagram-deployment/k8s/backend-feed-deployment.yaml
    kubectl apply -f udagram-deployment/k8s/backend-user-deployment.yaml
    kubectl apply -f udagram-deployment/k8s/reverseproxy-deployment.yaml
    kubectl apply -f udagram-deployment/k8s/frontend-deployment.yaml
    kubectl apply -f udagram-deployment/k8s/backend-feed-service.yaml
    kubectl apply -f udagram-deployment/k8s/backend-user-service.yaml
    kubectl apply -f udagram-deployment/k8s/reverseproxy-service.yaml
    kubectl apply -f udagram-deployment/k8s/frontend-service.yaml

## Verify deployment
    kubectl get pods
    kubectl get svc

## Verify application
    kubectl port-forward service/frontend 8100:8100
    kubectl port-forward service/frontend 8080:8080
Then visit `localhost:8100`.