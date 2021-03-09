# Notes I've taken in order to study for Introduction to Containers, Kubernetes and Red Hat OpenShift

I have not taken the DO180 exam.  I am bound by a non discloure agreement and cannot and will not answer any questions about any Red Hat Exam.  These are notes that I've taken while studying for the exam.


# Misc

insert into Projects (id, name) values (1, "devops");

# Chapter 1:  Introducing Container Technology

Namespaces:  responsible for ensuring that only specific resources are able to see certain processes.  ISOLATION
Control Groups:  responsible for limiting how many resources processes are allowed to use.
Seccomp:  limits how processes uses system calls, whitelisting system calls, parameters and file descriptors.
SELinux:  mandatory access control.  Processes can only interact with resources that belong to the same users or groups.


image:  A file system bundle that contains all dependencies required to execute a process.
        - files
        - packages
        - processes




sudo podman images # shows images stored locally
sudo podman pull <image> # pulls image from  repository
sudo podman run <image> # starts 'latest' container from image
sudo podman run <image>:<version> # starts a particular version
sudo podman search <text> # search for an image

podman inspect -l -f "{{ something interesting to query }}"

--name  <name>
-i, --interactive  # interactive
-d, --detach  # detach
-t, --tty # interactive

## Selinux

semanage fcontext -a -t container_file_t '/var/local/mysql(/.*)?'  #\*
restorecon -RV /var/local/mysql

## Mapping Network Ports

sudo podman run -d --name apache -p 8080:80 httpd:2.4

# What port translations is running on a container?
sudo podman port <container name>
80/tcp - > 127.0.0.1:35123  # Port 35123 on the host will be forwarded to 80 on the container.

# Chapter 3

podman ps --formt="{{ .ID }} {{ .Names }} {{ .Status }}"

podman inspect -f '{{ .NetworkSettings.IPAddress }}' <container>


# Execute a command in a running container
sudo podman exec mysql /bin/bash -c 'mysql -uuser1 -pmypa55 -e "select * from items.Projects;"'



# Chapter 4:  Managing Container Images


podman save -o <file> image[:TAG} # Saves image/tag to a tarball.
podman rmi <image>  # remove local image
podman rm <container> # remove non-running container instance

podman build -t mytag --build-arg NEXUS_BASE_URL=https://nexus.io/bschonec

podman commit <container> [repository]<image name>:[TAG]

podman diff <image>  # Shows what changes have been made compared to the original image.
# "A" for added files, "C" for changed files, "D" for deleted files.

podman inspect -f "{{ query }}"

podman tag [options] <image>:[TAG] <name of image>
# podman automatically adds 'latest' when you don't specify a tag

podman push <image>  # pushes image to registrys /etc/containers/regsitries.conf


Stop, commit, tag, push


# Chapter 5:  Creating Custom Container Images

First command MUST be a FROM command.
Commands are not case sensitive.


's2i' package = source to image.  Installed via YUM.

# Chapter 6:  Deploying Containerized Applications on OpenShift
Kubernetes is an orchestration service that simplifies deployment, management and scaling of containerized applications.

Node:  Server that hosts applications in a kubernetes cluster
  - master node:  responsible for managing all other nodes.  (control plane)
  - worker/compute node:  where the applications live
  - infra node:  responsible for monitoring, logging or external routing.
Resources:  pod, service, route.  Any kind of component definition managed by kubernetes
Controller:  Kubernetes process that watches resources.  Responsible for maintaning a desired state
Label:  key-value pair that can be assigned to any kubernetes resource
Namespace:  A bundle of resources and processes.

Persistent volume: defines storage
persistent volume claim:
ConfigMaps and secrets: 


sudo oc expose <service name>  # creates a route to the service
oc exec pod -- command  # same as 'podman exec'
oc export
:q
which oc


## Creating Kubernetes Resources

sudo oc login <clusterurl>


sudo oc port-forward mysql-openshift 3306:3306 # forwards port from the developer machine to port 3306 on the DB pod where mysql container is running.

sudo oc new-app   # create a new application that containes resources

sudo oc new-app --as-deployment-config -docker-image=registry/image:latest --name=<name> -e VAR -e VAR2=xx 
sudo oc get all # retrieve the most important components of a cluster
sudo oc get pods
sudo oc status
oc expose <service name> # create an external route to the service

oc port-forward <service> <port>:<port>
oc create -f <file of resource definition>

podman cp standalone.conf <container>:/opt/jboss/conf.d
podman cp <container>:/opt/jbos/conf.d/standalone.conf .

oc describe template <name> -n openshift
        
   
# Chapter 7: Deploying Multi-Container Applications

oc get templates -n openshift # shows preinstalled templates
oc process -o yaml -f <filename template> -p PARAMETER1 -p PARAMETER2 > myapp.yaml
        
 oc process -o yaml -f mysql.yaml -p MYSQL_USER=dev -P MYSQL_PASSWORD=$P4SSD -p MYSQL_DATABASE=bank -p VOLUME_CAPACITY=10Gi > mysqlProcessed.yml

oc ## upload template


## Environment variables created when a service is created

eg: service name = mysql

MYSQL_SERVICE_HOST=10.0.0.11    # <service name> _SERVICE_HOST
MYSQL_SERVICE_PORT=3306         # <service name> _SERVICE_PORT
MYSQL_PORT=tcp://10.0.0.11:3306
MYSQL_PORT_3306_TCP=tcp://10.0.0.11:3306
MYSQL_PORT_3306_TCP_PROTO=tcp
MYSQL_PORT_3306_TCP_PORT=3306
MYSQL_PORT_3306_TCP_ADDR=10.0.0.11

# Show parameters that the template expects.
oc process --parameters mysql-persistent -n openshift


"oc process" is a two step process.  oc process -o yaml -f <filename>  # process the template and output toyaml


podman run --ip <static IP of container>

# Chapter 8:  Troubleshooting Containerized Applications

Ensure container_file_t SELinux context is set for any local volumes attached to a container.


oc delete pv <volume>  # Delete persistent volume
oc create -f <pv_claim> # recreate the persistent volume clean


oc get bc
oc logs bc/<buildconfig>
oc expose service <service>
oc get routes

oc port-forward db 30306 3306 # port forwards 30303 into pod "db" to 3306

oc get events
oc describe pod mysql


podma exec [options]

oc exec -it pod -c <container> 

oc exec -it myhttpdpod /bin/bash

podman cp <local file> <container>:/path/to/file
podman cp /etc/hosts ubi7:/etc/hosts


