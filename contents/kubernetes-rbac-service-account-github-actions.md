---
title: Kubernetes Service Account &amp; RBAC for GitHub Actions
slug: kubernetes-rbac-service-account-github-actions

publish_timestamp: March 2, 2023
url: https://www.codingforentrepreneurs.com/blog/kubernetes-rbac-service-account-github-actions/

---


In this post, we'll create a Kubernetes Service Account with specific permissions to various assets in our Kubernetes cluster. The concept behind this is called Role-Based Access Controls or RBAC for short. Using this method will allow us to control all or parts of any given Kubernetes cluster for use in a tool like GitHub Actions or Gitlab CI/CD. You can also use RBAC service accounts to share with members of your team if you want to grant cluster access.

RBAC does not necessarily block internal Kubernetes communication between pods, but it does allow us to control what a given service account can do to said pods.

## Assumptions

- Kubectl installed locally
- A running Kubernetes cluster just like we did in the [Terraforming Kubernetes course](https://www.codingforentrepreneurs.com/courses/terraforming-kubernetes-on-linode/), it's [repo](https://github.com/codingforentrepreneurs/terraforming-kubernetes) or the [rapid-fire terraforming kubernetes repo](https://github.com/codingforentrepreneurs/terraforming-kubernetes-rapid)
- The correct privileges to the Kubernetes cluster (or admin privileges for learning purposes which is done in the above course)



## Create the App Namespace

To better test our service account, we'll create a specific namespace for it. Doing so is not required but it is a good practice to leverage namespaces to better organize and secure your cluster.

```bash
kubectl create namespace apps
```


## Create a Service Account

kubectl has a built-in way to create a service account:

```
kubectl create serviceaccount github-actions-sa -n apps
```
This will create a service account named `github-actions-sa` in the `apps` namespace. The reason we opt for the kubectl command is because it will automatically create a secret for this service account for us. We'll need this secret later on.

Let's review the service account:

```bash
kubectl get serviceaccount github-actions-sa -n apps -o yaml
```

Example response:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: github-actions-sa
  namespace: apps
secrets:
- name: github-actions-sa-token-f9kpn
```

We can see that the service account has a secret associated with it. We'll need this secret later on.


## Create a Kubernetes Role

By default, our service account has no permissions to do anything in our cluster. Creating a role will start the process of allowing permissions to our service account. Once we have the Role configured, we can bind the role to our service account using a RoleBinding. 

Keep in mind that `Role` and `RoleBinding` are namespace specific. If we need custer-wide roles, we can use `ClusterRole` and `ClusterRoleBinding` as drop-in replacements.

For this role, we want to be able to have access to the following resources:
- Pods
- Logs (maps to `pods/log`)
- Deployments
- Services
- StateFulSets
- ConfigMaps
- Secrets
- Persistent Volume Claims

Now that we have a list of resources we want to permit for this service account's role, let's decide on the exact permissions we want to allow for. These permissions correspond to Create Retrieve Update Delete (CRUD) operations. 

So we'll allow the service account to do the following on each of the above resources:

- create, patch, and update
- get, list, watch
- delete

For the role, we refer to each one of these activities as a _verb_ and the entire set as _verbs_. Not surprisingly, the resources and the verbs (CRUD operations) correspond to how we use kubectl:

```bash
kubectl get pods
kubectl get deployment some-deployment
kubectl delete service some-service
```

In the _resources_ list above, we have `StateFulSets` which is the only resource that has a different apiVersion than the rest (at least for now). Let's look at the start of a manifest file for a StateFulSet:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: postgres-db
  name: apps
...
```
As we see, the apiVersion here is `apps/v1` where the other resources apiVersion is simply `v1`. Now these apiVersions may change in the future so it begs the question, how do we know which apiVersion to use for a given resource?

It's as simple as this:

```bash
kubectl explain StatefulSet
```

The first two lines will respond with:

```stdout
KIND:     StatefulSet
VERSION:  apps/v1
```

And here is our answer! We can use the `KIND` as our manifest `kind` and `VERSION` as our manifest `apiVersion`. We can _also_ use these items for our Role's `apiGroups` field as we'll see below.

Here's the full manifest for our Role with the above considerations:


In, `role.yaml`, add the following:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: github-actions-role
  namespace: apps
rules:
- apiGroups: [""]
  resources: 
    - pods
    - pods/log
    - services
    - secrets
    - configmaps
    - persistentvolumeclaims
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: 
    - deployments
    - statefulsets
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

For the `statefulsets` resource, we can drop the `/v1` from the apiVersion to account for future apiVersion changes (e.g `apps/v2` if the ever come to be).


## Binding a Role to a Service Account

We now have the foundation to bind this role to our service account. We just need to have the following:

- Service account's name and namespace (this can be different than the role/rolebinding's namespace)
- Role name and namespace (this must be the same as the RoleBinding's namespace) 

In, `role-binding.yaml`, add the following:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: github-actions-sa-role-binding
  namespace: apps
subjects:
# You can specify more than one service account "subject" if needed
- kind: ServiceAccount
  name: github-actions-sa # "name" is case sensitive
  namespace: apps # namespace must match the correct Service Account namespace
roleRef:
  kind: Role # this must be Role in the same namespace or ClusterRole
  name: github-actions-role # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

## Apply the Role and RoleBinding

Now that we have our Role and RoleBinding manifests, we can apply them to our cluster:

```bash
kubectl apply -f role.yaml
kubectl apply -f role-binding.yaml
```
The order of operations (at least the first time) matters since the `role-binding.yaml` depends on the `role.yaml` existing in the cluster.


## Acting as a Service Account

One of the best parts of using Kubectl, is you can act as a service account to verify various aspects of a role working (or not working).

Let's test this by first changing our namespace context:

```
kubectl config set-context --current --namespace=apps
```

Then, let's try to get a list of pods as th `github-actions-sa` service account:

```bash
kubectl get pods --as=system:serviceaccount:apps:github-actions-sa
```

The `--as` flag is in the following format:

```
--as=system:serviceaccount:{namespace}:{service_account_name}
```
It should be no surprise that `namespace` and `service_account_name` are the same as the ones we used in our RoleBinding manifest as well as when we created the service account in the first place (e.g. `kubectl create serviceaccount github-actions-sa`).

Feel free to test your other resources. You can also test `kubectl apply -f ` with the `--as` flag to see how if those permissions are working correctly.



## Creating a Kubernetes Config File for a Service Account

Now that we have a service account with the correct permissions, we can create a Kubernetes config file for it. This will allow us to replace a `kubeconfig.yaml` file with the `github-actions-sa` service account's credentials.


At this point, we will start using a few _bash shell commands_ to make it easier to modify for future use. If you're using PowerShell on Windows, you will have to adjust your commands slightly. 

First off, let's set some environment variables to make things easier related to our exact service account and related namespace:
```bash
export SERVICE_ACCOUNT="github-actions-sa"
export SERVICE_ACCOUNT_NAMESPACE="apps"
export SERVICE_ACCOUNT_SECRET_NAME=$(kubectl get serviceaccounts $SERVICE_ACCOUNT -n $SERVICE_ACCOUNT_NAMESPACE -o jsonpath="{.secrets[0].name}")

export SERVICE_ACCOUNT_SECRET_CERT=$(kubectl get secret $SERVICE_ACCOUNT_SECRET_NAME -o jsonpath="{.data['ca\.crt']}")
export SERVICE_ACCOUNT_SECRET_TOKEN=$(kubectl get secret $SERVICE_ACCOUNT_SECRET_NAME -o jsonpath="{.data.token}" | base64 -d)
```

Now the Kubernetes cluster details:

```bash
export CLUSTER_URL=$(kubectl config view --minify -o 'jsonpath={.clusters[0].cluster.server}')
export CLUSTER_NAME=$(kubectl config view --minify -o 'jsonpath={.clusters[0].name}')
```

Finally, let's use these values to output a Kubernetes config file for our service account:

  
Still using terminal we'll write:
```bash
cat << EOF > kubeconfig-sa.yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: $SERVICE_ACCOUNT_SECRET_CERT
    server: $CLUSTER_URL
  name: $CLUSTER_NAME
contexts:
- context:
    cluster: $CLUSTER_NAME
    user: $SERVICE_ACCOUNT
  name: $CLUSTER_NAME-ctx
current-context: $CLUSTER_NAME-ctx
kind: Config
users:
- name: $SERVICE_ACCOUNT
  user:
    token: $SERVICE_ACCOUNT_SECRET_TOKEN
EOF
```

Verify the output values are correct by:

```bash
cat kubeconfig-sa.yaml
```

You should see:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: SOME_SUPER_LONG_STRING
    server: https://some-value-here.us-east-1.linodelke.net:443
  name: lke-some-value
contexts:
- context:
    cluster: lke-some-value
    user: lke-some-value-admin
  name: lke-some-value-ctx
current-context: lke-some-value-ctx
kind: Config
users:
- name: lke-some-value-admin
  user:
    token: some-long-token-here
```
and _not_:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: 
    server: 
  name: 
contexts:
- context:
    cluster: 
    user: 
  name: -ctx
current-context: -ctx
kind: Config
users:
- name: 
  user:
    token:
```
If you see a bunch of empty values, you may have missed a step or two. Go back and verify the values are correct. At this point, you have a `KUBECONFIG` file that you could test locally with `kubectl` or use in a Github Action. We'll use a Github Action since we already tested using the `--as` flag.

## Github Action Workflow

For GitHub Actions, we need the following:

- A Github Repo
- The Service Account config file as a Github Secret
- A Github Actions Workflow file

I'll assume you have a GitHub repo already created. If not, create one now.


### Github Secrets

We are copy the contents of our `kubeconfig-sa.yaml` file as a Github Secret. You could break this file apart and store the individual values as secrets, but I find it easier to just copy the entire file as a secret. If you ever need to replace this service account config, you could just replace the entire file as needed.

```
cat kubeconfig-sa.yaml | pbcopy
```
Then, in your Github repo, go to `Settings > Secrets > New Repository Secret` and paste the contents of the `kubeconfig-sa.yaml` file as the value for the secret. I named mine `KUBECONFIG_SA`.

### Create the Github Actions Workflow

Now that we have our Github Secret created, let's implement our Github Actions Workflow as a simple verify workflow.

First, create the `.github/workflows` directory:

```bash
cd path/to/your/local/repo
mkdir -p .github/workflows
```

In `path/to/your/local/repo/.github/workflows/verify-kubectl-sa.yaml`:

```yaml
name: Verify Kubectl Service Account
on: 
  workflow_dispatch:

jobs:
  verify_service_account:
    name: Verify K8s Service Account
    runs-on: ubuntu-latest
    steps:
      - uses: azure/setup-kubectl@v3
      - name: Create/Verify `.kube` directory
        run: mkdir -p ~/.kube/
      - name: Create kubectl config
        run: |
          cat << EOF >> ~/.kube/kubeconfig.yaml
          ${{ secrets.KUBECONFIG_SA }}
          EOF
      - name: Echo pods
        run: |
          KUBECONFIG=~/.kube/kubeconfig.yaml kubectl get pods
      - name: Echo deployments
        run: |
          KUBECONFIG=~/.kube/kubeconfig.yaml kubectl get deployments
```
This workflow is straightforward:

1. We use `uses: azure/setup-kubectl@v3` to ensure `kubectl` is available on this workflow. This is an official Github Action created by Microsoft Azure 
2. We create a kubectl config file via the stored secrets file. For security, GitHub actions secrets are hidden from output even if you run `echo ${{ secrets.KUBECONFIG_SA }}` on a step.
3. We use our newly created kubectl config file to verify access to read `pods` and `deployments`. This verification is merely ensuring our RBAC is working correctly for this Service account. Add other commands as needed to further verify access.

### Run Workflow on Github

Before we can run the workflow, let's push it to GitHub:
```bash
cd path/to/your/local/repo
git add .github/workflows/verify-kubectl-sa.yaml
git commit -m "Created K8S SA Verification Workflow"
git push origin main
```

Since we declared `workflow_dispatch` as the trigger, we can run the workflow manually by going to `Actions > Verify Kubectl Service Account > Run workflow`:

How did it go? Did you have errors? 

Now you are ready to start adding manifests into this GitHub repo and have them automatically deployed to your Kubernetes cluster.
