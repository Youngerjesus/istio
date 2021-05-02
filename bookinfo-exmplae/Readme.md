# Getting started 

## Deploy the sample application

##### Deploy the Bookinfo sample application:      
```
$ kubectl apply -f bookinfo.yaml
```

##### The application will start. 
````
$ kubectl get services
````

and 

````
$ kubectl get pods
```` 

##### Verify everything is working correctly up to this point. 
````
$ kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>
````

***

## Open the application to outside traffic

The Bookinfo application is deployed but not accessible from the outside. To make it accessible, you need to create an Istio Ingress Gateway, which maps a path to a route at the edge of your mesh.


1. Associate this application with the Istio gateway:

```
$ kubectl apply -f bookinfo-gateway.yaml
```

2. Ensure that there are no issues with the configuration:
````yaml
$ istioctl analyze
No validation issues found when analyzing namespace: default.
````   

*** 

## Determining the ingress IP and ports

Follow these instructions to set the INGRESS_HOST and INGRESS_PORT variables for accessing the gateway. Use the tabs to choose the instructions for your chosen platform:

1. Set the ingress ports:
   
````
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
````

2. Set GATEWAY_URL:
   
````yaml
$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
````

3. Ensure an IP address and port were successfully assigned to the environment variable:
   
```
$ echo "$GATEWAY_URL"
```


