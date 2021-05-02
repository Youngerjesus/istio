# Istio 

*** 

***

## Virtual Service 

들어온 트래픽을 서비스로 라우팅 하는 기능이다. 목적지는 쿠버네티스의 service가 된다. 

실제로 Virtual Service의 기능은 URI 기반으로 라우팅을 하는 Ingress와 유사하다. 

예제는 다음과 같다.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```
- hosts는 클라이언트에서 요청한 request에 대해서 어떤 host를 호출할 것인지 결정한다. 
  여기서는 host: "*" 로 지정했기 떄문에 모든 서비스를 호출하는 것에 대해서 이 VirtualService가   
  라우팅 처리를 한다. 
- gateways는 어떤 게이트웨이에서 들어온 요청을 트래픽 처리할 것인지 지정해준다. 여기서는
  gateways: bookinfo-gateway로 지정했으므로 bookinfo-gateway에서 들어온 요청에 대해 라우팅
  처리를 해준다. 
- http는 URL 별로 라우팅을 하도록 정의했다. http: match: 부분을 보면 해당 url에 대해 정확히
  일치하는 가를 따질려면 exact를 쓰면 되고 해당 url과 일부만 맞으면 된다면 prefix를 쓰면 된다. 
- route: destination: 는 이 url로 요청이 오면 호출할 서비스를 지정한 것이다. 여기서는 productpage
  서비스를 호출한다. 특별한 네임스페이스를 지정하지 않는다면 {namespace}.svc.cluster.local 서비스로 포워딩된다.
  