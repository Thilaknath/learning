## Learning Objectives

* Create Compute Engine instances
* Create instance templates from source instances
* Create managed instance groups
* Create and test managed instance group health checks
* Create HTTP(S) Load Balancers
* Create load balancer health checks
* Use a Content Delivery Network (CDN) for Caching

## Create Compute Engine instances
Refer startup-script.sh

The startup script performs the following tasks:

* Installs the Logging agent. The agent automatically collects logs from syslog.
* Installs Node.js and Supervisor. Supervisor runs the app as a daemon.
* Clones the app's source code from Cloud Storage Bucket and installs dependencies.
* Configures Supervisor to run the app. Supervisor makes sure the app is restarted if it exits unexpectedly or is stopped by an admin or process. It also sends the app's stdout and stderr to syslog for the Logging agent to collect.

## Deploy the backend instance
'''
gcloud compute instances create backend \
    --zone=$ZONE \
    --machine-type=e2-standard-2 \
    --tags=backend \
   --metadata=startup-script-url=https://storage.googleapis.com/fancy-store-$DEVSHELL_PROJECT_ID/startup-script.sh
'''

## Configure Network
'''
gcloud compute firewall-rules create fw-fe \
    --allow tcp:8080 \
    --target-tags=frontend
'''

'''
gcloud compute firewall-rules create fw-be \
    --allow tcp:8081-8082 \
    --target-tags=backend
'''

## Autohealing 
Note: Separate health checks for load balancing and for autohealing will be used. Health checks for load balancing can and should be more aggressive because these health checks determine whether an instance receives user traffic. You want to catch non-responsive instances quickly so you can redirect traffic if necessary.
In contrast, health checking for autohealing causes Compute Engine to proactively replace failing instances, so this health check should be more conservative than a load balancing health check.

## Load Balancing
Google Cloud offers many different types of load balancers. For this lab you use an HTTP(S) Load Balancer for your traffic. An HTTP load balancer is structured as follows:

A forwarding rule directs incoming requests to a target HTTP proxy.
The target HTTP proxy checks each request against a URL map to determine the appropriate backend service for the request.
The backend service directs each request to an appropriate backend based on serving capacity, zone, and instance health of its attached backends. The health of each backend instance is verified using an HTTP health check. If the backend service is configured to use an HTTPS or HTTP/2 health check, the request will be encrypted on its way to the backend instance.
Sessions between the load balancer and the instance can use the HTTP, HTTPS, or HTTP/2 protocol. If you use HTTPS or HTTP/2, each instance in the backend services must have an SSL certificate.