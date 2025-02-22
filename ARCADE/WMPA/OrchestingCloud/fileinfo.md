# Activating Cloud Shell

Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud. Cloud Shell provides command-line access to your Google Cloud resources.

1. **Activate Cloud Shell**:
   - Click the *Activate Cloud Shell* icon at the top of the Google Cloud console.
   - Continue through the Cloud Shell information window.
   - Authorize Cloud Shell to use your credentials to make Google Cloud API calls.

   When connected, you are already authenticated, and the project is set to your `PROJECT_ID`. The output contains a line that declares the `PROJECT_ID` for this session:
   ```
   Your Cloud Platform project in this session is set to "PROJECT_ID"
   ```

2. **Using gcloud**:
   - gcloud is the command-line tool for Google Cloud. It comes pre-installed on Cloud Shell and supports tab-completion.
   - (Optional) List the active account name with this command:
     ```
     gcloud auth list
     ```
   - To set the active account, run:
     ```
     gcloud config set account `ACCOUNT`
     ```
   - (Optional) List the project ID with this command:
     ```
     gcloud config list project
     ```

   For full documentation of gcloud, refer to the [gcloud CLI overview guide](https://cloud.google.com/sdk/gcloud).

# Google Kubernetes Engine

1. **Setting the Zone**:
   - In the Cloud Shell environment, set the zone:
     ```
     gcloud config set compute/zone Zone
     ```

2. **Starting a Cluster**:
   - Start up a cluster for use in this lab:
     ```
     gcloud container clusters create io --zone Zone
     ```
   - If you lose connection to your Cloud Shell, re-authenticate with:
     ```
     gcloud container clusters get-credentials io
     ```

   Note: It will take a while to create a cluster as Kubernetes Engine provisions a few Virtual Machines.

# Task 1: Get the Sample Code

1. **Copy the Source Code**:
   - Copy the source code from the Cloud Shell command line:
     ```
     gsutil cp -r gs://spls/gsp021/* .
     ```
   - Change into the directory needed for this lab:
     ```
     cd orchestrate-with-kubernetes/kubernetes
     ```
   - List the files to see what you're working with:
     ```
     ls
     ```

   The sample layout includes directories like `deployments/`, `nginx/`, `pods/`, `services/`, `tls/`, and a cleanup script `cleanup.sh`.

# Task 2: Quick Kubernetes Demo

1. **Launch nginx Container**:
   - Use `kubectl create` to launch a single instance of the nginx container:
     ```
     kubectl create deployment nginx --image=nginx:1.10.0
     ```

   - Kubernetes creates a deployment. Use `kubectl get pods` to view the running nginx container:
     ```
     kubectl get pods
     ```

   - Expose the nginx container outside of Kubernetes:
     ```
     kubectl expose deployment nginx --port 80 --type LoadBalancer
     ```

   - List services:
     ```
     kubectl get services
     ```

   - Add the External IP to this command to hit the Nginx container remotely:
     ```
     curl http://<External IP>:80
     ```

# Task 3: Pods

1. **Creating Pods**:
   - Navigate to the required directory:
     ```
     cd ~/orchestrate-with-kubernetes/kubernetes
     ```

   - Explore the monolith pod configuration file:
     ```
     cat pods/monolith.yaml
     ```

   - Create the monolith pod:
     ```
     kubectl create -f pods/monolith.yaml
     ```

   - Examine your pods:
     ```
     kubectl get pods
     ```

   - Describe the monolith pod:
     ```
     kubectl describe pods monolith
     ```

# Task 4: Interacting with Pods

1. **Setup Port-Forwarding**:
   - Open a second Cloud Shell terminal and run:
     ```
     kubectl port-forward monolith 10080:80
     ```

   - In the first terminal, start talking to your pod using curl:
     ```
     curl http://127.0.0.1:10080
     ```

   - Get an auth token:
     ```
     curl -u user http://127.0.0.1:10080/login
     ```

   - Create an environment variable for the token:
     ```
     TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq -r '.token')
     ```

   - Hit the secure endpoint with curl:
     ```
     curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure
     ```

2. **Viewing Logs**:
   - View logs for the monolith Pod:
     ```
     kubectl logs monolith
     ```

   - Get a stream of the logs in real-time:
     ```
     kubectl logs -f monolith
     ```

3. **Interactive Shell**:
   - Run an interactive shell inside the Monolith Pod:
     ```
     kubectl exec monolith --stdin --tty -c monolith -- /bin/sh
     ```

   - Test external connectivity:
     ```
     ping -c 3 google.com
     ```

   - Log out when done:
     ```
     exit
     ```

# Task 5: Services

1. **Creating a Service**:
   - Navigate to the required directory:
     ```
     cd ~/orchestrate-with-kubernetes/kubernetes
     ```

   - Explore the secure-monolith service configuration file:
     ```
     cat pods/secure-monolith.yaml
     ```

   - Create the secure-monolith pods and their configuration data:
     ```
     kubectl create secret generic tls-certs --from-file tls/
     kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
     kubectl create -f pods/secure-monolith.yaml
     ```

   - Explore the monolith service configuration file:
     ```
     cat services/monolith.yaml
     ```

   - Create the monolith service:
     ```
     kubectl create -f services/monolith.yaml
     ```

   - Allow traffic to the monolith service on the exposed nodeport:
     ```
     gcloud compute firewall-rules create allow-monolith-nodeport --allow=tcp:31000
     ```

# Task 6: Adding Labels to Pods

1. **Adding Labels**:
   - Add the missing `secure=enabled` label to the secure-monolith Pod:
     ```
     kubectl label pods secure-monolith 'secure=enabled'
     kubectl get pods secure-monolith --show-labels
     ```

   - View the list of endpoints on the monolith service:
     ```
     kubectl describe services monolith | grep Endpoints
     ```

# Task 7: Deploying Applications with Kubernetes

1. **Creating Deployments**:
   - Examine the auth deployment configuration file:
     ```
     cat deployments/auth.yaml
     ```

   - Create the auth deployment:
     ```
     kubectl create -f deployments/auth.yaml
     ```

   - Create the auth service:
     ```
     kubectl create -f services/auth.yaml
     ```

   - Create and expose the hello deployment:
     ```
     kubectl create -f deployments/hello.yaml
     kubectl create -f services/hello.yaml
     ```

   - Create and expose the frontend deployment:
     ```
     kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
     kubectl create -f deployments/frontend.yaml
     kubectl create -f services/frontend.yaml
     ```

   - Interact with the frontend:
     ```
     kubectl get services frontend
     curl -k https://<EXTERNAL-IP>
     ```

Congratulations! You've developed a multi-service application using Kubernetes. The skills you've learned here will allow you to deploy complex applications on Kubernetes using a collection of deployments and services.