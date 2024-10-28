This Assignment hosts our first containerized application in the locally simulated single-node K8s cluster.
Use terraform code https://github.com/billzhaohongwei/clo835_fall2024_assignment2_terraform to provision resources
Use Github action in https://github.com/billzhaohongwei/clo835_fall2024_assignment1_app to push images to Amazon ECR

# Copy and paste manifests to the host machine.
scp -i a2 mysql-pod.yaml mysql-service.yaml frontend-pod.yaml frontend-service.yaml mysql-replicaset.yaml frontend-replicaset.yaml frontend-deployment.yaml mysql-deployment.yaml <instance IP>:/tmp/ 
ssh -i a2 <instance IP>

# set up credentials
aws configure
# set up aws_session_token
vim ~/.aws/credentials
export ECR=444868690040.dkr.ecr.us-east-1.amazonaws.com/clo835-assignment1-mysql-repo
aws ecr get-login-password --region us-east-1 | docker login -u AWS ${ECR} --password-stdin

# check K8s components status
kubectl get nodes
kubectl get pods -n kube-system
kubectl cluster-info

alias k=kubectl
k create ns assignment2

# Deploy MySQL and web applications as pods in their respective namespaces.
k apply -f /tmp/mysql-pod.yaml -n assignment2
k apply -f /tmp/mysql-service.yaml -n assignment2
k apply -f /tmp/frontend-pod.yaml -n assignment2
k apply -f /tmp/frontend-service.yaml -n assignment2

kubectl logs frontend -n assignment2 -f
# Use a new terminal window to ssh into EC2 instance and check connection to app

# Deploy ReplicaSets of the web application with 3 replicas using ReplicaSet manifest. Use the “app:employees” label.
# Use the “app:mysql” label to create ReplicaSets for MySQL application.
k apply -f /tmp/mysql-replicaset.yaml -n assignment2
k apply -f /tmp/frontend-replicaset.yaml -n assignment2
k get all -n assignment2

# Create deployments of MySQL and web applications using deployment manifests. 
k apply -f /tmp/mysql-deployment.yaml -n assignment2
k apply -f /tmp/frontend-deployment.yaml -n assignment2

# edit image version of frontend-deployment.yaml
k apply -f /tmp/frontend-deployment.yaml -n assignment2

# Monitor the rollout status and check image version in new pods
k rollout status deployment.apps/frontend -n assignment2
k describe pod frontend-xxxxxxxxx -n assignment2
