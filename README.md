## we will install NGINX Gateway Fabric as the controller 

1. **Install the Gateway API resources**:
```bash
# With help of kustomize we install the Gateway API
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.5.1" | kubectl apply -f -
#resources:
# - https://github.com/kubernetes-sigs/gateway-api/config/crd?timeout=120&ref=v1.3.0
## Result
#customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
#customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
#customresourcedefinition.apiextensions.k8s.io/grpcroutes.gateway.networking.k8s.io created
#customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
#customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io created
```
2. **Deploy the NGINX Gateway Fabric CRDs**:
```bash
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/crds.yaml
# Result:
#customresourcedefinition.apiextensions.k8s.io/clientsettingspolicies.gateway.nginx.org created
#customresourcedefinition.apiextensions.k8s.io/nginxgateways.gateway.nginx.org created
#customresourcedefinition.apiextensions.k8s.io/nginxproxies.gateway.nginx.org created
#customresourcedefinition.apiextensions.k8s.io/observabilitypolicies.gateway.nginx.org created
#customresourcedefinition.apiextensions.k8s.io/snippetsfilters.gateway.nginx.org created
#customresourcedefinition.apiextensions.k8s.io/upstreamsettingspolicies.gateway.nginx.org created
```

3. **Deploy NGINX Gateway Fabric Class**
```bash
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/nodeport/deploy.yaml
# Result:
#namespace/nginx-gateway created
#serviceaccount/nginx-gateway created
#clusterrole.rbac.authorization.k8s.io/nginx-gateway created
#clusterrolebinding.rbac.authorization.k8s.io/nginx-gateway created
#configmap/nginx-includes-bootstrap created
#service/nginx-gateway created
#deployment.apps/nginx-gateway created
#gatewayclass.gateway.networking.k8s.io/nginx created
#nginxgateway.gateway.nginx.org/nginx-gateway-config created
```

## Verify the Deployment
```bash
kubectl get pods -n nginx-gateway
kubectl get deploy -n nginx-gateway
```

## View the nginx-gateway service
```bash
kubectl get svc -n nginx-gateway nginx-gateway -o yaml
```

## Update the nginx-gateway service to expose ports 30080 for HTTP and 30081 for HTTPS (Optional)
```bash
kubectl patch svc nginx-gateway -n nginx-gateway --type='json' -p='[
  {"op": "replace", "path": "/spec/ports/0/nodePort", "value": 30080},
  {"op": "replace", "path": "/spec/ports/1/nodePort", "value": 30081}
]'

```
## Check Gatewaye class 
```bash
kubectl get gc
```
4. **Now We Create Our Gateway That Listen On Port 80 And Accecpt All Namespaces**:

```bash
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
    - name: http
      port: 80
      protocol: HTTP
      allowedRoutes: 
        namespaces: 
          from: All
```
## To verify the succesfull deployment, run the commands below:
```bash
kubectl get gateways -n nginx-gateway
kubectl describe gateway nginx-gateway -n nginx-gateway
```
