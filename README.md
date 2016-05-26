Background
----------

*terms*
docker (ubuntu) or Rocket (coreOS) = a container technology to allow developers to package apps in linux containers. operating-system-level virtualization rather than hyperV hardware virtualization.

scheduling (or packaging) = placing a single container on a single host based on the needs of the container (ie container needs 2Gm memory)

Orchestration =  that an application topology is deployed across two or more hosts in such a way as to meet the business needs for the application, and deployment cycles (HA etc)

Fleet, by CoreOs, is a packing primitive.  It is somewhat analogous to Docker's Swarm

Mesosphere & Kubernetes = a Docker container management solution. Once specific containers are no longer bound to specific machines, host-centric infrastructure no longer works: managed groups, load balancing, auto-scaling, etc. One needs container-centric infrastructure. That’s what Kubernetes provides.

*kubernetes* 
  Kubernetes is unopinionated in the source-to-image space, but support layering CI workflows on Kubernetes 
  a number of PaaS systems run on Kubernetes, such as Openshift, Deis, and Gondor, or roll your own.
  
Deis PaaS v1 = Fleet -> v2 = kubernetes
 
*systemd* is an init system that provides many powerful features for starting, stopping and managing processes. Within the CoreOS world, you will almost exclusively use systemd to manage the lifecycle of your Docker containers
On CoreOS, unit files are located  /etc/systemd/system

*etcd*: A highly-available distributed key value store for shared configuration and service discovery written in Go. Written by x-heroku guy, it Serves as the backbone of distributed systems by providing a canonical hub for cluster coordination and state management, used by Kubermetes & Cloud Foundry & Fleet. Uses the _Raft_ protocol for multiple nodes to maintain identical logs of state changing commands where any node can be treated as master.

For a group of CoreOS machines to form a cluster, their etcd instances need to be connected. A discovery service, https://discovery.etcd.io, is provided as a free service to help connect etcd instances together by storing a list of peer addresses, metadata and the initial size of the cluster under a unique address, known as the discovery URL - You can generate them very easily:
 curl -w "\n" 'https://discovery.etcd.io/new?size=2'
 
 
*docker* Containers can boot extremely fast (in milliseconds!) which gives you unprecedented flexibility in managing load across your cluster. They are easier to build than VMs, and because they are decoupled from the underlying infrastructure. For example, instead of running chef on each of your VMs, it’s faster and more reliable to have your build system create a container, When these containers start, they can signal your proxy (via etcd) to start sending them traffic. Immutable container images can be created at build/release time, enables a consistent environment to be carried from development into production

A container is a stripped-to-basics version of a Linux operating system. An image is software you load into a container.

 The docker deamon 1. pull program image from dockerhub (or local), (2) creates a container for the image & runs it (3) stream the stdout toe docker client
 
 > docker run image # runs the image
 > docker images # list images on the local m/c
 > create Dockerfile > 
    FROM node:latest           # define image we want to build from (image with node/npm)
    RUN mkdir -p /usr/src/app  # create directory to hold the application inside the image
    WORKDIR /usr/src/app
    COPY package.json /usr/src/app/ # copy dependencies file and resolve
    RUN npm install
    COPY . /usr/src/app     # copy application code
    EXPOSE 4000             # export port
    CMD [ "npm", "start" ]  # launch command



 > docker build -t docker-whale # build a new image (-t tag image)
 > docker run -p 49160:4000 -d <your username>/node-web-app
 
 > docker [ps|logs|exec]
 

*fleet* fleet is a cluster manager that controls systemd at the cluster level, you can treat your CoreOS cluster as if it shared a single init system, It encourages users to write applications as small, ephemeral units that can easily migrate around a cluster of self-updating CoreOS machines

