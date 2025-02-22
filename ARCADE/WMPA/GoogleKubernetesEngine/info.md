# Google Cloud Skills Boost - Lab Reference

## Lab: Configuring IAM Permissions

### Overview
In this lab, you will learn how to configure Identity and Access Management (IAM) permissions in Google Cloud. IAM allows you to manage access control by defining who (identity) has what access (role) for which resource.

### Objectives
- Create IAM policies.
- Assign roles to users.
- Use IAM policies to control access to resources.

### Steps

#### 1. Set Up and Initialize Cloud Shell
Activate Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud. Cloud Shell provides command-line access to your Google Cloud resources.

1. Click Activate Cloud Shell icon at the top of the Google Cloud console.
2. Click through the following windows:
   - Continue through the Cloud Shell information window.
   - Authorize Cloud Shell to use your credentials to make Google Cloud API calls.

When you are connected, you are already authenticated, and the project is set to your Project_ID, PROJECT_ID. The output contains a line that declares the Project_ID for this session:

```
Your Cloud Platform project in this session is set to "PROJECT_ID"
```

`gcloud` is the command-line tool for Google Cloud. It comes pre-installed on Cloud Shell and supports tab-completion.

(Optional) You can list the active account name with this command:
```sh
gcloud auth list
```
Output:
```
ACTIVE: *
ACCOUNT: "ACCOUNT"
```

To set the active account, run:
```sh
gcloud config set account ACCOUNT
```

(Optional) You can list the project ID with this command:
```sh
gcloud config list project
```
Output:
```
[core]
project = "PROJECT_ID"
```
Note: For full documentation of `gcloud`, in Google Cloud, refer to the [gcloud CLI overview guide](https://cloud.google.com/sdk/gcloud).

#### Task 1. Set a Default Compute Zone
Your compute zone is an approximate regional location in which your clusters and their resources live. For example, `us-central1-a` is a zone in the `us-central1` region.

In your Cloud Shell session, run the following commands.

Set the default compute region:
```sh
gcloud config set compute/region "REGION"
```
Expected output:
```
Updated property [compute/region].
```

Set the default compute zone:
```sh
gcloud config set compute/zone "ZONE"
```
Expected output:
```
Updated property [compute/zone].
```

#### Task 2. Create a GKE Cluster
A cluster consists of at least one cluster master machine and multiple worker machines called nodes. Nodes are Compute Engine virtual machine (VM) instances that run the Kubernetes processes necessary to make them part of the cluster.

Note: Cluster names must start with a letter and end with an alphanumeric, and cannot be longer than 40 characters.
Run the following command to create a cluster:
```sh
gcloud container clusters create --machine-type=e2-medium --zone=ZONE lab-cluster
```
You can ignore any warnings in the output. It might take several minutes to finish creating the cluster.

Expected output:
```
NAME: lab-cluster
LOCATION: ZONE
MASTER_VERSION: 1.22.8-gke.202
MASTER_IP: 34.67.240.12
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.22.8-gke.202
NUM_NODES: 3
STATUS: RUNNING
```
Click "Check my progress" to verify the objective.

#### Task 3. Get Authentication Credentials for the Cluster
After creating your cluster, you need authentication credentials to interact with it.

Authenticate with the cluster:
```sh
gcloud container clusters get-credentials lab-cluster
```
Expected output:
```
Fetching cluster endpoint and auth data.
kubeconfig entry generated for my-cluster.
```

#### Task 4. Deploy an Application to the Cluster
You can now deploy a containerized application to the cluster. For this lab, you'll run `hello-app` in your cluster.

GKE uses Kubernetes objects to create and manage your cluster's resources. Kubernetes provides the Deployment object for deploying stateless applications like web servers. Service objects define rules and load balancing for accessing your application from the internet.

To create a new Deployment `hello-server` from the `hello-app` container image, run the following `kubectl create` command:
```sh
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```
Expected output:
```
deployment.apps/hello-server created
```
This Kubernetes command creates a deployment object that represents `hello-server`. In this case, `--image` specifies a container image to deploy. The command pulls the example image from a Container Registry bucket. `gcr.io/google-samples/hello-app:1.0` indicates the specific image version to pull. If a version is not specified, the latest version is used.

Click "Check my progress" to verify the objective.

To create a Kubernetes Service, which is a Kubernetes resource that lets you expose your application to external traffic, run the following `kubectl expose` command:
```sh
kubectl expose deployment hello-server --type=LoadBalancer --port 8080
```
In this command:
- `--port` specifies the port that the container exposes.
- `type="LoadBalancer"` creates a Compute Engine load balancer for your container.

Expected output:
```
service/hello-server exposed
```
To inspect the `hello-server` Service, run `kubectl get`:
```sh
kubectl get service
```
Expected output:
```
NAME             TYPE            CLUSTER-IP      EXTERNAL-IP     PORT(S)           AGE
hello-server     LoadBalancer    10.39.244.36    35.202.234.26   8080:31991/TCP    65s
kubernetes       ClusterIP       10.39.240.1                     443/TCP           5m13s
```
Note: It might take a minute for an external IP address to be generated. Run the previous command again if the EXTERNAL-IP column status is pending.

To view the application from your web browser, open a new tab and enter the following address, replacing `[EXTERNAL IP]` with the `EXTERNAL-IP` for `hello-server`:
```
http://[EXTERNAL-IP]:8080
```
Expected output: The browser tab displays the message `Hello, world!` as well as the version and hostname.

Click "Check my progress" to verify the objective.

#### Task 5. Delete the Cluster
To delete the cluster, run the following command:
```sh
gcloud container clusters delete lab-cluster
```
When prompted, type `Y` and press Enter to confirm.

Deleting the cluster can take a few minutes. For more information, refer to the [Google Kubernetes Engine (GKE) article on Deleting a cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/deleting-a-cluster).

Click "Check my progress" to verify the objective.

### Conclusion
Congratulations! You have just deployed a containerized application to Google Kubernetes Engine! In this lab, you created a GKE cluster, deployed a containerized application to the cluster, created a Kubernetes service, and deleted the cluster. You can now apply this knowledge to deploy your own applications with GKE.

### Additional Resources
- [Google Cloud IAM Documentation](https://cloud.google.com/iam/docs)
- [Google Cloud Console](https://console.cloud.google.com/)