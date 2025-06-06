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
kubectl apply -f vs.yaml
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

#### Testing with curl

To test HTTP access:
```bash
curl -v http://nonprod.domain.net/get
```

To test HTTPS access with proper TLS certificate validation:
```bash
# Using the CA certificate for validation (recommended approach)
curl -v --cacert /path/to/ca.pem https://nonprod.domain.net/get

# To specify SNI (Server Name Indication) explicitly
curl -v --cacert /path/to/ca.pem --resolve nonprod.domain.net:443:<INGRESS_IP> https://nonprod.domain.net/get
```

#### Testing with mTLS (Mutual TLS)

For mTLS testing, you need both server validation and client authentication:

```bash
# Full mTLS with client certificate authentication
curl -v --cacert /path/to/ca.pem \
  --cert /path/to/client.pem \
  --key /path/to/client-key.pem \
  https://nonprod.domain.net/get
  
# If your client certificate has a passphrase
curl -v --cacert /path/to/ca.pem \
  --cert /path/to/client.pem \
  --key /path/to/client-key.pem \
  --pass mypassphrase \
  https://nonprod.domain.net/get
```

The httpbin service provides different endpoints for testing:
```bash
# Test with JSON data in the response
curl --cacert /path/to/ca.pem https://nonprod.domain.net/json

# Test with headers in the response  
curl --cacert /path/to/ca.pem https://nonprod.domain.net/headers

# Test with request information echo
curl --cacert /path/to/ca.pem -X POST -d "test data" https://nonprod.domain.net/anything
```

#### Verifying TLS Connection Details

To verify the TLS certificate details:
```bash
echo | openssl s_client -connect nonprod.domain.net:443 -servername nonprod.domain.net -CAfile /path/to/ca.pem
```

#### Setting Up Client Certificates for mTLS

If you need to create client certificates for testing mTLS:

1. You can use the sample certificates in the Istio distribution:
```bash
# Copy sample certificates from Istio
cp -r /mydata/codes/2025/minikube-nova-tls/istio-1.24.6/samples/certs /mydata/codes/2025/minikube-nova-tls/client-certs
```

2. Or generate new client certificates using the Istio script:
```bash
cd /mydata/codes/2025/minikube-nova-tls/istio-1.24.6/samples/certs
./generate-workload.sh client
```

This will create client certificates that can be used for mTLS testing.
