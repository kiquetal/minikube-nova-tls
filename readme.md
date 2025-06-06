#### Cluster to initialize testing

minikube -p nova start

#### Installing istio

helm install istio-base istio/base -n istio-system --create-namespace
helm install istiod istio/istiod -n istio-system --wait
helm install istio-ingress istio/gateway -n istio-system

#### Deploying the Demo Application

1. Create the necessary namespaces:
```bash
kubectl create namespace istio-system --dry-run=client -o yaml | kubectl apply -f -
kubectl create namespace apps --dry-run=client -o yaml | kubectl apply -f -
```

2. Enable Istio injection for the apps namespace:
```bash
kubectl label namespace apps istio-injection=enabled
```

3. Create the TLS secret (replace with your actual certificate and key):
```bash
# First, prepare your actual certificate and key files
# Then create the secret with:
kubectl apply -f secret.yaml
```

4. Deploy the application:
```bash
kubectl apply -f deployment.yaml
```

5. Create the Gateway and VirtualService:
```bash
kubectl apply -f gateway.yaml
```

6. Access the application:

Add the following entry to your hosts file:
```
<INGRESS_IP> nonprod.domain.net
```

Where `<INGRESS_IP>` is the external IP of your Istio ingress gateway, which you can get with:
```bash
kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

For Minikube, you may need to use:
```bash
minikube -p nova tunnel
```
In another terminal to expose the LoadBalancer services.

Then you can access the application at:
```
http://nonprod.domain.net/
https://nonprod.domain.net/
```
