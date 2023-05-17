---
title: Configure Kubernetes for NGINX with ingress-nginx
slug: kubernetes-nginx-ingress-configuration

publish_timestamp: May 16, 2023
url: https://www.codingforentrepreneurs.com/blog/kubernetes-nginx-ingress-configuration/

---


Ingesting traffic to a Kubernetes deployment or service is made more powerful by leveraging the Ingress NGINX controller. In this blog post, I'll show you a simple and effective way to use this controller on a sample deployment and service.

### Requirements
- a Kubernetes Cluster
- Correct permissions to create a deployment, service, and and installing the Ingress NGINX controller
- [kubectl](https://kubernetes.io/docs/reference/kubectl/) installed


If you're new to Kubernetes, consider watching our [Terraforming Kubernetes on Linode](https://www.codingforentrepreneurs.com/courses/terraforming-kubernetes-on-linode/) series.


### Create a Deployment

For this, we'll use a simple deployment of a containerized Python web application ([container repo](https://hub.docker.com/r/jmitchel3/tf-python), [source code](https://github.com/codingforentrepreneurs/tf-python)).

`ops/1-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tf-python-d
spec:
    replicas: 3
    selector:
        matchLabels:
            app: tf-python-d
    template:
        metadata:
            labels:
                app: tf-python-d
        spec:
            containers:
            - name: tf-python-c
              image: jmitchel3/tf-python:latest
              ports:
                - name: tf-py-port
                  containerPort: 8080
              env:
                - name: PORT
                  value: "8080"
                - name: ENV_MESSAGE
                  value: "Hello from Kubernetes"
                - name: SECRET_MESSAGE
                  value: "Sweet blog post"
```
As we see the image `jmitchel3/tf-python:latest` is running on port `8080` with a environment variable `PORT` set to the same port. The running port is important for the next step but the environment variable is not.

### Create a Service
Kubernetes services are great because they load balance traffic by default. We can allow traffic to be external (i.e. internet-based) traffic by using a `LoadBalancer` type or we can force the traffic to be internal (i.e. our Kubernetes cluster-based) by using a `ClusterIP`.

An advantage, and potential downside, to using the external `LoadBalancer` type is that the cloud provider will provision often provision a new IP Address for each new service you create with this type. Given enough services provisioned with the type of `LoadBalancer` we might end up having far too many IP Addresses than what we really need.  For this reason, we'll use the `ClusterIP` type. 

The obvious downside to using a `ClusterIP` for a service is that it's not accessible from the internet and thus the designed web-traffic we want simply cannot get to us. This will be solved by adopting the Ingress NGINX controller later in this post.

Let's create our `ClusterIP` service:

`ops/2-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: tf-python-s
spec:
    type: ClusterIP
    ports:
        - name: http
          port: 80
          targetPort: tf-py-port
          protocol: TCP
    selector:
        app: tf-python-d
```


Now let's provision both the service and deployment:

```
kubectl apply -f ops/
```
If done correctly, `ops/1-deployment.yaml` will be first followed by `ops/2-service.yaml`. This ordering is deliberate to ensure the service is created after the deployment. If the service is created before the deployment, the service will not be able to find the deployment and thus will not be able to load balance traffic to it.



### Install the Ingress NGINX Controller

The more full-featured guide is on [the official docs](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start) but we'll use the quick-start method here by using the following command:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.1/deploy/static/provider/cloud/deploy.yaml
```

This manifest will create a number of resources in the `ingress-nginx` namespace. We can verify this by running:

```
kubectl get all -n ingress-nginx
```

The key one I want to draw your attention to is located at:

```
kubectl get service ingress-nginx-controller -n ingress-nginx
```
What you should see, depending on the cloud provider your Kubernetes cluster is running on, is an `EXTERNAL-IP` that is either `pending` or an actual IP Address. If it's `pending` then you'll need to wait a few minutes for the cloud provider to provision an IP Address for you. If it's an actual IP Address then you're ready to move on.

This is the IP Address we can use to map domain names directly to our services within our Kubernetes cluster.

### Create an Ingress
Now that we have `ingress-nginx` installed on our Kubernetes cluster and we have a valid external IP address, we can now map a domain name to that IP address.

A shortcut to get the exact IP Address is to run:

```
kubectl get service ingress-nginx-controller -n ingress-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```
With that, we'll get a response like `23.91.24.24` (not a real IP Address). This is the IP Address we can forward our domain name to. Setup an `A Record` that maps to this address such as:

- hostname: `www`
- value: `23.91.24.24`
- TTL: `1 Hour`

or 

- hostname: `` (blank)
- value: `23.91.24.24`
- TTL: `1 Hour`

This would map to the domains:

- `www.example.com`
- `example.com`

Naturally, you'll need to use a domain name that you have control over. Most of the time, I'll configure the above _before_ I create the ingress manifest so that the DNS (domain name system) has time to propagate. 

Now we can create our ingress manifest:

`ops/3-ingress.yaml`
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tf-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    # cert-manager.io/cluster-issuer: "letsencrypt"
    # nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    # nginx.ingress.kubernetes.io/ssl-passthrough: "true"
spec:
  rules:
   - host: www.example.com
     http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: tf-python-s
              port:
                name: http
   - host: example.com
     http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: tf-python-s
                port:
                  name: http
```

Now we can provision the ingress:

```
kubectl apply -f ops/
```

We can verify the ingress is configured correctly by either:

- Visit the domain name(s) you configured in the ingress in the browser
- Use `curl` to make a request to the IP Address of the ingress controller

```
export MY_NGINX_IP=$(kubectl get service ingress-nginx-controller -n ingress-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl "http://$MY_NGINX_IP" -H "Host: www.example.com"
```

If everything above is configured correctly, the response should be one of the following:


1. A standard NGINX 404 Not Found page (not ideal)
2. Almost anything else (ideal -- if are using the `jmitchel3/tf-python:latest` image in your deployment)
   

Example standard NGINX 404 Not Found page (not what we want):
```html
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```


#### Closing thoughts

Ingress with NGINX and Kubernetes is great for a lot of reasons which you can now experiment with. I like the idea that any given domain can forward a request to any given Kubernetes service based on a subdomain or path (just like non-kubernetes NGINX can). The request forwarding allows for a lot of flexibility with your deployments, container images, and so on. 

There are a few more things to keep in mind:

1. When in doubt, use the same namespace for everything above (ingress-nginx will use it's own namespace so no need to worry about that). 
2. If you are not sure where you provisioned things, use `kubectl get all -A` and you can scroll through what is currently provisioned
3. IP Addresses and DNS sometimes take a _long time_ to work correctly. Remember you might have to wait a bit.
4. Using different Deployment or Service settings might require different Ingress settings.
5. Wildcard domains are possible but not covered in this post.
6. TLS/HTTPs is possible but not covered in this post (although a few configuration lines are commented out in preparation for this step-- not covered in this post).

Let me know if you have any questions or comments below. Thank you!
