## Getting started 

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



