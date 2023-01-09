---
title: Terraforming Kubernetes on Linode
slug: terraforming-kubernetes-on-linode

publish_timestamp: Jan. 2, 2023
url: https://www.codingforentrepreneurs.com/blog/terraforming-kubernetes-on-linode/

---


What if starting a remote virtual machine instance was as simple as changing a number on a document and running a command? That is how simple terraform can be and why it's so powerful.

Using documents to change what server we need running is a feature of Infrastructure as Code and can be a paradigm shift for many developers since it's so simple yet effective. One of the biggest advantages of IaC and terraform: version control and CI/CD pipeline automation. 

Terraform will turn on (and off) the servers we need. Kubernetes will manage how we allocate the resources on those servers to run the applications we want and need. The pair is a juggernaut of automation. This post will teach you exactly what to do to use them both.


__Want a minimal and rapid-fire version of this post? Check out [this repo](https://github.com/codingforentrepreneurs/terraforming-kubernetes-rapid).__

__Watch the related [Terraforming Kubernetes course](https://www.codingforentrepreneurs.com/courses/terraforming-kubernetes-on-linode/)__.

## What is Terraform?

Terraform is an infrastructure as code technology that allows you to declare what hardware requirements your projects needs and, like magic, terraform will start provisioning the resources you need. Terraform isn't exactly magic though since it really leverages a bunch of API calls on your behalf but does so in a way that's elegant and easy to understand.

Furthermore, Terraform is concerned with _results_ not _actions_. Python is concerned with _actions_ not _results_. In other words, Terraform is declarative while Python is imperative. With declarative programming, you just state the outcome you want, with imperative programming, you state all the steps to get the outcome you want.

With the rise in what Machine Learning and Artificial Intelligence can do, we're going to see a _massive rise_ in declarative programming as well leveraging tools like Python and JavaScript as well. With that said, Terraform can give us a glimpse into our declarative future.


## What is Kubernetes?

Kubernetes is a tool that manages running your containerized software. Containers are a way to package your software regardless of the programming language(s) you used to create it. Kubernetes has a lot of key benefits but the primary one we'll focus on is just how simple it can be to deploy your first containerized application on it.

The great thing about Kubernetes is that you can run many applications on only a few virtual machines and Kubernetes will help you efficiently run these applications. There's also built-in load balancing to distribute traffic across running instances of your application. You can also have dedicated IP addresses for specific applications in addition to what your Kubernetes virtual machines have.

There's a _lot_ more to Kubernetes than just this as I will cover in the future.

## Before getting started

There's a few things I recommend you do prior to getting started with this tutorial.

- Install Terraform with the [official install guide](https://developer.hashicorp.com/terraform/downloads)
- Docker experience or watch the [recommended course](https://www.codingforentrepreneurs.com/courses/docker-and-docker-compose/)
- Install the Kubernetes command-line tool `kubectl` with the [official install guide](https://kubernetes.io/docs/tasks/tools/)
-  A Linode account: [new accounts get a $100 credit](https://linode.com/cfe)


## How to get help

Terraform and Kubernetes are technologies that have a _lot_ that can be covered. Here's a few resources to help if or when you get stuck:

- Course: __Terraforming Kubernetes__ _Coming Soon_
- Course: [Try IaC Terraform](https://www.codingforentrepreneurs.com/courses/try-iac-terraform/)
- Course: [Django & Kubernetes](https://www.codingforentrepreneurs.com/courses/django-kubernetes/)
- Reference Code on GitHub: [Terraforming Kubernetes](https://github.com/codingforentrepreneurs/terraforming-kubernetes)
- Discussions on [GitHub](https://github.com/codingforentrepreneurs/terraforming-kubernetes/discussions).


## Creating your Project's Directory
Let's start by creating a new project directory to hold all of our Terraform files and relevant Kubernetes files (called manifests). 

```bash
mkdir -p ~/Dev/terraforming-kubernetes
cd ~/Dev/terraforming-kubernetes
```
For most of my software projects, I work in the `Dev` directory in my user's root folder (right next to `Desktop`,  `Documents`, and `Downloads`).

Now let's make our `devops` folder:

```bash
mkdir -p ~/Dev/terraforming-kubernetes/devops
```

### Optional: Using Git 
At this point, I would initialize git:

`git init`

Then create a `.gitignore` file that combines [the Terraform gitignore](https://github.com/github/gitignore/blob/main/Terraform.gitignore) and the [Kubernetes gitignore](https://github.com/kubernetes/kubernetes/blob/master/.gitignore) files.

This blog post won't touch on git again but it's always a good practice to use especially when working with Terraform and Kubernetes as we'll setup in this one.


## Configure Linode
We start with getting the necessary setup for Linode. It's true that Terraform removes a lot of ClickOps (i.e. click operations in a console), we still have to do some manual work for Terraform to be ready.

### Create an Object Storage Bucket
We want to be able to manage our Terraform project from any computer as well as in a CI/CD pipeline (GitHub Actions, Gitlab, etc) at some point. To do this, we'll use Linode's Object Storage Bucket to store all Terraform state files. 

Terraform creates state files as a way to track what resources need to be created or removed based on the Terraform files we'll create in this project. The reason we have to worry about this now has to do with _how we initialize our Terraform project_.

To create an Object Storage Bucket (it's S3-compatible storage), do the following:

1. Login to Linode.com (or [create an account](https://linode.com/cfe) if you haven't already)
2. Navigate to _Object Storage_
3. Click _Create Bucket_ with the following configuration
   - Label: `terraforming-kubernetes-bucket` _(you must use a new label)_
   - Region: `Atlanta, GA` _(or pick a region close to you)_
   - Click _Create Bucket_ to create the bucket

Object Storage on Linode works a _lot_ like AWS S3 Storage so if you're familiar with the core features of AWS S3, you'll be familiar with Object Storage. Comparable S3 Storage services on other platforms are Google Cloud Storage, Azure Blob Storage, and DigitalOcean Spaces.

### Get Object Storage Bucket Access Key
Now that we have an Object Storage Bucket, we need an access key and secret key so Terraform can store and retrieve state files from the bucket.

1. In the Linode Console, navigate to _Object Storage_
2. Click the tab `Access Keys`
3. Click _Create Access Key_
4. For configuration use:
   - Label: `terraforming-kubernetes-bucket-key`
   - Limit access: yes
   - Locate the `terraforming-kubernetes-bucket` from the previous step and select `Read/Write`
   - Click _Create Access Key_ to create the access key.

Make note of the `Access Key` and `Secret Key` that we get from the above steps as we'll use them in our Terraform `backend` file. Let's create that now:

```bash
echo "" >> ~/Dev/terraforming-kubernetes/devops/backend
```

Notice that `backend` does not have an extension. Now let's add the contents:
```
skip_credentials_validation=true
skip_region_validation=true
bucket="YOUR_CUSTOM_OBJECT_STORAGE_BUCKET_NAME"
key="terraforming-kubernetes.tfstate"
region="us-southeast-1"
endpoint="us-southeast-1.linodeobjects.com"
access_key="YOUR_CUSTOM_S3_ACCESS_KEY"
secret_key="YOUR_CUSTOM_S3_SECRET_KEY"
```
Replace the following:
-  `YOUR_CUSTOM_OBJECT_STORAGE_BUCKET_NAME` with the object storage bucket you just created
-  `YOUR_CUSTOM_S3_ACCESS_KEY` with the access key you just created
-  `YOUR_CUSTOM_S3_SECRET_KEY` with the secret key you just created
- The `region` and `endpoint` will need to change depending on what your bucket says. For example, my bucket endpoint is `terraforming-kubernetes-bucket.us-southeast-1.linodeobjects.com`

These Object Storage steps are critical to ensuring our Terraform project is cloud-based and _not_ tied to a single computer thus being far more resilient to failure.

### Create a Personal Access Token
The final piece we need is a Linode Personal Access Token (PAT) so Terraform can leverage the Linode API to create/update/destroy resources on our behalf.


1. Navigate to [API Tokens](https://cloud.linode.com/profile/tokens) on the Linode Console (Under Profile > API Tokens or [this link](https://cloud.linode.com/profile/tokens))
2. Click `Create a Personal Access Token`
3. For configuration use:
   - Label: `terraforming-kubernetes-PAT-local`
   - Expiry: `In 3 Months`
   - Under `Select All` hit `None` to deselect all permissions
   - Select `Read/Write` to the following:
     - Databases
     - Domains
     - Events
     - Firewalls
     - Kubernetes
     - Images
     - IPs
     - Linode
     - NodeBalancers
     - Object Storage
     - Volumes


Let's create the `terraform.tfvars` file `devops`:

```bash
echo "" >> ~/Dev/terraforming-kubernetes/devops/terraform.tfvars
```

Add the line:
```
linode_api_token="YOUR_LINODE_PAT"
```
- Replace `YOUR_LINODE_PAT` with the token we just created. 

We'll add more variables to the `terraform.tfvars` file later but for now this is good.

Now these are the actual variables we'll inject into our Terraform project. To use the values in `terraform.tfvars` we need a `variables` declaration file in our project so let's create one now.

In `devops/variables.tf` add:

```hcl
variable "linode_api_token" {
    description = "Your Linode API Personal Access Token. (required)"
    sensitive   = true
}
```
What this will do is allow us to use the `linode_api_token` variable in our Terraform project with the reference declared in `variables.tf` and the value directly from `terraform.tfvars` (we'll see this value when we use `terraform console` in a few sections).

At this point, we should have everything we need to do within the Linode console. From here on out, we'll use Terraform to start/update/stop services (resources) we may need.


## Initialize Terraform

Terraform treats all `.tf` files as 1 big file if it's in the same directory. In our case, the root terraform project directory is `devops` and it currently should contain at least three files:

- `devops/backend`
- `devops/terraform.tfvars`
- `devops/variables.tf`

Let's create our main Terraform file:

In `devops/main.tf` add:
```hcl
terraform {
    required_version = ">= 0.15"
    required_providers {
        linode = {
            source = "linode/linode"
        }
    }
     backend "s3" {}
}

provider "linode" {
    token = var.linode_api_token
}
```
- `terraform {}` is the root block for our Terraform providers. In this case, we're using the Linode provider but we could always add other providers that you can find on the [Terraform Registry](https://registry.terraform.io/browse/providers).
- `required_version` is the minimum version of Terraform we need to run this project. We're using `0.15` or greater to ensure all the features of this project work.
- `backend "s3" {}` tells terraform to use the `s3` backend for the state files. It does not have a reference to the `backend` file we created just yet but we'll add that soon.
- `provider "linode" {}` is a required declaration to initialize this specific provider with the required `token` we created earlier. You can review the official [Linode Terraform Provider Docs](https://registry.terraform.io/providers/linode/linode/latest/docs) documentation for more.
- `var.linode_api_token` works because `linode_api_token` is declared in `variables.tf`. The value of `linode_api_token` comes from `terraform.tfvars`.


Now we can initialize Terraform:

```bash
cd ~/dev/terraforming-kubernetes
terraform -chdir=./devops init --backend-config=backend
```
Let's break this down:
- `terraform` is the CLI command
- `-chdir=./devops` is the directory change into for our various Terraform commands. Personally I use `-chdir` constantly since my terraform projects are often a subdirectory in a project with other technologies. You can also use `cd ./devops` and then run `terraform init --backend-config=backend` but I prefer the `-chdir` option as it is more explicit and easier to reuse.
- `init` is the command to initialize this Terraform project
- `-backend-config=backend` is the relative path to the `backend` file. Since we used `-chdir` we do not need to add additional paths to this file (e.g. `terraform -chdir=./devops init --backend-config=./devops/backend` would be looking in `~dev/terraforming-kubernetes/devops/devops/` ).

Now that we have our Terraform project and state initialized, let's just use terraform to create a simple file.

### Path-based Variables in Terraform
Before we can create a file, we have to understand how paths work in Terraform by leveraging local variables. We'll start start by declaring an absolute filepath to store this file as a localized variable. 

In `devops/local.tf` add:
```hcl
locals {
    root_dir = "${dirname(abspath(path.root))}"
    k8s_config_dir = "${local.root_dir}/.kube/"
    k8s_config_file = "${local.root_dir}/.kube/kubeconfig.yaml"
}
```

We have 3 definitions here. Let's break them down:

- `root_dir`, `k8s_config_dir`, and `k8s_config_file` are all local variables we can use anywhere by simply referencing `local.<key>` such as `locals.k8s_config_file` or `locals.root_dir
- `dirname()`, `abspath()`, and `path.root` I'll explain in the Terraform Console section next.
- `"${}"` - this is how we can use string substition in terraform much like `${}` in JavaScript.
- `"${local.root_dir}/.kube/kubeconfig.yaml"` and `"${local.root_dir}/.kube/` both reference the `root_dir` variable in the same way we can in other places. 

To better understand locals (or any other variables in Terraform), it's a good idea to use the _Terraform Console_ as it's a great way to experiment with the values in our current Terraform project.

### Terraform Console

Now we'll better understand what's going on in `locals.tf` by using the Terraform Console.

```bash
cd ~/dev/terraforming-kubernetes
terraform -chdir=./devops
```

Now enter a few commands:

- `path.root`
- `abspath(path.root)`
- `dirname(abspath(path.root))`
- `local.root_dir`
- `local.k8s_config_dir`
- `local.not_real`
- `var.linode_api_token`

Seeing the results of these commands will help us understand what's going on in `locals.tf` but here's how it shakes out:

- `path.root` is a named value that is available to us in Terraform. It is the root directory of the project. In our case, it is `~/dev/terraforming-kubernetes/devops`
- `abspath()` is a function that returns the absolute path of a given path. In my case, it will return `/Users/cfe/dev/terraforming-kubernetes/devops` 
- `dirname()` is a function that returns the parent directory of that path that is passed in. In my case, it will return `~/dev/terraforming-kubernetes`
- `local.root_dir` and `local.k8s_config_dir` should give us the expected values on our local file system.
- `local.not_real` should give us an error `Error: Reference to undeclared local value`.
- `var.linode_api_token` should respond with `(sensitive value)` because in `variables.tf` we added `sensitive = true` to it's declaration.

Going forward if you _ever_ need to better understand something in Terraform, the `terraform console` is a great way to do so. 

Now let's create our local file using these variables and placeholder data.

### Creating a Local File in Terraform

The file we're going to create is going to be a placeholder for the Kubernetes config file (e.g. `kubeconfig.yaml`). Using a placeholder is great as it gives us a introduction to how Terraform can add/update/remove resources and files on our behalf.

First, we'll declare a location we want to store this file.  I want it to be in `~/dev/terraforming-kubernetes/.kube`

In `devops/cluster.tf` add:

```
resource "local_file" "k8s_config" {
    content = "Actual data coming soon"
    filename = "${local.k8s_config_file}"
    file_permission = "0600"
}
```
Let's break this down:
- `resource "local_file"` is a built-in (non provider specific) Terraform resource that we're declaring.
- `k8s_config` is the name we have given this resource. We can reference this resource (after we `apply it` in the next part) with `resource.local_file.k8s_config` or `local_file.k8s_config`
- `content` expects a string value. In this case, we're just using a placeholder.
- `filename` is, in our case, is the absolute path to where we want to output this file
- `file_permission = "0600"` A permission of 0600 will make the file readable and writable only by the owner of the file which is our current user.

With this data understood, let's exit the terraform console with:

```
exit
```

Now that we are back in our terminal, let's review the changes we want to terraform to make as well as apply them.

### Using Terraform Plan and Apply for the first time

Since our project now has 1 resource, we can apply the changes using terraform. Before we apply the changes we can review them with `terraform plan`:

```bash
terraform -chdir=./devops plan
```
This will output a lot of information that shows us what _will_ happen if we apply these changes. In this case, it should be just creating 1 resource (aka our local file).

Now let's apply the changes:

```bash
terraform -chdir=./devops apply
```
Now we see the same information as `terraform plan` but with a confirmation dialog. If we type `yes` and hit enter, Terraform will create the file we declared in `cluster.tf`.

Let's verify that the file was created:

```bash
cat ./.kube/kubeconfig.yaml
```
As a response, we should see:
```stdout
Actual data coming soon
```

Let's now delete this file and all other terraform resources with:

```
terraform -chdir=./devops destroy
```
Once again you'll be asked to confirm this action. If you type `yes` and hit enter, Terraform will delete the file we just created.

If you're a software developer, what we just did is _mind numbing_ since it provides very little value.

Let's add some real value by creating a Kubernetes cluster.

## Terraform a Kubernetes Cluster

Linode has a managed Kubernetes cluster service called LKE (Linode Kubernetes Engine). LKE removes a lot of the complexity of self-managing a Kubernetes cluster and for no additional cost.

Upon reviewing the Linode Terraform Provider, we can see the available resource `linode_lke_cluster` ([docs](https://registry.terraform.io/providers/linode/linode/latest/docs/resources/lke_cluster)). This is the resource we need to configure.

In `devops/cluster.tf` add:

```
resource "linode_lke_cluster" "terraform_k8s" {
    k8s_version="1.24"
    label="terraform-k8s"
    region="us-east"
    tags=["terraform-k8s"]
    pool {
        type  = "g6-standard-1"
        count = 3

    }
}
```
Using hard-coded values like this is fine to get started but as our projects grow more complex, it's better to use input variables that we can change at will.


### Input Variables in Terraform

Now we are going to abstract the _hard-coded_ values that we set in the previous section to into nput variables so we can _easily re-use_ this entire terraform project in the future for another/different Kubernetes cluster.

To do so, let's first define what we want as variables by updating `devops/terraform.tfvars`:

```
linode_api_token="YOUR_LINODE_PAT"
k8s_label="terraform-k8s"
k8s_region="us-east"
tags=["terraform-k8s"]
pools=[
    {
        type = "g6-standard-1"
        count = 3
    }
]
```
Notice these values are identical to the values we used for `resource "linode_lke_cluster" "terraform_k8s"` but just changed slightly to fit variable values.

Values added to `terraform.tfvars` is useless unless we decare them as variables in `devops/variables.tf` much like we did with the `linode_api_token` earlier in this tutorial.

Let's update `devops/variables.tf` with the simple variables first:

```hcl
variable "k8s_version" {
    description = "The Kubernetes version to use for this cluster. (required)"
    default = "1.24"
}

variable "k8s_label" {
    description = "The unique label to assign to this cluster. (required)"
    default = "tf-k8s-cluster"
}

variable "k8s_region" {
    description = "The region where your cluster will be located. (required)"
    default = "us-east"
}
```
As we see here, each variable has a name (e.g. `k8s_version`), a description, and a default value. The default value means that we can omit this variable in `terraform.tfvars` and the default value will be used. To reference each of these variables anywhere in our project, it's a simple as:
- `"${var.k8s_version}"` or `var.k8s_version`
- `"${var.k8s_label}"` or `var.k8s_label`
- `"${var.k8s_region}"` or `var.k8s_region`

Now let's define the `tags` variable:

```hcl
variable "tags" {
    description = "Tags to apply to your cluster for organizational purposes."
    type = list(string)
    default = ["terraform-k8s-cluster"]
}
```
Again we see a name, description, type, and default value. We also see a new argument know nas `type`. This argument is a check that ensures the value we set for ths variable is of the correct type that we define here.  The type is `list(string)` which means that we are declaring that this variable will be a list of strings values. 

Let's take a look at a more advanced type delcaration: a list of objects by defining the `pools` variable. First let's see how we define an object by considering the following:
```
my_object = {
    label = "Hello World"
    id = 123
    tags = ["terraform", "kubernetes"]
}
```
In Terraform files, we would define this as:
```hcl
my_object = {
    label = string
    id = number
    tags = list(string)
}
```
This way of defining an object, is how we define a variable that contains a list of objects: 
```hcl
variable "pools" {
    description = "The Node Pool specifications for the Kubernetes cluster. (required)"
    type = list(object({
        type = string
        count = number
    }))
    default = [
        {
            type = "g6-standard-1"
            count = 3
        }
    ]
}
```

Let's hone in on the data type declaration:
```
type = list(object({
    type = string
    count = number
}))
```
This should make matters confusing:
- `type = list()`
- `type = string`

The first type is defined on the variable itself. The second type is defined in the object inside the list that defines the variable data type.

In other words:

- `type = list(object(...))`: As before, this is the data type definition for this variable that contains objects.
- `type = string`: Defines the data type for the `type` key in the object that is inside the list that makes up this variable's data type.

This declaration ensures that `variable "pool"` is set to a list of objects (dictionaries) that include two keys: `type` and `count`. The `type` key contains a value that is of the data type of `string` and the `count` key with a value that is of the data type `number`.

When we declare variables, we have 3 options for setting the value of each variable:

- `default` declared in the variable definition. This is the fallback value if it's not declared in `terraform.tfvars`.
- `terraform.tfvars` with the same name as the variable definition (e.g. `pools`)
- Inputted at runtime for `terraform apply`. This happens if the `default` and the `terraform.tfvars` value for this variable are not set.

To read up on these kinds of variables, consider checking out the [input variables](https://developer.hashicorp.com/terraform/language/values/variables) documentation on Terraform.

The last thing to note with this variable declaration is `g6-standard-1`. This is the type of instance that Linode provides as you can see in the list [here](https://api.linode.com/v4/linode/types).

### Leveraging Dynamic Input Variables in Terraform

Now that we have a few more input variables, it's time to update our cluster definition;

First, let's do the easy ones:

```hcl
resource "linode_lke_cluster" "terraform_k8s" {
    k8s_version = var.k8s_version
    label = var.k8s_label
    region = var.k8s_region
    tags = var.tags
    pool {
        type  = "g6-standard-1"
        count = 3

    }
```
As we can see, we can simply use `var` to set the value of each of these variables.

How about our `pool` values? This requires a `dynamic` declaration on the resource. Heres how that's done:

```hcl
resource "linode_lke_cluster" "terraform_k8s" {
    ...
    dynamic "pool" {
        for_each = var.pools
        content {
            type  = pool.value["type"]
            count = pool.value["count"]
        }
    }
}
```
- `dynamic "pool"` is a way to tell terraform to dynamically set this entire argument for this resource. 
- `for_each = var.pools` is a way to tell terraform to iterate over the list of objects that we defined in `var.pools` in `variables.tf` and `terraform.tfvars`.
- `content {}` is a way to tell terraform to set the values of the keys in the object that we are iterating over.
- `type =` and `count =` are the keys we will set for this argument as defined by the `lke_cluster` resource in the [Linode Terraform Provider docs](https://registry.terraform.io/providers/linode/linode/latest/docs/resources/lke_cluster).

Now that we have the resource `linode_lke_cluster.terraform_k8s` we can use it to update our local Kubernetes config file.

### Dynamic Output based on Terraform Resources
The final step in our Terraform code is to output an actual Kubernetes configuration file from the `linode_lke_cluster` resource just defined. Just as a reminder, to reference a resource in Terraform, we use the following syntax: `resource_type.resource_name`. In our case, we want to reference the `linode_lke_cluster` resource we just defined, so we use `linode_lke_cluster.terraform_k8s`. Once we can reference a resource, we can see various attributes avaialble to us by using the `terraform console` command or reviewing the [docs](https://registry.terraform.io/providers/linode/linode/latest/docs/resources/lke_cluster#attributes-reference).

What we'll find is the following key attributes available on the `linode_lke_cluster` resource:

- `status` - The status of the cluster.

- `kubeconfig` - The base64 encoded kubeconfig for the Kubernetes cluster.

- `dashboard_url` - The Kubernetes Dashboard access URL for this cluster.

There are other attributes available, but we really need the `kubeconfig` attribute to get our local Kubernetes config file updated. Here's how that's done:

```hcl
resource "local_file" "k8s_config" {
    content = "${nonsensitive(base64decode(linode_lke_cluster.terraform_k8s.kubeconfig))}"
    filename = "${local.k8s_config_file}"
    file_permission = "0600"
}
```
As we see, we have just updated the `content` argument for our local file which contains:

- `nonsensitive()` - This is a way to tell Terraform that this value can be outputted to a file and/or console. This is important because the `kubeconfig` attribute is typically treated as sensitive data since it gives us admin access to our Kubernetes cluster. Do _not_ use this function lightly.
- `base64decode()` will decode the values from base64.
- `linode_lke_cluster.terraform_k8s.kubeconfig` will output the base64 encoded value that contains the Kubernetes cluster's config file. You can also download this file manually form the Linode console. Once you have access to kubernetes, it's recommended that you start creating Role-Based Access controls (RBAC) for your cluster instead of this file. Doing so is outside the context of this tutorial.

### Apply and Destroy
Now we should have everything we need to Terraform our Kubernetes cluster. Let's do that now:

```bash
terraform -chdir=./devops apply
```

After about 10 or 15 minutes, your Kubernetes cluster will be ready. We'll work with this cluster in the next step.

When in doubt, destroy your cluster to avoid accruing charges:

```bash
terraform -chdir=./devops apply -destroy
```
The shortcut of this command is `terraform -chdir=./devops destroy`.

At this point, I recommend you practice destroying and creating your Kubernetes cluster a few times to get the hang of how easy it is. Definitely consider changing some of the variables (as sometimes you have to change labels based on Linode's API requirements). The goal of doing this will help you fully appreciate how simple and effective terraform can be.

You might be wondering about how to actually deploy an application on Kubernetes. That's what comes next.

## Deploy a Sample Application to Kubernetes

If this is your first foray into Kubernetes, strap in because it will be a bumpy ride. If this is your first foray into Docker Containers, this section is probably not for you and I recommend you watch my [Docker & Docker Compose](https://www.codingforentrepreneurs.com/courses/docker-and-docker-compose/) course.

Kubernetes is a powerful tool that runs our Docker containers for us. There's a _lot_ more to leveraging Kubernetes then what's in this post but here's what we'll do:

- Define a Deployment based on a Public Docker Image on DockerHub
- Define a Service to expose our Deployment to the Internet
- Use kubectl to apply our Deployment and Service to our Kubernetes cluster
- Use kubectl to take down our Deployment and Service


### Manifests and YAML 101
Kubernetes uses the concept of "manifests" which, like Terraform, are declarative files. Kubernetes manifests are defined in the YAML format (`.yaml` or `.yml`). YAML format _really_ depends on how you space things but it's a lot like writing Objects in JavaScript or Dictionaries in Python or JSON except instead of using `{}` or `[]` we use spacing, `:` and `-`.

Let's consider the following JSON:

```JSON
{
    "name": "John McClain",
    "age": 30,
    "cars": [
        {"name": "Ford", "models": ["Fiesta", "Focus", "Mustang"]},
        {"name": "BMW", "models": ["320", "X3", "X5"]},
        {"name": "Fiat", "models": ["500", "Panda"]}
    ]
}
```
If we want to see this same object in YAML, it would be:

```yaml
name: John McClain
age: 30
cars:
  - name: Ford
    models:
      - Fiesta
      - Focus
      - Mustang
  - name: BMW
    models:
      - 320
      - X3
      - X5
  - name: Fiat
    models:
      - 500
      - Panda
```
Aside from John's taste in cars, this should help you understand the formatting for YAML. If it's still too confusing, you might consider looking for a YAML validator online.

Now, we're going to use YAML to define a Kubernetes deployment.

### Define a Deployment
If you ask me, deployments are the best building block of an application in Kubernetes. You can define everything you application needs to run including:
- The container image
- The ports the container expects
- The ports you want to expose to the container (not to the internet yet)
- non-sensitive environment variables 
- References to secrets (which are sensitive environment variables)
- health check paths (for web applications)
- And much more

Deployments tell Kubernetes how many _pods_ we'll need running this containerized application. _Pods_ are more fundamental to Kubernetes but defining pods directly is much like telling a mexican restaurant which tortillas they should use for each of your 1000 tacos order.

For now, we'll just define a deployment and let Kubernetes handle the rest.

In `k8s/deployment.yaml`, add the following:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cfe-nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: cfe-nginx-deployment
  template:
    metadata:
      labels:
        app: cfe-nginx-deployment
    spec:
      containers:
        - name: cfe-nginx-container
          image: codingforentrepreneurs/cfe-nginx:latest
          imagePullPolicy: Always
          ports:
            - name: cfe-nginx-port
              containerPort: 80
```
Let's break this down:
- `apiVersion: apps/v1` and `kind: Deployment` tell Kubernetes that we're defining a Deployment. These two are required for all Kubernetes manifests.
- `cfe-nginx-deployment` is the name of our deployment and one I tend to reuse for other labels as well. The other places we use this label will be described in another tutorial
- `replicas: 3` tells Kubernetes to run 3 pods of this container. Think of a pod as a "running instance" of your containerized application. Pods are units that run on our Kubernetes cluster. If you were to deploy this manually, you would likely be running this container on 3 different virtual machines. Kubernetes and specifically Kubernetes Deployments manage this for us. When pods fail, Kubernetes will automatically bring them back up. 
- `image: codingforentrepreneurs/cfe-nginx:latest` is a _public_ Docker container image that is based on the NGINX container image with custom HTML for demos like this. The code for this container is available on [GitHub](https://github.com/codingforentrepreneurs/cfe-nginx) and on [DockerHub](https://hub.docker.com/r/codingforentrepreneurs/cfe-nginx).
- `imagePullPolicy: Always` tells Kubernetes to always pull the tagged version of the image (in this case we use the `latest` tag in the `image` declaration above). 
- `ports:` tells Kubernetes which ports we want to expose to the container. In this case, we're exposing port 80 to the container. This is the default port for NGINX. We use the `name: cfe-nginx-port` to help us identify this port in the Kubernetes Service manifest we'll define soon.


### Define a Service
A service is what tells Kubernetes how to expose our application to the internet. We can use a service to expose a single port or multiple ports. We can also use a service to expose a single application or multiple applications.

In `k8s/service.yaml`, add the following:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: cfe-nginx-service
spec:
  selector:
    app: cfe-nginx-deployment
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80
      targetPort: cfe-nginx-port
  
```
Let's break this down:
- `apiVersion: v1` and `kind: Service` tell Kubernetes that we're defining a Service. Like before, these are required.
- `cfe-nginx-service` is the name of our service.
- `selector: app: cfe-nginx-deployment` tells Kubernetes which deployments we want this service to target. In this case, we're targeting the `cfe-nginx-deployment` deployment we in `deployment.yaml` above.
- `type: LoadBalancer` tells Kubernetes to expose this service to the internet as a Load Balancer. This definition will give this service a Public IP Address that will forward traffic to the any of our 3 available pod replicas of the `cfe-nginx-deployment` labeled deployment.
- `protocol: TCP` tells Kubernetes to use the TCP protocol to expose this service to the internet. This is the default protocol for HTTP traffic.
- `port: 80` is the default port for non-secure HTTP traffic. This is the port we want to expose to the internet.
- `targetPort: cfe-nginx-port` `cfe-nginx-port` is the name of the port we defined in `deployment.yaml` above. This is the port we want port 80 to forward to. Technically speaking, `cfe-nginx-port` is currently tied to `80` but if we ever change that port in our deployment, this service will not need to be  updated at all. (In docker terms, it's much like doing `-p 80:80` in the `docker run` command). 

Now that we have our manifests defined, we can deploy them to our Kubernetes cluster.

### Kubernetes Configuration with Environment Variables

To use work with our Kubernetes cluster, we must use the `kubectl` command line tool. `kubectl` is the official command line tool for Kubernetes. It's a powerful tool that allows us to manage our Kubernetes cluster from the command line. At the beginning of this tutorial, I recommended that you install `kubectl` ([official install guide](https://kubernetes.io/docs/tasks/tools/)) so do that now if you haven't.

We need to configure our `kubectl` to use the terraform-generated `kubeconfig.yaml` file. This file contains the configuration for our Kubernetes cluster and grants `kubectl` permission to manage our cluster and make real changes. As a security measure, we don't want to commit this file to our repository _and_ we should look into leveraging Role-Based Access Control (RBAC) to limit the permissions of our `kubectl` configuration. For now, we'll just use the `kubeconfig.yaml` file as-is.

The easiest way I have found to is by setting the `KUBECONFIG` environment variable on a per-project basis. Setting this variable tells `kubectl` where to find your project's `kubeconfig.yaml` file.

_on macOS and Linux_
```bash
mkdir -p ~/Dev/terraforming-kubernetes
export KUBECONFIG=./.kube/kubeconfig.yaml
```

_on Windows_
```bash
mkdir -p ~\Dev\terraforming-kubernetes
$Env:KUBECONFIG=(".\\.kube\\kubeconfig.yaml")
```

#### Visual Studio Code Setup

If you are using Visual Studio Code (VS Code) and the VS Code terminal, you can use the following declaration in your `settings.json` file to set the `KUBECONFIG` environment variable for you:

```json
{
	"folders": [
		{
			"path": "."
		}
	],
	"settings": {
		"files.autoSave": "afterDelay",
		"terminal.integrated.env.osx": {
            "KUBECONFIG": "${workspaceFolder}/.kube/kubeconfig.yaml"
        },
        "terminal.integrated.env.windows": {
            "KUBECONFIG": "${workspaceFolder}\\.kube\\kubeconfig.yaml"
        },
        "terminal.integrated.env.linux": {
            "KUBECONFIG": "${workspaceFolder}/.kube/kubeconfig.yaml"
        },
	}
}
```

Now, let's verify that our `kubectl` is configured correctly by running the following command:

```bash
kubectl get nodes
```

This should respond the following or similar:

```stdout
NAME                           STATUS   ROLES    AGE   VERSION
lke85823-131094-63b398ac5c86   Ready    <none>   17h   v1.24.8
lke85823-131094-63b398ac7e0c   Ready    <none>   17h   v1.24.8
lke85823-131094-63b398ac9ee4   Ready    <none>   17h   v1.24.8
```
These 3 nodes exist because of our definition in Terraform and the `terraform.tfvars` file.

If you see an error with the above, you can try:

```bash
cd ~/Dev/tetraforming-kubernetes
KUBECONFIG=./.kube/kubeconfig.yaml kubectl get nodes
```
If that works, then you set your environment variable incorrectly. If that fails as well, you might consider re-running `terraform -chdir=./devops apply` (and possibly `terraform -chdir=./devops destroy`) to regenerate your `./kube/kubeconfig.yaml` file.


### Kubectl Apply Deployments

Assuming `kubectl get nodes` worked, and we have `k8s/deployment.yaml` and `k8s/service.yaml`, let's deploy our application to Kubernetes. We can do this by running the following command:

```bash
kubectl apply -f k8s/deployment.yaml
```

Right away we can run:

```
kubectl get pods -w
```
This will `watch` our pods being created. You should see something like this:

```stdout
cfe-nginx-deployment-645c56d8fb-mtcds   1/1     Running             0          4s
cfe-nginx-deployment-645c56d8fb-mw5fc   0/1     ContainerCreating   0          4s
cfe-nginx-deployment-645c56d8fb-n4s97   0/1     ContainerCreating   0          4s
cfe-nginx-deployment-645c56d8fb-mw5fc   1/1     Running             0          5s
cfe-nginx-deployment-645c56d8fb-n4s97   1/1     Running             0          5s
```
Use `CTRL` + `c` to stop watching the pods being created. Once you do, you can run:

```bash
kubectl get deployments
```

And you should see:
```stdout
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
cfe-nginx-deployment   3/3     3            3           60s
```
This shows us that we have 3/3 of our replicas running, up-to-date, and available for 60 seconds.

At this point our application is not exposed to the internet but we can enter the shell of any given deployment with:

```
kubectl exec -it deployments/cfe-nginx-deployment -- /bin/bash
```
This is a lot like using SSH into a virtual machine the only difference is that we're going into the shell of a running container within a pod based on the deployment we specified.

To go into a specific pod, we can use:

```
kubectl exec -it cfe-nginx-deployment-645c56d8fb-mtcds -- /bin/bash
```
Keep in mind that `cfe-nginx-deployment-645c56d8fb-mtcds` is unique to my cluster and will definitely change over time. I tend to opt for `kubectl exec -it deployments/<deploymen-name> -- /bin/bash` over `kubectl exec -it <podname> -- /bin/bash` as it's easier to remember.

Exit the `kubectl exec` shell with entering `exit` and/or `CTRL` + `d`.


### Kubectl Apply Services

Now that we have our application deployed, we can expose it to the internet by running:

```bash
kubectl apply -f k8s/service.yaml
```
Now we can run:

```
kubectl get service
```
And we should see something that resembles:
```
NAME                TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)        AGE
cfe-nginx-service   LoadBalancer   10.128.31.238   139.144.240.198   80:30834/TCP   20s
kubernetes          ClusterIP      10.128.0.1      <none>            443/TCP        17h
```
Notice that we have an External IP address. This address will _not match_ the IP Address of any of the Kubernetes cluster nodes (i.e. virtual machine instances) as Kubernetes services can/will be assigned an IP Address directly from Linode (or any other managed service provider you run this on).

If you visit the External IP address in your browser, you should see the following:

![cfe-nginx screenshot](https://github.com/codingforentrepreneurs/cfe-nginx/blob/main/repo-assets/homepage-screenshot.png?raw=true)

This, of course, is the default image from the [cfe-nginx](https://github.com/codingforentrepreneurs/cfe-nginx) project we used in our deployment.


### Kubectl Destroy and Apply

Before we delete our deployment and service, let's use a nice shortcut to apply any/all changes our manifests may have:

```bash
kubectl apply -f k8s/
``` 
We can use this command because the `k8s` directory contains both our `deployment.yaml` and `service.yaml` files. We should see the following:

```
deployment.apps/cfe-nginx-deployment unchanged
service/cfe-nginx-service unchanged
```

It's a pretty neat shortcut that we can use to _bring down_ all kinds of Kubernetes resources with:

```
kubectl delete -f k8s/
```
Which should respond with:

```
deployment.apps "cfe-nginx-deployment" deleted
service "cfe-nginx-service" deleted
```

We can bring it back up with:

```bash
kubectl apply -f k8s/
```

And we should see:

```
deployment.apps/cfe-nginx-deployment created
service/cfe-nginx-service created
```
Then we can verify our service with:

```bash
kubectl get services
```
Notice that your `cfe-nginx-service` External IP address will most likely be different than what we had before. This is because we destroyed our service and created a new one. Kubernetes and Linode will most likely assign a new IP address to our service.

A shortcut to get this IP address is:

```bash
kubectl get service cfe-nginx-service -o "jsonpath={.status.loadBalancer.ingress[0].ip}"
```
There are so many shortcuts like this that you can use with `kubectl` and Kubernetes in general. It's a great tool to learn and use.



## Conclusion and Clean up

At this point, we are ready to dive deeper into using Kubernetes in our projects. Terraform was the jumping off point that we can continue to use and update our cluster as we see fit. 

Before we leave, let's remove this cluster by running:

```
terraform -chdir=./devops destroy
```

After a minute or so, our entire cluster will be destroyed including any services and deployments Kubernetes is running. To bring it back up, here's what we can do:

```
terraform -chdir=./devops apply -auto-approve
```
Wait until terraform completes then run
```
KUBECONFIG=./devops/kubeconfig kubectl apply -f k8s/
```

I will be covering Kubernetes in a lot more detail in the near future so I hope this serves as a taste of what you can do with Terraform and Kubernetes.
