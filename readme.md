#### Cluster to initialize testing

minikube -p nova start



#### Installing istio




helm install istio-base istio/base -n istio-system --create-namespace
helm install istiod istio/istiod -n istio-system --wait
helm install istio-ingress istio/gateway -n istio-system

