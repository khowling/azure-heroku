systemd is an init system that provides many powerful features for starting, stopping and managing processes. Within the CoreOS world, you will almost exclusively use systemd to manage the lifecycle of your Docker containers
On CoreOS, unit files are located within the R/W filesystem at /etc/systemd/system

etcd: A highly-available distributed key value store for shared configuration and service discovery written in Go. Written by x-heroku guy, it Serves as the backbone of distributed systems by providing a canonical hub for cluster coordination and state management, used by Kubermetes & Cloud Foundry & Fleet. Uses the _Raft_ protocol for multiple nodes to maintain identical logs of state changing commands where any node can be treated as master.

For a group of CoreOS machines to form a cluster, their etcd instances need to be connected. A discovery service, https://discovery.etcd.io, is provided as a free service to help connect etcd instances together by storing a list of peer addresses, metadata and the initial size of the cluster under a unique address, known as the discovery URL - You can generate them very easily:
 curl -w "\n" 'https://discovery.etcd.io/new?size=2'
 
 


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



Your cloud-config YAML file is processed during each boot
	etcd2 : will generate a systemd unit drop-in for etcd.service 

master-config.yaml: cloud-config for the Kubernetes master components
	Kubectl API server & scheduler
node-config.yaml: cloud-config for each Kubernetes node
	has the services necessary to run application containers and be managed from the master systems
	kubelet  manages pods and their containers, their images, their volumes
	kube-proxy : runs a simple network proxy and load balancer 
	runs Docker

ssh keys
https://help.ubuntu.com/community/SSH/OpenSSH/Keys
Public key authentication is more secure than password authentication. This is particularly important if the computer is visible on the internet.
The authenticating entity has a public key and a private key,  The private key is kept on the computer you log in from, while the public key is stored on the .ssh/authorized_keys file on all the computers you want to log in to, the SSH server uses the public key to "lock" messages in a way that can only be "unlocked" by your private key 

Generating public/private rsa key pair:
ssh-keygen -t rsa
	~/.ssh/id_rsa # identification
	~/.ssh/id_rsa.pub # public key

openssl x509: true, 
102     nodes: true, 
103     newkey: 'rsa:2048', 
104     subj: '/O=Weaveworks, Inc./L=London/C=GB/CN=weave.works', 
out ssh.pem
keyout: ssh.key

openssl req
openssl rsa keyin = keyout out: opts.keyout



AZ_SUBSCRIPTION=95efa97a-9b5d-4f74-9f75-a3396e23344d ./create-kubernetes-cluster.js


azure.queue_default_network(), 
	'network', 'vnet', 'create',   '--address-space=172.16.0.0',
azure.queue_storage_if_needed(), 
	'storage', 'account', 'create', '--type=LRS',
azure.queue_machines('etcd', 'stable',  kube.create_etcd_cloud_config),

	kube.create_etcd_cloud_config
		var input_file = './cloud_config_templates/kubernetes-cluster-etcd-node-template.yml'; 
		var output_file = util.join_output_file_path('kubernetes-cluster-etcd-nodes', 'generated.yml'); 

	

	'vm', 'create',
	'--ssh-cert=' + conf.resources['ssh_key']['pem']
	--ssh=<%= port %>
        --custom-data=<%= cloud_config_file %>

	hosts.ssh_port_counter += 1;
kube.create_etcd_cloud_config), 

azure.queue_machines('kube', 'stable', 

kube.create_node_cloud_config), 

-------------

https://github.com/Azure/azure-quickstart-templates/blob/master/coreos-with-fleet-multivm/azuredeploy.json

#cloud-config
coreos:
  etcd2:
    discovery: discoveryUrl,
    advertise-client-urls: http://$private_ipv4:2379,http://$private_ipv4:4001
    initial-advertise-peer-urls: http://$private_ipv4:2380
    listen-client-urls: http://0.0.0.0:2379,http://0.0.0.0:4001
    listen-peer-urls: http://$private_ipv4:2380
  units:
    - name: etcd2.service
      command: start
    - name: fleet.service
      command: start'

~/kubernetes/docs/getting-started-guides/coreos/azure/cloud_config_templates

coreos:
  units:
    - name: etcd2.service
      enable: true
      command: start
  etcd2:
    name: '%H'
    initial-cluster-token: 'etcd-cluster'
    initial-advertise-peer-urls: 'http://%H:2380'
    listen-peer-urls: 'http://%H:2380'
    listen-client-urls: 'http://0.0.0.0:2379,http://0.0.0.0:4001'
    advertise-client-urls: 'http://%H:2379,http://%H:4001'
    initial-cluster-state: 'new'
  update:
    group: stable
    reboot-strategy: off