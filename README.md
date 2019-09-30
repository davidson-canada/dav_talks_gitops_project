# davtalks_gitops_project
Contient les fichiers d'architecture gcloud du projet et les liens vers les repos back et front

# Deploiement sur Google Cloud : 
## Conf Google Cloud
gcloud services enable container.googleapis.com cloudbuild.googleapis.com sourcerepo.googleapis.com containeranalysis.googleapis.com
gcloud container clusters create davtalks-gitops-cluster-production --num-nodes 3 --zone  northamerica-northeast1-a --machine-type=g1-small --enable-autoscaling --min-nodes 1 --max-nodes 4

## Deploy front 1st time : 
docker build -t gcr.io/davtalks-gitops/davtalks-gitops-front-production:v1 .
docker push gcr.io/davtalks-gitops/davtalks-gitops-front-production:v1
kubectl run davtalks-gitops-front-production --image=gcr.io/davtalks-gitops/davtalks-gitops-front-production:v1 --port 80
kubectl expose deployment davtalks-gitops-front-production --type=LoadBalancer --port 80 --target-port 80 // pour creer le load balancer

## Update front
docker build -t gcr.io/davtalks-gitops/davtalks-gitops-front-production:v2 .
docker push gcr.io/davtalks-gitops/davtalks-gitops-front-production:v2
kubectl set image deployment/davtalks-gitops-front-production davtalks-gitops-front-production=gcr.io/davtalks-gitops/davtalks-gitops-front-production:v2 // pour reassigner un pod au load balancer

## Deploy back 1st time : 
docker build -t gcr.io/davtalks-gitops/davtalks-gitops-back-production:v1 .
docker push gcr.io/davtalks-gitops/davtalks-gitops-back-production:v1
kubectl run davtalks-gitops-back-production --image=gcr.io/davtalks-gitops/davtalks-gitops-back-production:v1 --port 8080 --env="NODE_ENV=production"
kubectl expose deployment davtalks-gitops-back-production --type=LoadBalancer --port 80 --target-port 8080 // pour creer le load balancer

## Update back
docker build -t gcr.io/davtalks-gitops/davtalks-gitops-back-production:v2 .
docker push gcr.io/davtalks-gitops/davtalks-gitops-back-production:v2
kubectl set image deployment/davtalks-gitops-back-production davtalks-gitops-back-production=gcr.io/davtalks-gitops/davtalks-gitops-back-production:v2 // pour reassigner un pod au load balancer