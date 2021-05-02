# Istio 

*** 

## Gateway 

Gateway는 외부로부터 트래픽을 받는 최앞단으로 여기서 지정할 값은 트래픽을 받을 호스트 명과

포트, 프로토콜을 정의한다. 

예제는 다음과 같다. 

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
        - "ext-host.example.com"
```
- 여기서는 ext-host.example.com 으로 들어오는 요청 중 HTTP 프로토콜을 쓴다면 이 게이트웨이가 요청을 받겠다 라는 의미다. 
 
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
- hosts는 클라이언트에서 요청한 request에 대해서 어떤 host를 호출할 것인지 결정한다. (host는 service 라고 생각하면 된다.)
  여기서는 hosts: "*" 로 지정했기 떄문에 virtualservice가 라우팅 할 모든 요청에 대해서는 gateway: bookinfo-gateway 와 연관되어 있다.
  이 bookinfo-gateway로 들어온 요청에 대해 라우팅 처리로 서비스를 호출한다. 
- gateways는 어떤 게이트웨이에서 들어온 요청을 트래픽 처리할 것인지 지정해준다. 여기서는
  gateways: bookinfo-gateway로 지정했으므로 bookinfo-gateway에서 들어온 요청에 대해 라우팅
  처리를 해준다. 
- http는 URL 별로 라우팅을 하도록 정의했다. http: match: 부분을 보면 해당 url에 대해 정확히
  일치하는 가를 따질려면 exact를 쓰면 되고 해당 url과 일부만 맞으면 된다면 prefix를 쓰면 된다. 
- route: destination: host:는 http: match: 와 맞는 url로 요청이 오면 호출할 서비스를 지정한 것이다. 여기서는 productpage
  서비스를 호출한다. {namespace}.svc.cluster.local 서비스로 포워딩된다. namespace를 지정하지 않는다면 기본적으로는 default가 된다. 
  
virtualService의 다음 예제는 gateway와 연동되지 않고 service와 연동되서 service를 호출할 때 라우팅 처리를 정의한 것이다.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews-route
spec:
  hosts:
  - reviews.prod.svc.cluster.local
  http:
  - name: "reviews-v2-routes"
    match:
    - uri:
        prefix: "/wpcatalog"
    - uri:
        prefix: "/consumercatalog"
    rewrite:
      uri: "/newcatalog"
    route:
    - destination:
        host: reviews.prod.svc.cluster.local
        subset: v2
  - name: "reviews-v1-route"
    route:
    - destination:
        host: reviews.prod.svc.cluster.local
        subset: v1
```
 - 이 예제는 reviews.prod.svc.cluster.local 서비스를 호출할 때 동작하는 virtualservice이다. 
 - 요청 uri이 /wpcatalog 나 /consumercatalog 로 시작한다면 uri를 /newcatalog로 바꾼다. 
 - 그 후 subset을 v2로 (service version을 말한다) 해서 reviews.prod.svc.cluster.local이 서비스에 v2 버전으로 전달한다.
   - subset 에 대해서 좀 더 설명하자면 apiVersion을 위해 subset을 사용할 수 있고 카나리 배포환경에서도 사용할 수 있다. 
 - 요청 uri이 /wpcatealog 나 /consumercatalog 로 시작하지 않다면 reviews.prod.svc.cluster.local 서비스에 v1 버전으로 전달한다. 

마지막으로 다른 예제를 보자. 

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-productpage-rule
  namespace: istio-system
spec:
  hosts:
  - productpage.prod.svc.cluster.local # ignores rule namespace
  http:
  - timeout: 5s
    route:
    - destination:
        host: productpage.prod.svc.cluster.local
```
 - 이 예제는 별도의 라우팅 없이 productpage.prod.svc.cluster.local을 그대로 호출하도록 정의했다. 
 - 다만 특별한 점은 timeout: 5s 로 줘서 해당 서비스가 5초동안 응답이 없다며ㅑㄴ timeout 처리를 하도록 했다. 
 
VirtualService의 개념을 이해할때 헷갈릴 수 있는데 다음 두 가지를 기억하면 이해에 도움이 된다.

- 쿠버네티스의 Ingress와 같이 들어오는 트래픽에 대해 URI나 Request Header 정보를 바탕으로 라우팅을 할 수 있다. 
- Gateway나 service를 통해서 들어오는 트래픽에 대해서 service로 라우팅을 시킬 수 있다.  


Reference: [Istio Traffic management](https://bcho.tistory.com/1367)
***

## Destination Rule 

***

## Policy
