# Tech-case

Deploy two application to securley expose  
/app1   Grafana Dashboard  
/app2   Ghost Blog  

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for testing purposes. 

## Design Diagram
![alt text](https://github.com/Eslamanwar/tech-case/blob/master/design/gke.jpg?raw=true)
  
### Prerequisites

- you need to install kubectl on your local machine
- you need to install GKE Cluster from Google console create k8s cluster
- connect to k8s cluster
```
gcloud auth login
gcloud container clusters get-credentials standard-cluster-1 --zone {ZONE} --project {PROJECT-ID}
```


### Installing

A step by step instructions that tell you how to get a testing env running

Install Helm client

```
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > install-helm.sh
chmod u+x install-helm.sh
./install-helm.sh
```

Install Helm server side (Tiller)

```
kubectl -n kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller
```

## Install nginx-ingress controller
```
helm install stable/nginx-ingress
```
- get nginx-ingress-controller-IP
```
kubectl get svc |grep controller
```
- please update your hosts file with (nginx-ingress-controller-IP eslam.local)

## Install cert-manager with helm
(certificate management controller deals with Let's Encrypt)
```
helm repo add jetstack https://charts.jetstack.io
kubectl create namespace cert-manager
kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true
kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/deploy/manifests/00-crds.yaml
```

```
(cat <<EOF
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
 labels:
   name: letsencrypt-prod
 name: letsencrypt-prod
spec:
 acme:
   email: eslam.anwar96@gmail.com
   http01: {}
   privateKeySecretRef:
     name: letsencrypt-prod
   server: https://acme-v02.api.letsencrypt.org/directory
EOF
) | kubectl apply -f -;
```


```
helm install --name cert-manager --namespace cert-manager --version v0.8.1 jetstack/cert-manager \
   --set ingressShim.defaultIssuerName=letsencrypt-prod \
   --set ingressShim.defaultIssuerKind=ClusterIssuer
```

## Install Grafana
```
## Deploy the Grafana chart using the modified values file
helm install --name test-grafana  stable/grafana -f grafana/grafana-values.yaml
```

## Ghost Blog
```
## Deploy the Ghost chart using the modified values file
helm install --name test stable/ghost  -f ghost/ghost-values.yaml
```



## install prometheus
```
helm install --name test-prom  stable/prometheus-operator -f prometheus/prom-values.yaml
```


## Scrape Mariadb Metrics
```
## please update the mariadb root password in prometheus/ghost-mariadb-exporter.yaml 
## kubectl get secret --namespace default test-mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 --decode

## apply mariadb exporter deployment/service/servicemonitor 
kubectl create -f prometheus/ghost-mariadb-exporter.yaml


## please restart prometheus pod in order the servicemonitor reload
kubectl delete pod prometheus-test-prom-prometheus-opera-prometheus-0

## check by port-forward promethues service and go to targets
kubectl port-forward svc/test-prom-prometheus-opera-prometheus 9090:9090
```
![alt text](https://github.com/Eslamanwar/tech-case/blob/master/design/Screenshot%20from%202019-10-28%2001-08-52.png?raw=true)
![alt text](https://github.com/Eslamanwar/tech-case/blob/master/design/Screenshot%20from%202019-10-28%2001-09-02.png?raw=true)
![alt text](https://github.com/Eslamanwar/tech-case/blob/master/design/Screenshot%20from%202019-10-28%2000-59-40.png?raw=true)
