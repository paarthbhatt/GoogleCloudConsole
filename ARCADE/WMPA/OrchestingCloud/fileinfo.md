# Orchestrating the Cloud with Kubernetes

**Lab**: GSP021  
**Schedule**: 1 hour 15 minutes  
**Cost**: No cost  
**Level**: Intermediate  
**Info**: This lab may incorporate AI tools to support your learning.

![Google Cloud self-paced labs logo](https://cloud.google.com/images/labs/logo.svg)

## Overview
Kubernetes is an open source project (available on [kubernetes.io](https://kubernetes.io/)) which can run on many different environments, from laptops to high-availability multi-node clusters, from public clouds to on-premise deployments, from virtual machines to bare metal.

For this lab, using a managed environment such as Kubernetes Engine allows you to focus on experiencing Kubernetes rather than setting up the underlying infrastructure. Kubernetes Engine is a managed environment for deploying containerized applications. It brings the latest innovations in developer productivity, resource efficiency, automated operations, and open source flexibility to accelerate your time to market.

**Note**: App is hosted on GitHub and provides an example 12-Factor application. During this lab you will be working with the following Docker images:
- `kelseyhightower/monolith` - Monolith includes auth and hello services.
- `kelseyhightower/auth` - Auth microservice. Generates JWT tokens for authenticated users.
- `kelseyhightower/hello` - Hello microservice. Greets authenticated users.
- `nginx` - Frontend to the auth and hello services.

### Objectives
In this lab you will learn how to:
- Provision a complete Kubernetes cluster using Kubernetes Engine.
- Deploy and manage Docker containers using `kubectl`.
- Break an application into microservices using Kubernetes' Deployments and Services.

### Setup and Requirements
Before you click the Start Lab button:
- Read these instructions. Labs are timed and you cannot pause them. The timer, which starts when you click Start Lab, shows how long Google Cloud resources are made available to you.
- This hands-on lab lets you do the lab activities in a real cloud environment, not in a simulation or demo environment. It does so by giving you new, temporary credentials you use to sign in and access Google Cloud for the duration of the lab.
- To complete this lab, you need:
  - Access to a standard internet browser (Chrome browser recommended).
  - **Note**: Use an Incognito (recommended) or private browser window to run this lab. This prevents conflicts between your personal account and the student account, which may cause extra charges incurred to your personal account.
  - Time to complete the lab—remember, once you start, you cannot pause a lab.
  - **Note**: Use only the student account for this lab. If you use a different Google Cloud account, you may incur charges to that account.

### How to Start Your Lab and Sign In to the Google Cloud Console
1. Click the **Start Lab** button. If you need to pay for the lab, a dialog opens for you to select your payment method. On the left is the **Lab Details** pane with the following:
   - The **Open Google Cloud console** button
   - Time remaining
   - The temporary credentials that you must use for this lab
   - Other information, if needed, to step through this lab
2. Click **Open Google Cloud console** (or right-click and select **Open Link in Incognito Window** if you are running the Chrome browser).
3. The lab spins up resources, and then opens another tab that shows the Sign in page.
4. **Tip**: Arrange the tabs in separate windows, side-by-side.
5. **Note**: If you see the Choose an account dialog, click **Use Another Account**.
6. If necessary, copy the **Username** below and paste it into the Sign in dialog.
7. Click **Next**.
8. Copy the **Password** below and paste it into the Welcome dialog.
9. Click **Next**.
10. **Important**: You must use the credentials the lab provides you. Do not use your Google Cloud account credentials.
11. **Note**: Using your own Google Cloud account for this lab may incur extra charges.
12. Click through the subsequent pages:
    - Accept the terms and conditions.
    - Do not add recovery options or two-factor authentication (because this is a temporary account).
    - Do not sign up for free trials.
13. After a few moments, the Google Cloud console opens in this tab.
14. **Note**: To access Google Cloud products and services, click the **Navigation menu** or type the service or product name in the **Search field**.

### Activate Cloud Shell
Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud. Cloud Shell provides command-line access to your Google Cloud resources.

1. Click **Activate Cloud Shell** icon at the top of the Google Cloud console.
2. Click through the following windows:
   - Continue through the Cloud Shell information window.
   - Authorize Cloud Shell to use your credentials to make Google Cloud API calls.
3. When you are connected, you are already authenticated, and the project is set to your Project_ID, `PROJECT_ID`. The output contains a line that declares the `Project_ID` for this session:
   ```
   Your Cloud Platform project in this session is set to "PROJECT_ID"
   ```
4. `gcloud` is the command-line tool for Google Cloud. It comes pre-installed on Cloud Shell and supports tab-completion.
   - (Optional) You can list the active account name with this command:
     ```sh
     gcloud auth list
     ```
     **Output**:
     ```
     ACTIVE: *
     ACCOUNT: "ACCOUNT"
     ```
     To set the active account, run:
     ```sh
     gcloud config set account `ACCOUNT`
     ```
   - (Optional) You can list the project ID with this command:
     ```sh
     gcloud config list project
     ```
     **Output**:
     ```
     [core]
     project = "PROJECT_ID"
     ```
   **Note**: For full documentation of `gcloud`, in Google Cloud, refer to the [gcloud CLI overview guide](https://cloud.google.com/sdk/gcloud).

### Google Kubernetes Engine
In the cloud shell environment type the following command to set the zone:
```sh
gcloud config set compute/zone Zone
```

Start up a cluster for use in this lab:
```sh
gcloud container clusters create io --zone Zone
```

You are automatically authenticated to your cluster upon creation. If you lose connection to your Cloud Shell for any reason, run the `gcloud container clusters get-credentials io` command to re-authenticate.

**Note**: It will take a while to create a cluster - Kubernetes Engine is provisioning a few Virtual Machines behind the scenes for you to play with!

#### Task 1. Get the Sample Code
Copy the source code from the Cloud Shell command line:
```sh
gsutil cp -r gs://spls/gsp021/* .
```

Change into the directory needed for this lab:
```sh
cd orchestrate-with-kubernetes/kubernetes
```

List the files to see what you're working with:
```sh
ls
```

The sample has the following layout:
```
deployments/  /* Deployment manifests */
  ...
nginx/        /* nginx config files */
  ...
pods/         /* Pod manifests */
  ...
services/     /* Services manifests */
  ...
tls/          /* TLS certificates */
  ...
cleanup.sh    /* Cleanup script */
```

Now that you have the code -- it's time to give Kubernetes a try!

#### Task 2. Quick Kubernetes Demo
The easiest way to get started with Kubernetes is to use the `kubectl create` command.

Use it to launch a single instance of the nginx container:
```sh
kubectl create deployment nginx --image=nginx:1.10.0
```

Kubernetes has created a deployment -- more about deployments later, but for now all you need to know is that deployments keep the pods up and running even when the nodes they run on fail.

In Kubernetes, all containers run in a pod.

Use the `kubectl get pods` command to view the running nginx container:
```sh
kubectl get pods
```

Once the nginx container has a Running status you can expose it outside of Kubernetes using the `kubectl expose` command:
```sh
kubectl expose deployment nginx --port 80 --type LoadBalancer
```

So what just happened? Behind the scenes Kubernetes created an external Load Balancer with a public IP address attached to it. Any client who hits that public IP address will be routed to the pods behind the service. In this case that would be the nginx pod.

List our services now using the `kubectl get services` command:
```sh
kubectl get services
```

**Note**: It may take a few seconds before the ExternalIP field is populated for your service. This is normal -- just re-run the `kubectl get services` command every few seconds until the field populates.

Add the External IP to this command to hit the Nginx container remotely:
```sh
curl http://<External IP>:80
```

And there you go! Kubernetes supports an easy to use workflow out of the box using the `kubectl run` and `expose` commands.

**Test completed task**
Click **Check my progress** below to check your lab progress. If you successfully created a Kubernetes cluster and deploy a Nginx container, you'll see an assessment score.

Create a Kubernetes cluster and launch Nginx container

Now that you've seen a quick tour of Kubernetes, it's time to dive into each of the components and abstractions.

#### Task 3. Pods
At the core of Kubernetes is the Pod.

Pods represent and hold a collection of one or more containers. Generally, if you have multiple containers with a hard dependency on each other, you package the containers inside a single pod.

**Pod containing the monolith and nginx containers**

In this example there is a pod that contains the monolith and nginx containers.

Pods also have Volumes. Volumes are data disks that live as long as the pods live, and can be used by the containers in that pod. Pods provide a shared namespace for their contents which means that the two containers inside of our example pod can communicate with each other, and they also share the attached volumes.

Pods also share a network namespace. This means that there is one IP Address per pod.

Next, a deeper dive into pods.

#### Task 4. Creating Pods
Pods can be created using pod configuration files. Take a moment to explore the monolith pod configuration file.

Go to directory:
```sh
cd ~/orchestrate-with-kubernetes/kubernetes
```

Run the following:
```sh
cat pods/monolith.yaml
```

The output shows the open configuration file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: monolith
  labels:
    app: monolith
spec:
  containers:
    - name: monolith
      image: kelseyhightower/monolith:1.0.0
      args:
        - "-http=0.0.0.0:80"
        - "-health=0.0.0.0:81"
        - "-secret=secret"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
```

There's a few things to notice here. You'll see that:
- Your pod is made up of one container (the monolith).
- You're passing a few arguments to our container when it starts up.
- You're opening up port 80 for http traffic.

Create the monolith pod using `kubectl`:
```sh
kubectl create -f pods/monolith.yaml
```

Examine your pods. Use the `kubectl get pods` command to list all pods running in the default namespace:
```sh
kubectl get pods
```

**Note**: It may take a few seconds before the monolith pod is up and running. The monolith container image needs to be pulled from the Docker Hub before you can run it.

Once the pod is running, use `kubectl describe` command to get more information about the monolith pod:
```sh
kubectl describe pods monolith
```

You'll see a lot of the information about the monolith pod including the Pod IP address and the event log. This information will come in handy when troubleshooting.

Kubernetes makes it easy to create pods by describing them in configuration files and easy to view information about them when they are running. At this point you have the ability to create all the pods your deployment requires!

#### Task 5. Interacting with Pods
By default, pods are allocated a private IP address and cannot be reached outside of the cluster. Use the `kubectl port-forward` command to map a local port to a port inside the monolith pod.

**Note**: From this point on the lab will ask you to work in multiple cloud shell tabs to set up communication between the pods. Any commands that are executed in a second or third command shell will be denoted in the command's instructions.

Open a second Cloud Shell terminal. Now you have two terminals, one to run the `kubectl port-forward` command, and the other to issue `curl` commands.

In the 2nd terminal, run this command to set up port-forwarding:
```sh
kubectl port-forward monolith 10080:80
```

Now in the 1st terminal start talking to your pod using `curl`:
```sh
curl http://127.0.0.1:10080
```

Yes! You got a very friendly "hello" back from your container.

Now use the `curl` command to see what happens when you hit a secure endpoint:
```sh
curl http://127.0.0.1:10080/secure
```

Uh oh.

Try logging in to get an auth token back from the monolith:
```sh
curl -u user http://127.0.0.1:10080/login
```

At the login prompt, use the super-secret password `password` to login. Logging in caused a JWT token to print out.

Since Cloud Shell does not handle copying long strings well, create an environment variable for the token.
```sh
TOKEN=$(curl http://127.0.0.1:10080/login -u user | jq -r '.token')
```

Enter the super-secret password `password` again when prompted for the host password.

Use this command to copy and then use the token to hit the secure endpoint with `curl`:
```sh
curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure
```

At this point you should get a response back from our application, letting us know everything is right in the world again.

Use the `kubectl logs` command to view the logs for the monolith Pod.
```sh
kubectl logs monolith
```

Open a 3rd terminal and use the `-f` flag to get a stream of the logs happening in real-time:
```sh
kubectl logs -f monolith
```

Now if you use `curl` in the 1st terminal to interact with the monolith, you can see the logs updating (in the 3rd terminal):
```sh
curl http://127.0.0.1:10080
```

Use the `kubectl exec` command to run an interactive shell inside the Monolith Pod. This can come in handy when you want to troubleshoot from within a container:
```sh
kubectl exec monolith --stdin --tty -c monolith -- /bin/sh
```

For example, once you have a shell into the monolith container you can test external connectivity using the `ping` command:
```sh
ping -c 3 google.com
```

Be sure to log out when you're done with this interactive shell.
```sh
exit
```

As you can see, interacting with pods is as easy as using the `kubectl` command. If you need to hit a container remotely, or get a login shell, Kubernetes provides everything you need to get up and going.

#### Task 6. Services
Pods aren't meant to be persistent. They can be stopped or started for many reasons - like failed liveness or readiness checks - and this leads to a problem:

What happens if you want to communicate with a set of Pods? When they get restarted they might have a different IP address.

That's where Services come in. Services provide stable endpoints for Pods.

**Service network diagram**

Services use labels to determine what Pods they operate on. If Pods have the correct labels, they are automatically picked up and exposed by our services.

The level of access a service provides to a set of pods depends on the Service's type. Currently there are three types:
- ClusterIP (internal) -- the default type means that this Service is only visible inside of the cluster,
- NodePort gives each node in the cluster an externally accessible IP and
- LoadBalancer adds a load balancer from the cloud provider which forwards traffic from the service to Nodes within it.

Now you'll learn how to:
- Create a service
- Use label selectors to expose a limited set of Pods externally

#### Task 7. Creating a Service
Before you can create services, first create a secure pod that can handle https traffic.

If you've changed directories, make sure you return to the `~/orchestrate-with-kubernetes/kubernetes` directory:
```sh
cd ~/orchestrate-with-kubernetes/kubernetes
```

Explore the monolith service configuration file:
```sh
cat pods/secure-monolith.yaml
```

Create the secure-monolith pods and their configuration data:
```sh
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
kubectl create -f pods/secure-monolith.yaml
```

Now that you have a secure pod, it's time to expose the secure-monolith Pod externally. To do that, create a Kubernetes service.

Explore the monolith service configuration file:
```sh
cat services/monolith.yaml
```

**Output**:
```yaml
kind: Service
apiVersion: v1
metadata:
  name: "monolith"
spec:
  selector:
    app: "monolith"
    secure: "enabled"
  ports:
    - protocol: "TCP"
      port: 443
      targetPort: 443
      nodePort: 31000
  type: NodePort
```

Things to note:
- There's a selector which is used to automatically find and expose any pods with the labels `app: monolith` and `secure: enabled`.
- Now you have to expose the nodeport here because this is how you'll forward external traffic from port 31000 to nginx (on port 443).

Use the `kubectl create` command to create the monolith service from the monolith service configuration file:
```sh
kubectl create -f services/monolith.yaml
```

**Output**:
```
service/monolith created
```

**Test completed task**
Click **Check my progress** below to check your lab progress. If you successfully created Monolith pods and service, you'll see an assessment score.

**Create Monolith Pods and Service**
