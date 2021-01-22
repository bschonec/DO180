# Notes I've taken in order to study for Introduction to Containers, Kubernetes and Red Hat OpenShift

I have not taken the EX447 exam.  I am bound by a non discloure agreement and cannot and will not answer any questions about any Red Hat Exam.  These are notes that I've taken while studying for the exam.



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

# Chapter 4:  Managing Container Images


podman save -o <file> image[:TAG} # Saves image/tag to a tarball.
podman rmi <image>  # remove local image
podman rm <container> # remove non-running container instance

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
oc exec
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


