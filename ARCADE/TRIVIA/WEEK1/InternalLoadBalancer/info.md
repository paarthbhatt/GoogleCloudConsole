# Google Cloud DLP API - Quickstart Guide

**Date:** 2025-02-23  
**User:** paarthbhatt

## Task 1: Create a Virtual Environment

Python virtual environments are used to isolate package installations from the system.

### Step 1: Install the virtualenv environment
```sh
sudo apt-get install -y virtualenv
```

### Step 2: Build the virtual environment
```sh
python3 -m venv venv
```

### Step 3: Activate the virtual environment
```sh
source venv/bin/activate
```

## Task 2: Create Backend Managed Instance Group

### Step 1: Create the Startup Script

Create the `backend.sh` script in the home directory:
```sh
touch ~/backend.sh
```

Open the file in an editor and add the following script:
````bash name=backend.sh
sudo chmod -R 777 /usr/local/sbin/
sudo cat << EOF > /usr/local/sbin/serveprimes.py
import http.server

def is_prime(a): return a!=1 and all(a % i for i in range(2,int(a**0.5)+1))

class myHandler(http.server.BaseHTTPRequestHandler):
  def do_GET(s):
    s.send_response(200)
    s.send_header("Content-type", "text/plain")
    s.end_headers()
    s.wfile.write(bytes(str(is_prime(int(s.path[1:]))).encode('utf-8')))

http.server.HTTPServer(("",80),myHandler).serve_forever()
EOF
nohup python3 /usr/local/sbin/serveprimes.py >/dev/null 2>&1 &

## Task 3: Setup Internal Load Balancer

### Step 1: Create a Health Check

```sh
gcloud compute health-checks create http ilb-health --request-path /2
```

### Step 2: Create a Backend Service

```sh
gcloud compute backend-services create prime-service \
--load-balancing-scheme internal --region=$REGION \
--protocol tcp --health-checks ilb-health
```

### Step 3: Add the Instance Group

```sh
gcloud compute backend-services add-backend prime-service \
--instance-group backend --instance-group-zone=$ZONE \
--region=$REGION
```

### Step 4: Create the Forwarding Rule

```sh
gcloud compute forwarding-rules create prime-lb \
--load-balancing-scheme internal \
--ports 80 --network default \
--region=$REGION --address 10.132.0.10 \
--backend-service prime-service
```

## Task 4: Test the Load Balancer

### Step 1: Create a New Instance

```sh
gcloud compute instances create testinstance \
--machine-type=e2-standard-2 --zone $ZONE
```

### Step 2: SSH into the Instance

```sh
gcloud compute ssh testinstance --zone $ZONE
```

### Step 3: Query the Load Balancer

```sh
curl 10.132.0.10/2
curl 10.132.0.10/4
curl 10.132.0.10/5
```

### Step 4: Exit and Delete the Test Instance

```sh
exit
gcloud compute instances delete testinstance --zone=$ZONE
```

## Task 5: Create a Public Facing Web Server

### Step 1: Create the Startup Script

Create the `frontend.sh` script in the home directory:
```sh
touch ~/frontend.sh
```

Open the file in an editor and add the following script:
````bash name=frontend.sh
sudo chmod -R 777 /usr/local/sbin/
sudo cat << EOF > /usr/local/sbin/getprimes.py
import urllib.request
from multiprocessing.dummy import Pool as ThreadPool
import http.server
PREFIX="http://10.132.0.10/" #HTTP Load Balancer
def get_url(number):
    return urllib.request.urlopen(PREFIX+str(number)).read().decode('utf-8')
class myHandler(http.server.BaseHTTPRequestHandler):
  def do_GET(s):
    s.send_response(200)
    s.send_header("Content-type", "text/html")
    s.end_headers()
    i = int(s.path[1:]) if (len(s.path)>1) else 1
    s.wfile.write("<html><body><table>".encode('utf-8'))
    pool = ThreadPool(10)
    results = pool.map(get_url,range(i,i+100))
    for x in range(0,100):
      if not (x % 10): s.wfile.write("<tr>".encode('utf-8'))
      if results[x]=="True":
        s.wfile.write("<td bgcolor='#00ff00'>".encode('utf-8'))
      else:
        s.wfile.write("<td bgcolor='#ff0000'>".encode('utf-8'))
      s.wfile.write(str(x+i).encode('utf-8')+"</td> ".encode('utf-8'))
      if not ((x+1) % 10): s.wfile.write("</tr>".encode('utf-8'))
    s.wfile.write("</table></body></html>".encode('utf-8'))
http.server.HTTPServer(("",80),myHandler).serve_forever()
EOF
nohup python3 /usr/local/sbin/getprimes.py >/dev/null 2>&1 &

## Task 6: Inspect a string for sensitive information

This section shows you how to ask the service to scan sample text using the `projects.content.inspect` REST method. The JSON file you create contains an `InspectConfig` and a `ContentItem` object.

### Step 1: Create a JSON request file

Using your preferred editor (nano, vim, etc.) or Cloud Shell, create a JSON request file with the following text, and save it as `inspect-request.json`:

````json name=inspect-request.json
{
  "item": {
    "value": "My phone number is (206) 555-0123."
  },
  "inspectConfig": {
    "infoTypes": [
      {
        "name": "PHONE_NUMBER"
      },
      {
        "name": "US_TOLLFREE_PHONE_NUMBER"
      }
    ],
    "minLikelihood": "POSSIBLE",
    "limits": {
      "maxFindingsPerItem": 0
    },
    "includeQuote": true
  }
}