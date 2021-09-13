# ArgoRolloutsDemo
My Argo Rollouts demo for my class

# Set proper project
```
gcloud config set project nicolas-notepad-dev-821
```

# Script variables. Edit to you own needs
```
export clustername=argo-demo-cluster
export region=us-central1
export network=vpc-nicolas-argos-demo
export subnetwork=subnet-nicolas-argos-demo
export clustersecondaryrange=pods
export servicessecondaryrange=services
```
# Set VPC
```
gcloud compute networks create vpc-nicolas-argos-demo --project=nicolas-notepad-dev-821 --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional
gcloud compute networks subnets create subnet-nicolas-argos-demo --project=nicolas-notepad-dev-821 --range=10.128.1.0/26 --network=$network --region=$region --secondary-range=pods=172.0.0.0/18,services=172.10.0.0/21 --enable-private-ip-google-access
```
# GKE cluster setup
```
gcloud container clusters create $clustername --region $region --enable-ip-alias --enable-network-policy --num-nodes 1 --machine-type "e2-standard-2" --release-channel stable --network $network --subnetwork $subnetwork --cluster-secondary-range-name $clustersecondaryrange --services-secondary-range-name $servicessecondaryrange 
gcloud container clusters get-credentials $clustername --region $region
```
# Install istioctl
```
curl -L https://istio.io/downloadIstio | sh - 
cd istio-1.11.2
export PATH=$PWD/bin:$PATH
cd ..
```
# Create static IP
```
gcloud compute addresses create myip
export istioip=$(gcloud compute addresses list --filter="name='myip'" --format="csv[no-heading](address)")
```

# Install Istio
```
echo $istioip
yes | istioctl install --set profile=demo --set values.gateways.istio-ingressgateway.loadBalancerIP=$istioip
kubectl get svc istio-ingressgateway -n istio-system
```

# Install the kubectl plugin
```
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x ./kubectl-argo-rollouts-linux-amd64
sudo mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
kubectl argo rollouts version
```

# Grant yourself the ability to create new cluster roles
```
kubectl create clusterrolebinding nicolas-cluster-admin-binding --clusterrole=cluster-admin --user=nicolas.hostert@mail.mcgill.ca
```

# Deploy Prometheus
```
cd ArgoRolloutsDemo
kubectl apply -f prometheus.yaml
```

# Deploy Argo
```
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

# Deploy the demo
```
cd ~/argo-demo/demo
kubectl kustomize . | kubectl apply -f -
```

# Open the application

http://nicolas.cloud-montreal.ca/

# Deploy an image and play
```
kubectl argo rollouts set image istio-rollout istio-rollout=argoproj/rollouts-demo:red
kubectl argo rollouts get rollout istio-rollout -w
```

You can also deploy other images to test, and play with the error % slider
```
kubectl argo rollouts set image istio-rollout istio-rollout=argoproj/rollouts-demo:yellow
kubectl argo rollouts set image istio-rollout istio-rollout=argoproj/rollouts-demo:red
kubectl argo rollouts set image istio-rollout istio-rollout=argoproj/rollouts-demo:green
```