Deploying fleet is as simple as dropping the fleetd binary on a machine with access to etcd and starting it. Deploying fleet on CoreOS is even simpler: just run systemctl start fleet.  The built-in configuration assumes each of your hosts is serving an etcd endpoint at one of the default locations (http://127.0.0.1:4001 or http://127.0.0.1:2379). 

The recommended way to run docker containers on your CoreOS is with *fleet*, a tool that presents your entire cluster as a single init system. fleet works by receiving systemd unit files and scheduling them onto machines in the cluster based on declared conflicts and other preferences encoded in the unit file. With one command, fleet can start many instances of a container across the entire cluster, without requiring a complex deployment script or manual assignment of services to specific machines

> fleetctl list-machines
MACHINE         IP              METADATA
53fd2c09...     10.1.0.6        -
98683544...     10.1.0.5        -
> fleetctl list-unit-files
> fleetctl list-units  (running apps)

*ssh keys*
https://help.ubuntu.com/community/SSH/OpenSSH/Keys
Public key authentication is more secure than password authentication. This is particularly important if the computer is visible on the internet.
The authenticating entity has a public key and a private key,  The private key is kept on the computer you log in from, while the public key is stored on the .ssh/authorized_keys file on all the computers you want to log in to, the SSH server uses the public key to "lock" messages in a way that can only be "unlocked" by your private key 

Generating public/private rsa key pair:
> ssh-keygen -t rsa
	~/.ssh/id_rsa # identification
	~/.ssh/id_rsa.pub # public key



 
CoreOS VM "osProfile"."customData" property
--------------------------------------------

CoreOS Cloud-Config “the defacto multi-distribution package that handles early initialization of a cloud instance”
CoreOS allows you to declaratively customize various OS-level items, such as network configuration, user accounts, and systemd units. This document describes the full list of items we can configure. The coreos-cloudinit program uses these files as it configures the OS after startup or during runtime.

the Azure waagent buts this in : /var/lib/waagent/CustomData
the agent log is in : /var/log/waagnt.log

to run manually: sudo coreos-cloudinit --from-file=/var/lib/waagent/CustomData

writes to : /run/systemd/system/etcd2.service.d/20-cloudinit.conf

debugging:
systemctl status -l etcd2
systemctl status -l fleet
journalctl -xe
journalctl -u etcd2



install cluster
---------------


https://github.com/deis/workflow/blob/master/src/quickstart/index.md

curl -w "\n" 'https://discovery.etcd.io/new?size=2'
New-AzureRmResourceGroupDeployment -ResourceGroupName deisv2 -TemplateFile .\etc2dcluster_vmss.json -TemplateParameterFile .\params.json
set "discoveryurl" to result of curl

install kubernetes
http://kubernetes.io/docs/getting-started-guides/binary_release/

export KUBERNETES_PROVIDER=azure
export AZURE_SUBSCRIPTION_ID=95efa97a-9b5d-4f74-9f75-a3396e23344d
export AZURE_LOCATION=northeurope
export AZURE_TENANT_ID=72f988bf-86f1-41af-91ab-2d7cd011db47

wget -q -O - https://get.k8s.io | bash
## runs ./kubenetes/cluster/azure/get-kube.sh
  ## runs ./cluster/kube-up.sh
     ## runs ./cluster/azure/util.sh (function kube-up)
     
       ## deploys ARM template
     azure-deploy

	## creates ~/.kube/config
    kubectl config set-cluster "${AZURE_DEPLOY_ID}" --server="https://${AZURE_DEPLOY_ID}.${AZURE_LOCATION}.cloudapp.azure.com:6443" --certificate-authority="${AZURE_OUTPUT_DIR}/ca.crt" --api-version="v1"
    kubectl config set-credentials "${AZURE_DEPLOY_ID}_user" --client-certificate="${AZURE_OUTPUT_DIR}/client.crt" --client-key="${AZURE_OUTPUT_DIR}/client.key"
    kubectl config set-context "${AZURE_DEPLOY_ID}" --cluster="${AZURE_DEPLOY_ID}" --user="${AZURE_DEPLOY_ID}_user"
    kubectl config use-context "${AZURE_DEPLOY_ID}"

	## adds etcd2 nodes
    > ./validate-cluster.sh    


> kubectl cluster-info


https://github.com/deis/workflow/blob/master/src/installing-workflow/index.md

> curl -sSL https://get.helm.sh | bash
> sudo mv ./helmc /usr/local/sbin

> helmc target
> helmc repo add deis https://github.com/deis/charts
> helmc fetch deis/workflow-beta4             # fetches the chart into a
                                              # local workspace
> helmc generate -x manifests workflow-beta4  # generates various secrets
> helmc install workflow-beta4  

> kubectl get pods  --namespace=deis

> kubectl get services --namespace=deis
> kubectl get services deis-controller --namespace=deis
> kubectl describe services  deis-controller --namespace=deis


https://github.com/deis/workflow/blob/master/src/quickstart/deploy-an-app.md

deploy app

> deis register http://deis.104.197.125.75.nip.io
