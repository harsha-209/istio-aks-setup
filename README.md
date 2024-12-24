### we can directly enable the istio in aks cluster and it promote the option is it exteral or internal

az aks show --resource-group ${RESOURCE_GROUP} --name ${CLUSTER}  --query 'serviceMeshProfile.istio.revisions'

apply istio on particulare namespace from above output of istio version

kubectl label namespace default istio.io/rev=asm-X-Y

Deploy your app and it's shows 2/2, we confirm that istion sidecarae deployed

now let's deploy a gateway to map external ip of istio
kubectl get svc -n aks-istio-ingress ## it give us the external ip 

**now deploy gateway map with the above service name**

apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: notify-gateway-external
spec:
  selector:
    istio: aks-istio-ingressgateway-external
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
**if you using custom dns**

apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: notify-gateway-external
spec:
  selector:
    istio: aks-istio-ingressgateway-external
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*.harshad.shop"
   
   ** map your application with gateway using virtual service without dns or with dns**

    ---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: notify-vs-external
spec:
  hosts:
  - "*"
  gateways:
  - notify-gateway-external
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: core-app
        port:
          number: 80

**dns with virtual service**

---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: notify-vs-external
spec:
  hosts:
  - "istio-backend.harshad.shop"
  gateways:
  - notify-gateway-external
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: core-app
        port:
          number: 80



