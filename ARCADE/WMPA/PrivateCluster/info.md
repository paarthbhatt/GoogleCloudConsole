# Google Cloud Shell and Private Kubernetes Cluster Setup Guide

## Activate Cloud Shell

Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud. Cloud Shell provides command-line access to your Google Cloud resources.

1. Click **Activate Cloud Shell** icon at the top of the Google Cloud console.
2. Click through the following windows:
   - Continue through the Cloud Shell information window.
   - Authorize Cloud Shell to use your credentials to make Google Cloud API calls.

When you are connected, you are already authenticated, and the project is set to your `PROJECT_ID`. The output contains a line that declares the `PROJECT_ID` for this session:

```
Your Cloud Platform project in this session is set to "PROJECT_ID"
```

`gcloud` is the command-line tool for Google Cloud. It comes pre-installed on Cloud Shell and supports tab-completion.

(Optional) You can list the active account name with this command:
```
gcloud auth list
```
Output:
```
ACTIVE: *
ACCOUNT: "ACCOUNT"
```

To set the active account, run:
```
gcloud config set account `ACCOUNT`
```

(Optional) You can list the project ID with this command:
```
gcloud config list project
```
Output:
```
[core]
project = "PROJECT_ID"
```
*Note: For full documentation of `gcloud`, refer to the gcloud CLI overview guide.*

## Task 1. Set the Region and Zone

Set the project region for this lab:
```
gcloud config set compute/zone "Zone"
```

Create a variable for region:
```
export REGION=Region
```

Create a variable for zone:
```
export ZONE=Zone
```
*Learn more from the Regions & Zones documentation.*

*Note: When you run `gcloud` on your own machine, the config settings are persisted across sessions. But in Cloud Shell, you need to set this for every new session or reconnection.*

## Task 2. Creating a Private Cluster

When you create a private cluster, you must specify a /28 CIDR range for the VMs that run the Kubernetes master components and you need to enable IP aliases.

Next, you'll create a cluster named `private-cluster`, and specify a CIDR range of `172.16.0.16/28` for the masters. When you enable IP aliases, you let Kubernetes Engine automatically create a subnetwork for you.

You'll create the private cluster by using the `--private-cluster`, `--master-ipv4-cidr`, and `--enable-ip-alias` flags.

Run the following to create the cluster:
```
gcloud beta container clusters create private-cluster \
    --enable-private-nodes \
    --master-ipv4-cidr 172.16.0.16/28 \
    --enable-ip-alias \
    --create-subnetwork ""
```

## Task 3. View Your Subnet and Secondary Address Ranges

List the subnets in the default network:
```
gcloud compute networks subnets list --network default
```

In the output, find the name of the subnetwork that was automatically created for your cluster. For example, `gke-private-cluster-subnet-xxxxxxxx`. Save the name of the cluster, you'll use it in the next step.

Get information about the automatically created subnet:
```
gcloud compute networks subnets describe [SUBNET_NAME] --region=$REGION
```
The output shows you the primary address range with the name of your GKE private cluster and the secondary ranges:
```
...
ipCidrRange: 10.0.0.0/22
kind: compute#subnetwork
name: gke-private-cluster-subnet-163e3c97
...
privateIpGoogleAccess: true
...
secondaryIpRanges:
- ipCidrRange: 10.40.0.0/14
  rangeName: gke-private-cluster-pods-163e3c97
- ipCidrRange: 10.0.16.0/20
  rangeName: gke-private-cluster-services-163e3c97
...
```
In the output, you can see that one secondary range is for pods and the other secondary range is for services. Notice that `privateIpGoogleAccess` is set to true. This enables your cluster hosts, which have only private IP addresses, to communicate with Google APIs and services.

## Task 4. Enable Master Authorized Networks

At this point, the only IP addresses that have access to the master are the addresses in these ranges:

- The primary range of your subnetwork. This is the range used for nodes.
- The secondary range of your subnetwork that is used for pods.

To provide additional access to the master, you must authorize selected address ranges.

Create a VM instance:
```
gcloud compute instances create source-instance --zone=$ZONE --scopes 'https://www.googleapis.com/auth/cloud-platform'
```

Get the `<External_IP>` of the source-instance:
```
gcloud compute instances describe source-instance --zone=$ZONE | grep natIP
```
Example Output:
```
natIP: 35.192.107.237
```
Copy the `<nat_IP>` address and save it to use in later steps.

