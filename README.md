# davtalks_gitops_project
Contient les fichiers d'architecture gcloud du projet et les liens vers les repos back et front

# Deploiement sur Google Cloud : 
## 1. Configurer notre projet sur gcs
gcloud projects list
gcloud config set project PROJECT_ID

## 2. Startup Kubernetes Google Cloud
On commence par créer le cluster Kubernetes pour chaque environnement : 
gcloud services enable container.googleapis.com cloudbuild.googleapis.com sourcerepo.googleapis.com containeranalysis.googleapis.com
gcloud container clusters create davtalks-gitops-cluster-production --num-nodes 3 --zone  northamerica-northeast1-a --enable-autoscaling --min-nodes 1 --max-nodes 4
gcloud container clusters create davtalks-gitops-cluster-staging --num-nodes 1 --zone  northamerica-northeast1-a --machine-type=g1-small

## 3. Creation des deploiements et services
kubectl apply -f deployment.back.prod.yaml
kubectl apply -f deployment.front.prod.yaml

kubectl apply -f deployment.back.staging.yaml
kubectl apply -f deployment.front.staging.yaml

## 3. Creation des redirections dns sur les ip de services
Copier les ip publiques proposées pour les 2 services sur le fournisseur de DNS
Ne pas oublier de maj le fichier de conf du front pour pointer sur la bonne url

## 4. Création des declencheurs cloud build
gcloud projects describe davtalks-gitops
Recuperer le projectNumber et l'inserer dans la commande : 
gcloud projects add-iam-policy-binding PROJECT_NUMBER --member=serviceAccount:PROJECT_NUMBER@cloudbuild.gserviceaccount.com --role=roles/container.developer

Connecter master au cluster production et develop au cluster staging

#Deploiement manuel

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