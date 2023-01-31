---
title: Try Knative
slug: try-knative-on-lke

publish_timestamp: Jan. 26, 2023
url: https://www.codingforentrepreneurs.com/blog/try-knative-on-lke/

---


Knative is a Kubernetes-based platform for running serverless. Serverless means you can scale your application to 0 running instances but those instances to quickly scale up to _N_ number of instances within a few seconds. Scaling to 0 is fantastic because it allows Kubernetes and Knative to reallocate resources as needed. If you couple that with an LKE autoscaling feature (which will add compute nodes to your cluster), you can have a very robust system with not much financially invested.

The investment for Knative comes in the form of the learning curve to get it running and unlocking continuous delivery/deployment. This article and the [Try Knative course](https://www.codingforentrepreneurs.com/courses/try-knative/) are here to help you get started using Knative in production using a managed Kubernetes cluster on Linode's LKE service. 


Here's what we're going to cover in this article and [the course](https://www.codingforentrepreneurs.com/courses/try-knative/):

- Using Terraform to create our Kubernetes Cluster on Linode
- Install Knative and Istio
- Configure a Knative Service and Domain Mapping
- Install cert-manager for auto-provisioning of HTTPs certificates
- Configure an Istio Gateway for HTTP and HTTPS requests (ingress)
- Implement Knative service environment variables (both ConfigMap and Secrets)


The [Try Knative repository](https://github.com/codingforentrepreneurs/try-knative) is the reference code for our in-depth course [Try Knative](https://www.codingforentrepreneurs.com/courses/try-knative/) along with this article.

There are a few requirements before you get started.

## Requirements
- Watch [Terraforming Kubernetes on Linode](https://www.codingforentrepreneurs.com/courses/terraforming-kubernetes-on-linode/) or have experience with Kubernetes clusters
- [Git](https://git-scm.com/downloads) installed
- [Terraform](https://developer.hashicorp.com/terraform/downloads) installed
- [Kubectl](https://kubernetes.io/docs/tasks/tools/) installed

If you have all of these complete, let's do this.

## 1. Clone this repo
I have modified the [Terraforming Kubernetes Rapid-Fire](https://github.com/codingforentrepreneurs/terraforming-kubernetes-rapid) repo for this project. I removed the `git` history and `k8s.yaml` file.

```
git clone https://github.com/codingforentrepreneurs/try-knative
cd try-knative
```
Once you clone this repo, you'll have a `installers/install-knative.sh` file that will install Knative Serving and Istio on your Kubernetes cluster based on their standard installation instructions. More on this in a bit.

If you want to start fresh, checkout the `fresh` branch and remove the git history

```
git checkout fresh
rm -rf .git
git init
```
Now you have a fresh repo that includes only the code for the rest of this README.

## 2. Initialize Terraform
Create an account on [Linode](https://www.linode.com/cfe) and get an API Key in your linode account [here](https://cloud.linode.com/profile/tokens).

Once you have a key, do the following:

```bash
echo "linode_api_token=\"YOUR_API_KEY\"" >> terraform.tfvars
echo "k8s_node_type=\"g6-standard-2\"" >> terraform.tfstate
```

Through my tests, the minimum node instance type we need to use for Knative is `g6-standard-2` with a minimum of 3 nodes in the cluster.

Now run
```bash
terraform init
```
Be sure that if you're using _git_ that you have at least [this .gitignore](https://github.com/codingforentrepreneurs/try-knative/blob/main/.gitignore) file in your repo.


## 3. Add Autoscaling to your Kubernetes Cluster

In `infra.tf`, we'll update the `linode_lke_cluster.terraform_k8s` resource to add autoscaling to our cluster and update the `k8s_version` to `1.25`.

```hcl
resource "linode_lke_cluster" "terraform_k8s" {
    k8s_version="1.25"
    label="try-knative"
    region="us-east"
    tags=["try-knative"]
     pool {
        type  = var.k8s_node_type
        count = 3
        autoscaler {
            min = 3
            max = 8
        }
    }
}
```

The `autoscaler` declaration will ensure that Kubernetes has as many nodes as it needs to run your containerized applications. LKE will manage this for you without further intervention. The _autoscaler_ has strange behavior in Terraform so we'll update it in a bit.

_Optional_: I also updated the `label`, and `tags` to be `try-knative` instead of `terraform-k8s` for this project. You should update them as you see fit. I left the name of Terraform resource `linode_lke_cluster` as `terraform_k8s` in order to limit how much of `infra.tf` we need to change. 


## 4. Terraform your Kubernetes Cluster

```bash
terraform apply
```
> Use `terraform apply -auto-approve` if you're really in a hurry.


## 5. Update Terraform Lifecycle to ignore Autoscaling Changes

Adding the autoscaling declaration in our LKE resource is a bit wonky because Terraform  will _constantly_ try to update your cluster to the declare document state.

For example, if LKE autoscaled your cluster to 5 nodes, Terraform will try to update your cluster to 3 nodes. To fix this, we can just _ignore_ changes to the `pool` declaration until we want to explicitly change it.

```hcl
resource "linode_lke_cluster" "terraform_k8s" {
    k8s_version="1.24"
    label="terraform-k8s"
    region="us-east"
    tags=["terraform-k8s"]
     pool {
        type  = var.k8s_node_type
        count = 3
        autoscaler {
            min = 3
            max = 8
        }
    }
    lifecycle {
        ignore_changes = [
            pool,
        ]
        create_before_destroy = true
    }
}
```


## 6. Install Knative Serving and Istio

Knative Serving is the core Kubernetes component that allows you to run serverless containerized applications. Istio is a service mesh that allows you to manage traffic between your applications and, for our case, with the outside world.

You can install them on these links _or_ our bash script below:

- Install [Knative Serving Docs](https://knative.dev/docs/install/yaml-install/serving/install-serving-with-yaml/#install-the-knative-serving-component)
- Install [Knative + Istio Installation Docs](https://knative.dev/docs/install/yaml-install/serving/install-serving-with-yaml/#install-a-networking-layer)


The macOS/Linux bash script is avaiable in this repo at: [installers/install-knative-istio.sh](https://github.com/codingforentrepreneurs/try-knative/blob/main/installers/install-knative-istio.sh). Let's run it with now:

```bash
chmod +x ./installers/install-knative-istio.sh
./installers/install-knative-istio.sh
```

At this time, it's recommended that Windows users use the links for _Knative Serving_ and _Knative + Istio_ installation from above.


## 7. Confirm Installation

When you run `./installers/install-knative-istio.sh`, you should see the output related to your knative/istio installation. Here's the command we can run again to get our Knative Ingress IP address. If you're using Linode and LKE, this IP Address is configured from a Linode load balancing service called NodeBalancers which is how we can have Kubernetes provide us another IP Address.

```bash
kubectl --namespace istio-system get service istio-ingressgateway
export KNATIVE_INGRESS_IP=$(kubectl --namespace istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

echo "Your IP Address is: $KNATIVE_INGRESS_IP"
echo "Add a cname record for your domain using the above IP address."
```

## 8. Deploy a Knative Service

At this point, we should have Knative Installed on our Kubernetes cluster. The istio configuration will come later but for now, let's create a Knative Service.

Create a folder called `k8s` and `cfe-nginx` with the following:
```bash
mkdir -p k8s/cfe-nginx
```

In `k8s/cfe-nginx` create a file called `service.yaml` with the following:

```yaml
---
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: cfe-nginx
spec:
  template:
    spec:
      containers:
        - name: cfe-nginx-container
          image: codingforentrepreneurs/cfe-nginx:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 80
          env:
            - name: RELEASE_VERSION
              value: "V2"
```
This Knative service will be responsible for spinning up deployments (and indirectly pods) to run the `codingforentrepreneurs/cfe-nginx:latest` container image. The `imagePullPolicy` is set to `Always` so that if we make nearly *any* changes to `service.yaml` Knative will attempt to download a new version of our container image.

Now run:

```
kubectl apply -f k8s/cfe-nginx/service.yaml
```
The internal route for this service will be `cfe-nginx.apps.svc.cluster.local` that we will use shortly to test this service. You can also verify this url with:

```bash
kubectl get routes -n default
```
The service route *will change* later in this post. To get the cluster-based (internal) service url, you can always run:

```bash
kubectl get ksvc cfe-nginx  -o jsonpath='{.status.address.url}'
```
These commands are also referenced (including shortcuts) on [this document](https://github.com/codingforentrepreneurs/try-knative/blob/main/command-notes.md)





## 9. Apply a Kubernetes Deployment 

In order to test a domain-less Knative service, we need to create a Kubernetes deployment. With this Kubernetes deployment we can do a internal `curl` request to our Knative service thus verifying that it's working correctly (at least inside Kubernetes).

In `k8s/cfe-nginx` create a file called `deployment.yaml` with the following:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: example-deployment
  template:
    metadata:
      labels:
        app: example-deployment
    spec:
      containers:
        - name: example-container
          image: codingforentrepreneurs/cfe-nginx:latest
          imagePullPolicy: Always
          ports:
            - name: example-port
              containerPort: 80
```


Now we can run:

```
kubectl apply -f k8s/cfe-nginx/deployment.yaml
```

After some time, you can execute "curl" from within this deployment with:

```
kubectl exec -n default -it deployments/example-deployment -- bash -c "curl http://cfe-nginx.apps.svc.cluster.local"
```
Notice a few things:

- `-n default` assumes that we have not changed the namespace for any of our resources. If you have, you can change this to the appropriate namespace.
- `deployments/example-deployment` is the name of the deployment we just created.
- `-- bash -c "curl http://cfe-nginx.apps.svc.cluster.local"` is the command we want to run inside the container.

After we run this command we should see:

```html
<!DOCTYPE html>
... <!-- removed to conserve space-->
<body>
  <div class="center">
   <h1>Hello World</h1> 
   <p>This page is working with NGINX</p>
  </div>
</body>
</html>
```

If you do not see this result, that means your Knative service isn't working correctly. Remember, we used the container image `codingforentrepreneurs/cfe-nginx:latest`.


# 10. Creating a Custom Domain

If you want traffic to your service from the outside world, you'll need to implement a custom domain name such as TryKnative.com. 

To do this, it requires 2 steps:

- Update the Knative Serving default domain for services
- Add an A Record with the Knative/Istio IP Address


## Update the Knative Serving default domain for services

Now we need to update the default `knative-serving` configmap for our custom domain. Before we do, let's review the default routes based on the default installation:

```
kubectl get routes -A
```
For your service, you should see the url:

```
http://cfe-nginx.default.svc.cluster.local
```

We want this to be our custom domain name. Let's update the configmap to have the following block:

```yaml
data:
    tryknative.com: |
    
    _example: |
    ...
```
The `tryknative.com` represents your custom domain name. This blog post merely uses `tryknative.com` as an example you can work off.

The `_example` block is a non-functional block that gives you a lot of configuration options for your custom domain(s) within Knative Serving.

To edit this configmap, we run:

```bash
KUBE_EDITOR="nano" kubectl edit configmap config-domain -n knative-serving
```
This will open the `nano` editor instead of the `vim` default. 

As a reminder, add the following in the `data` block:

```yaml
data:
    tryknative.com: |

    _example: |
    ...
```
I have found that the line break between your custom domain and `_example: |` tends to yield the best results but it's likely unnecessary.

Close the editing session and save the file. (with Nano it's pressing `Ctrl + C`, `y` and `Enter`/`Return`)

To verify the changes, we can run:

```
kubectl get routes -A
```
This should show our custom domain name now.

## Add an A Record with the Knative/Istio IP Address

At this point, we should have already seen this IP address a couple of times.

To get it, we simply run:

```bash
kubectl --namespace istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```
This value we'll want to update our A Records on our Domain Registrar for our custom domain. I recommend using a wildcard record so any new Knative services you create will automatically work with this IP address.

For example purposes, let's say our `IP Address` from the istio-ingressgateway is: 

```
23.92.23.85
```
Then we can update our Domain A Records to:

| Hostname    | Target      | TTL         |
| ----------- | ----------- | ----------- |
| *      | 23.92.23.85       | 300         |
| www   | 23.92.23.85       | 300         |
| *.default | 23.92.23.85 | 300         |

The `*.default` record is directly correlated to the namespace the Knative service is running in. In this case, we never declared a namespace so it defaults to `default`. If you want to use a different namespace in your Knative service (such as `apps` or `demo`), you would add the `*.apps` or `*.demo` records to target the same IP address.

Changing these records will update the default Knative Service Route:

```
kubectl get route cfe-nginx
```

This should now yield our custom domain name:

`http://cfe-nginx.default.tryknative.com`


The internal route for this Knative service will remain the same:

```
kubectl get ksvc cfe-nginx -o jsonpath='{.status.address.url}')
```

This should yield:

`http://cfe-nginx.default.svc.cluster.local`

When we need to have internal communication to this Knative service, this is the url you *should* use as it will be far more efficient than the external url.



# 10. Creating a Virtual Service for a Root Custom Domain

Let's be real, `cfe-nginx.default.tryknative.com` is great but not ideal for end users unless we're testing a new version of our service of course.

Instead, I want:

- tryknative.com
- www.tryknative.com

To point to my recently deployed Knative service. To do this, we'll use an Istio Virtual Service.


In `k8s/virtual-service.yaml`, add the following:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: tryknative-root
  namespace: default
spec:
  gateways:
    - knative-shared-gateway.knative-serving.svc.cluster.local
    - knative-serving/knative-ingress-gateway
  hosts:
  - tryknative.com
  - www.tryknative.com
  http:
    - name: http-route
      match: 
        - uri:
           prefix: "/" # maps to http://cfe-nginx.apps.tryknative.com/
           # we direct these domain hosts to a different path on our
           # deployed service
           # prefix: "/blog" # would map to http://cfe-nginx.apps.tryknative.com/blog
      rewrite:
        authority: cfe-nginx.default.tryknative.com
      route:
      - destination:
          host: cfe-nginx.default.svc.cluster.local
          port:
            number: 80
        weight: 100
```

What we see here is a Virtual Service that will route traffic from `tryknative.com` and `www.tryknative.com` to our Knative service. The key things you'll need to customize over time are:

- `rewrite: authority: cfe-nginx.default.tryknative.com`
- `route: destination: host: cfe-nginx.default.svc.cluster.local`

At this point you should also recognize where these values are coming from but I'll reiterate it once again:

- `cfe-nginx` is the name of our Knative service
- `default` is the namespace our `cfe-nginx` Knative service.
- `tryknative.com` is the custom domain we added to the _knative-serving_ `config-domain` _configmap_ earlier. If we had *more than one* domain in this configmap, we would need to update how our Knative Gateway routes traffic (this is beyond the scope of this blog post)
- `svc.cluster.local` is the default Kubernetes cluster domain name that will route traffic to any given service.

We need to use rewrite on the custom domain to ensure that the `Host` header is set to the custom domain name. This is important because the Knative Gateway will use this header to route traffic to the correct Knative service.

We need to route the destination to the `cfe-nginx.default.svc.cluster.local` because this is the internal url for our Knative service. This is important because the Knative Gateway will use this url to route traffic to the correct Knative service.

## Next steps

To get a more in-depth look at how all this is done, consider enrolling in the [Try Knative](https://www.codingforentrepreneurs.com/courses/try-knative/) course. In the course, we also show you how to use secrets and configmaps for environment variables on your Knative service. We also show you how to deploy the default nginx container to really highlight how powerful Knative can be.

Watch the [course demo](https://www.codingforentrepreneurs.com/courses/try-knative/lessons/demo-x7l6/) right now.

The next big piece would be implementing HTTPs with [cert-manager](https://cert-manager.io/). If you are interested in seeing how, please comment below and/or [upvote the suggestion](https://www.codingforentrepreneurs.com/suggest/653/).


Did I miss anything? Let me know in the comments below. Thank you!
