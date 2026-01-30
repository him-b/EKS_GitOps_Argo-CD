Project on Go lang,It uses the net/http package to serve HTTP requests.

<img width="656" height="136" alt="image" src="https://github.com/user-attachments/assets/3a4083d3-c31e-4d82-8075-9f4dfe0b44ce" />


Application looks like below:
<img width="940" height="469" alt="image" src="https://github.com/user-attachments/assets/a3c51891-073e-415e-b6c6-b6ecc5f47bb5" />

Application is deployed on an EKS cluster usign 2 worker nodes.

Tools to install before getting started:
Docker
helm
kubectl
eksctl- to create cluster have proper role attached, for more use  link: https://docs.aws.amazon.com/eks/latest/eksctl/installation.html
aws cli

Check locally if application is working by using http://localhost:8080/courses (build docker image first) and run locally(app is exposed to 8080 port)

Configure aws using aws configure, add access key and secret key of iam user.

run on terminal:  eksctl create cluster -f cluster.yaml(inside eks folder)
                  eksctl create nodegroup -f fix.yaml(to create worker node)
check if nodes are up:
kubectl get nodes -o wide

install nginx controller exposed its service as load balancer:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml

now you will see NLB attached to ingress resource:
kubectl get ing
DNS mapping to be done as hosts is go-app.local
nslookup <NLB IP>: gets an address, copy this address and paste in hosts along with host name as go-app.local
change it in sudo vim /etc/hosts

check if application is running by hitting: go-app.local/courses

Install helm:
helm create go-web-app-chart

run : helm install go-web-app ./go-web-app-chart

CI part: using github actions
build, test, login to docker hub and push the image with tag and update it in values.yaml with tag as  gitrun_id

CD: argo cd is installed to watch helm path in github, changes found in values.yaml will be synchronised with kubernetes cluster.Pods will run the application with healthy status.

installation for Argo cd:
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Expose argocd as loadbalancer or nodeport type to see Argo cd ui:
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

get load balancer service IP for argo cd:
kubectl get svc argocd-server -n argocd

Fetch password using:
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

login as admin and password

click on new apps, configure the path of github and helm path, add values.yaml and other fields to set the repo.


<img width="940" height="475" alt="image" src="https://github.com/user-attachments/assets/b3bbed1b-75a9-4f38-a520-deaa9e034e22" />

<img width="940" height="475" alt="image" src="https://github.com/user-attachments/assets/d0caea88-2340-462d-b454-e8c3ce39ee7a" />