Authorize your external address range:
```
gcloud container clusters update private-cluster \
    --enable-master-authorized-networks \
    --master-authorized-networks [MY_EXTERNAL_RANGE]
```
*Note: In a production environment replace `[MY_EXTERNAL_RANGE]` with your network external address CIDR range.*

Now that you have access to the master from a range of external addresses, you'll install `kubectl` so you can use it to get information about your cluster.

SSH into source-instance:
```
gcloud compute ssh source-instance --zone=$ZONE
```
Press `Y` to continue. Enter through the passphrase questions.

In SSH shell, install `kubectl` component of Cloud-SDK:
```
sudo apt-get install kubectl
```

Configure access to the Kubernetes cluster from SSH shell:
```
sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
gcloud container clusters get-credentials private-cluster --zone=$ZONE
```
*Note: Please make sure that the assigned zone has been exported in the `ZONE` variable.*

Verify that your cluster nodes do not have external IP addresses:
```
kubectl get nodes --output yaml | grep -A4 addresses
```
The output shows that the nodes have internal IP addresses but do not have external addresses:
```
...
addresses:
- address: 10.0.0.4
  type: InternalIP
- address: ""
  type: ExternalIP
...
```
Here is another command you can use to verify that your nodes do not have external IP addresses:
```
kubectl get nodes --output wide
```
The output shows an empty column for `EXTERNAL-IP`:
```
STATUS ... VERSION        EXTERNAL-IP   OS-IMAGE ...
Ready      v1.8.7-gke.1                 Container-Optimized OS from Google
Ready      v1.8.7-gke.1                 Container-Optimized OS from Google
Ready      v1.8.7-gke.1                 Container-Optimized OS from Google
```
Close the SSH shell by typing:
```
exit
```

## Task 5. Clean Up

Delete the Kubernetes cluster:
```
gcloud container clusters delete private-cluster --zone=$ZONE
```
Press `Y` to continue.

## Task 6. Create a Private Cluster that Uses a Custom Subnetwork

In the previous section, Kubernetes Engine automatically created a subnetwork for you. In this section, you'll create your own custom subnetwork, and then create a private cluster. Your subnetwork has a primary address range and two secondary address ranges.

Create a subnetwork and secondary ranges:
```
gcloud compute networks subnets create my-subnet \
    --network default \
    --range 10.0.4.0/22 \
    --enable-private-ip-google-access \
    --region=$REGION \
    --secondary-range my-svc-range=10.0.32.0/20,my-pod-range=10.4.0.0/14
```

Create a private cluster that uses your subnetwork:
```
gcloud beta container clusters create private-cluster2 \
    --enable-private-nodes \
    --enable-ip-alias \
    --master-ipv4-cidr 172.16.0.32/28 \
    --subnetwork my-subnet \
    --services-secondary-range-name my-svc-range \
    --cluster-secondary-range-name my-pod-range \
    --zone=$ZONE
```

Retrieve the external address range of the source instance:
```
gcloud compute instances describe source-instance --zone=$ZONE | grep natIP
```
Example Output:
```
natIP: 35.222.210.67
```
Copy the `<nat_IP>` address and save it to use in later steps.

Authorize your external address range:
```
gcloud container clusters update private-cluster2 \
    --enable-master-authorized-networks \
    --zone=$ZONE \
    --master-authorized-networks [MY_EXTERNAL_RANGE]
```

SSH into source-instance:
```
gcloud compute ssh source-instance --zone=$ZONE
```

Configure access to the Kubernetes cluster from SSH shell:
```
gcloud container clusters get-credentials private-cluster2 --zone=$ZONE
```
*Note: Please make sure that the assigned zone has been exported in the `ZONE` variable.*

Verify that your cluster nodes do not have external IP addresses:
```
kubectl get nodes --output yaml | grep -A4 addresses
```
The output shows that the nodes have internal IP addresses but do not have external addresses:
```
...
addresses:
- address: 10.0.4.3
  type: InternalIP
...
```

At this point, the only IP addresses that have access to the master are the addresses in these ranges:
- The primary range of your subnetwork. This is the range used for nodes. In this example, the range for nodes is `10.0.4.0/22`.
- The secondary range of your subnetwork that is used for pods. In this example, the range for pods is `10.4.0.0/14`.

Congratulations! You learned how to create a private Kubernetes cluster.