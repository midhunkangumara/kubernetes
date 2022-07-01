# kubernetes
## cluster setup: 
                https://www.cloudsigma.com/how-to-install-and-use-kubernetes-on-ubuntu-20-04/
                https://www.cloudsigma.com/how-to-install-operate-docker-on-ubuntu-in-the-public-cloud/



## web-mongo (demo project):     
            https://youtu.be/s_o8dwzRlu4


## metric server :  
           https://github.com/kubernetes-sigs/metrics-server/releases


## error solving:   
          https://dev.to/docker/enable-kubernetes-metrics-server-on-docker-desktop-5434

## horizondal scaling:  
          https://appychip.com/how-to-autoscale-pods-in-kubernetes-using-hpa-horizontal-pod-autoscaler-minikube-demo/



### COMMANDS  :       
          https://phoenixnap.com/kb/kubectl-commands-cheat-sheet

  
### INGRESS CONTROLLER :    
          https://docs.k0sproject.io/v1.22.5+k0s.1/examples/nginx-ingress/
          
          https://www.youtube.com/watch?v=-wHRGFRnboY&t=1555s
          
  Install NGINX using NodePort#

Installing NGINX using NodePort is the most simple example for Ingress Controller as we can avoid the load balancer dependency. NodePort is used for exposing the NGINX Ingress to the external network.

    Install NGINX Ingress Controller (using the official manifests of version v1.0.0 by the ingress-nginx project, supports Kubernetes versions >= v1.19)

    $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/baremetal/deploy.yaml

Check that the Ingress controller pods have started

    $ kubectl get pods -n ingress-nginx

Check that you can see the NodePort service

     $ kubectl get services -n ingress-nginx

From version v1.0.0 of the Ingress-NGINX Controller, a ingressclass object is required.

In the default installation, an ingressclass object named nginx has already been created.

     $ kubectl -n ingress-nginx get ingressclasses
     
NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       162m

If this is only instance of the Ingresss-NGINX controller, you should add the annotation ingressclass.kubernetes.io/is-default-class in your ingress class:

      $ kubectl -n ingress-nginx annotate ingressclasses nginx ingressclass.kubernetes.io/is-default-class="true"

Try connecting the Ingress controller using the NodePort from the previous step (in the range of 30000-32767)

      $ curl <worker-external-ip>:<node-port>

If you don't yet have any backend service configured, you should see "404 Not Found" from nginx. This is ok for now. If you see a response from nginx, the Ingress Controller is running and you can reach it.

Deploy a small test application (httpd web server) to verify your Ingress controller.

Create the following YAML file and name it "simple-web-server-with-ingress.yaml":

apiVersion: v1
kind: Namespace
metadata:
  name: web
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
  namespace: web
spec:
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.48-alpine3.14
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-server-service
  namespace: web
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-server-ingress
  namespace: web
spec:
  ingressClassName: nginx
  rules:
  - host: web.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-server-service
            port:
              number: 5000

Deploy the app:

    $ kubectl apply -f simple-web-server-with-ingress.yaml
  
  remove validation(443)
  
***    $ kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission  ***


Verify that you can access your application using the NodePort from step 3.

     $ curl <worker-external-ip>:<node-port> -H 'Host: web.example.com'

    If you are successful, you should see <html><body><h1>It works!</h1></body></html>.
    
    
 #### FOR PATH SPECIFIC PROXY
 
 
                         apiVersion: v1
                 kind: Namespace
                 metadata:
                   name: web
                 ---
                 apiVersion: apps/v1
                 kind: Deployment
                 metadata:
                   name: web-server
                   namespace: web
                 spec:
                   selector:
                     matchLabels:
                       app: web
                   template:
                     metadata:
                       labels:
                         app: web
                     spec:
                       containers:
                       - name: httpd
                         image: httpd:2.4.48-alpine3.14
                         ports:
                         - containerPort: 80
                 ---
                 apiVersion: v1
                 kind: Service
                 metadata:
                   name: web-server-service
                   namespace: web
                 spec:
                   selector:
                     app: web
                   ports:
                     - protocol: TCP
                       port: 5000
                       targetPort: 80
                 ---
                 apiVersion: networking.k8s.io/v1
                 kind: Ingress
                 metadata:
                   name: web-server-ingress
                   namespace: web
                   annotations:
                     nginx.ingress.kubernetes.io/rewrite-target: /
                 
                 spec:
                   ingressClassName: nginx
                   rules:
                   - host: alen.com
                     http:
                       paths:
                       - path: /jomon
                         pathType: Prefix
                         backend:
                           service:
                             name: web-server-service
                             port:
                               number: 5000

    
    
    
    
    

## Install NGINX using LoadBalancer 

In this example you'll install NGINX Ingress controller using LoadBalancer on k0s.

    Install LoadBalancer

    There are two alternatives to install LoadBalancer on k0s. Follow the links in order to install LoadBalancer.
        MetalLB as a pure SW solution running internally in the k0s cluster
        Cloud provider's load balancer running outside of the k0s cluster

    Verify LoadBalancer

    In order to proceed you need to have a load balancer available for the Kubernetes cluster. To verify that it's available, deploy a simple load balancer service.

apiVersion: v1
kind: Service
metadata:
  name: example-load-balancer
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer

kubectl apply -f example-load-balancer.yaml

Then run the following command to see your LoadBalancer with an external IP address.

kubectl get service example-load-balancer

If the LoadBalancer is not available, you won't get an IP address for EXTERNAL-IP. Instead, it's <pending>. In this case you should go back to the previous step and check your load balancer availability.

If you are successful, you'll see a real IP address and you can proceed further.

You can delete the example-load-balancer:

kubectl delete -f example-load-balancer.yaml

Install NGINX Ingress Controller by following the steps in the previous chapter (step 1 to step 4).

Edit the NGINX Ingress Controller to use LoadBalancer instead of NodePort

kubectl edit service ingress-nginx-controller -n ingress-nginx

Find the spec.type field and change it from "NodePort" to "LoadBalancer".

Check that you can see the ingress-nginx-controller with type LoadBalancer.

kubectl get services -n ingress-nginx

Try connecting to the Ingress controller

If you used private IP addresses for MetalLB in step 2, you should run the following command from the local network. Use the IP address from the previous step, column EXTERNAL-IP.

curl <EXTERNAL-IP>

If you don't yet have any backend service configured, you should see "404 Not Found" from nginx. This is ok for now. If you see a response from nginx, the Ingress Controller is running and you can reach it using LoadBalancer.

Deploy a small test application (httpd web server) to verify your Ingress.

Create the YAML file "simple-web-server-with-ingress.yaml" as described in the previous chapter (step 6) and deploy it.

kubectl apply -f simple-web-server-with-ingress.yaml

Verify that you can access your application through the LoadBalancer and Ingress controller.

curl <worker-external-ip> -H 'Host: web.example.com'

If you are successful, you should see <html><body><h1>It works!</h1></body></html>.        
          
### PROMETHEUS IN KUBERNETES CLUSTER   :
               https://www.youtube.com/watch?v=QoDqxm7ybLc
               
### INGRESS 

                     https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html
