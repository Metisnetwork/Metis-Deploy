# Datum  Deployment [[中文](./README.md)]

## Datum Network Topology

The datum network consists of multiple Datum-network Nodes, each of which is actually a logical general term for an organization. The network topology of the Datum-network Node is as follows:

- ### *Network topology between Datum-network Nodes*

![organizations][OrganizationNetworkTopologyOfDatumNetworkNode]

The network topology between Datum-network Nodes (that is, between organizations) is connected to each other through a p2p network; the carrier of each Datum-network Node is linked with the carrier of the Datum-network Nodes outside for p2p connection.

- ### *Network topology of each service within a single Datum-network Node*

![inside organization][InternalServiceNetworkTopologyOfASingleDatumNetworkNode]

For each service in the closed beta test of Datum-network Node, there are such role services as admin, Glacier2, IceGrid, carrier, fighter(data), fighter(compute), and consul. Admin, carrier, Glacier2, IceGrid, and fighter all have their information automatically registered to consul, and each internal service discovers each other through other internal service information in consul. The specific functions of each service within the organization are as follows:

**ice_via node**: The ICE VIA nodes include Glacier2 and IceGrid services. Glacier2 can be used as a router, with the function of penetrating the firewall, and forwarding client requests to the server through Glacier2; IceGrid can be used as a registrar to register server-related information and services with IceGrid services; Glacier2 services and IceGrid services provide organizational and external The only external network port for multi-party assistance tasks (the internal and external network ports are the same).

**carrier node**: It provides task scheduling services for the entire organization. An organization can deploy (only) one scheduling service. Carrier acts as the brain of the entire Datum-network Node and is responsible for task scheduling, resource service scheduling, metadata and internal resource information management within the organization, as well as data synchronization, p2p, etc.

**admin node**: It provides the web backend service for the management console of the entire organization. An organization has one management console service (there can be multiple, and it is advised to start one). Admin is the management console of the carrier, which is convenient for users to manage internal data, resources and data statistics reports.

**data node**: [fighter(data)] is the storage resource node of the entire organization, and you can configure as many as you like.

**compute node**: [figter(compute)] is the computing resource node of the entire organization, and you can configure as many as you like.

**consul node**: It serves the registry of the entire organization. An organization is advised to configure an odd number (1, 3, 5, etc.) of consul nodes to facilitate the raft consensus algorithm to select leaders.

- ### *Services that must exist inside Datum-network Node and the order in which they are deployed*

> From the above figure, we get to know the network topology of each service inside Datum-network Node. It is thus clear that consul, carrier, and admin are the services required in the organization, and other services are deployed as needed; if you need to provide data capabilities or computing services for multi-party collaborative computing, you must also have ice_via(Glacier2/IceGrid) and fighter. In the former case, you need to deploy fighter (data), and in the latter case, you need to deploy fighter (compute).

**The order of service deployment:**

> (1) Services that must be deployed (whether involved in task computing or not): [1] consul service -> [2] carrier service -> [3] admin service.
> 
> (2) Optional services (only needed for task computing): [4] ice_via service(Glacier2/IceGrid) -> [5] fighter(data) service and fighter(compute) service.

- ### *Relationship between DatumPlatform and Datum-network Node*

![DatumPlatform and DatumNetworkNode][TopologyBetweenDatumPlatformAndDatumNetworkNode]

DatumPlatform can be seen as a platform that provides a data market, network-wide data statistics, and task workflow management. It can be built within your own organization, in which case it's in a one-to-one relationship with the organization's Datum-network Node; you can also add the carrier extranet ip and port of an external Datum-network Node through the DatumPlatform interface to connect to other external Datum-network Nodes, which means a one-to-many relationship. For instructions of the specific operation of DatumPlatform, please refer to the relevant documentation of DatumPlatform.

## Description of Datum-network Node Deployment Script

This script is an ansible automated deployment script, including admin, ice_via, carrier, fighter(data), fighter(compute), and consul role services within a single organization. The ansible script can **deploy, start, stop , and destroy** each role node.

## Environment Requirements

The master computer (where ansible issues commands) and the target computer (where each service is deployed).

1. The stable version of ansible master and target computers only support the Ubuntu 18.04 operating system.

2. The processor architecture only supports processors of the x86_64 architecture (required by the target computer only).

3. Suggested Host Resource Requirements

|Service Name|Machine Configuration Recommendations|
|---|---|
|ice_via(Glacier2/IceGrid)|4C, 8G memory, 200GB high-efficiency cloud disk|
|Fighter|8C, 16G memory, 200GB high-efficiency cloud disk|
|Carrier|8C, 16G memory, 200GB high-efficiency cloud disk|
|Admin (self-built MySQL) + Consul|8C, 16G memory, 200GB high-efficiency cloud disk|


### Preparation before Deployment

1. *Download the script on the master computer:* 


```shell
#Download the script project using git
git clone https://github.com/datumtechs/datum-network-deploy.git

#Enter the project directory
cd datum-network-deploy

#Switch project to the ansible branch
git checkout ansible

#Create a log directory
mkdir log
```

*2. Install the specified version of `ansible` and `dependency module` on the master computer, and execute the command (if ansible has been installed, please uninstall it first):*

```shell
# First enter the project directory
cd datum-network-deploy

# Install python-related tools (python2.7 or python3.6 by default in Ubuntu 18.04)
sudo apt install -y python python-pip python3 python3-pip

# Install ansible and dependencies
pip install -r ./requirements.txt
```

*3. Check the master computer environment and download the installation package:*

```shell

#  Check the ansible version (>2.4.2, 2.7.11 recommended), and check jinja2 installation and version (>=2.9.6) information.
#  Create a download directory and check the network connection. If you cannot connect to the extranet, an error will be reported and you will exit directly. Access to the extranet is required to download the installation package.
# Download dependencies and components (sudo password is required).
# Download the installation package to the download directory (go binary file, jar package, web static resource file, python whl file, and shell script). You can try downloading for three times at most, and the delay of each attempt is a random value within 10.

ansible-playbook --ask-sudo-pass local_prepare.yml
```

## Introduction to the Functions of Each File of the Project

inventory.ini: The host inventory file, the host ip (intranet) and variables of each service in the organization, defined by the user as needed.

group_vars/all.yml: Public variables. Modification is not recommended.

ansible.cfg: The ansible configuration file. Operation should be prudent. 

config directory: The configuration file. Modification is not recommended.

roles directory: A set of ansible roles. Operation should be prudent.

local_prepare.yml: The script for the master computer to perform tasks such as environment checking and installation package downloadng. Operation should be prudent.

bootstrap.yml: The script for initializing tasks such as each node of the cluster. Operation should be prudent.

deploy.yml: The script to install each service. Operation should be prudent.

start.yml: The script to start all services. Operation should be prudent.

stop.yml: The script to stop all services. Operation should be prudent.

cleanup.yml: The script for destroying the cluster. Operation should be prudent.

## The Host Inventory File `inventory.ini` in A Single Organization

The host inventory file `inventory.ini` configures the deployment host IP (intranet) and variables.

Configurations come in two common forms:

[1] All services are deployed on one machine (ie: a single host deploys all services of a single organization, which is recommended only in the testing phase).

[2] Only one service is deployed for each host (recommended in the production phase).

At present, the remote host ssh login does not support the root user but ordinary users only, and the ordinary users need to support sudo privilege escalation.

> To simplify the operation and avoid entering a password when executing the playbook, you can configure `ansible_ssh_user` as the ssh username to log in to the target host, `ansible_ssh_pass` as the user's ssh password, and `ansible_sudo_pass` as the password for privilege escalation on the target host. You can also skip the above configuration, and enter the password when executing the playbook.

### The `inventory.ini` File Template

```ini
# Inventory file, mainly to configure the host list and host group

# Task gateway. An organization has a gateway service
[ice_via]
${intranet IP of target computer} ansible_ssh_user="${ ssh account name }" ansible_ssh_pass="${ ssh account password }" ansible_sudo_pass="${ sudo privilege escalation password }"

# Scheduling. An organization has a scheduling service
[carrier]
${intranet IP of target computer} ansible_ssh_user="${ ssh account name }" ansible_ssh_pass="${ ssh account password }" ansible_sudo_pass="${ sudo privilege escalation password }"

# Management console. An organization has a management console service
[admin]
${intranet IP of target computer} ansible_ssh_user="${ ssh account name }" ansible_ssh_pass="${ ssh account password }" ansible_sudo_pass="${ sudo privilege escalation password }"

# Resource node. An organization can configure multiple resource services
[data]
${intranet IP of target computer} ansible_ssh_user="${ ssh account name }" ansible_ssh_pass="${ ssh account password }" ansible_sudo_pass="${ sudo privilege escalation password }"

# Computing nodes. An organization can configure multiple computing services
[compute]
${intranet IP of target computer} ansible_ssh_user="${ ssh account name }" ansible_ssh_pass="${ ssh account password }" ansible_sudo_pass="${ sudo privilege escalation password }"

# Registration center. An organization needs to configure an odd number (1, 3, 5, etc.) of registration centers to facilitate raft to choose the leader
[consul]
${intranet IP of target computer} ansible_ssh_user="${ ssh account name }" ansible_ssh_pass="${ ssh account password }" ansible_sudo_pass="${ sudo privilege escalation password }"


[all:vars]
# The name of the cluster, which you can customize 
cluster_name = demo-cluster

# Deploy service switch
enable_deploy_ice_via = True # Whether to deploy, start, close, destroy ice_via. If yes, True; if no, False
enable_deploy_carrier = True # Whether to deploy, start, shut down, and destroy carrier. If yes, True; if no, False.
enable_deploy_admin = True # Whether to deploy, start, close, and destroy admin. If yes, True; if no, False.
enable_deploy_data = True # Whether to deploy, start, close, and destroy data. If yes, True; if no, False.
enable_deploy_compute = True # Whether to deploy, start, shut down, and destroy compute. If yes, True; if no, False.
enable_deploy_consul = True # Whether to deploy, start, close, and destroy consul. If yes, True; if no, False.

# The port of the consul service, set according to your deployment. 
# The number should be consistent with the number of ips in the consul group.
consul_server_port = [8200, 8201, 8202]
consul_serf_lan_port = [8300, 8301, 8302]
consul_serf_wan_port = [8400, 8401, 8402]
consul_http_port = [8500, 8501, 8502]
consul_dns_port = [8600, 8601, 8602]

# admin web service certificate related configuration information.
enable_tls = False # Whether to enable https. If yes, True, and you need to configure the certificate and the corresponding domain name, the cipher suite in the certificate, etc. If no, False, and just ignore the following configuration.
admin_server_name = datum-admin.demo.network
admin_ssl_protocols = "TLSv1 TLSv1.1 TLSv1.2"
admin_ssl_ciphers = ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4

# mysql related username and password of admin web
mysql_root_password = datum_root
amin_user = datum_admin
admin_password = admin_123456

# admin web service port number
admin_web_port = 9090

#admin web grpc service port number
admin_grpc_server_port = 9091

#The user needs to apply for a pinata service account,
#For the generation of pinata_token,please refer to https://docs.pinata.cloud/master#your -api-keys.
#Only 'pinning/pinfiletoipfs' and 'pinning/pinjsontoipfs' permissions are required
#Pinata's token, used to upload IPFs images
pinata_token = To be filled in
#It is strongly recommended that you apply for Pinata private gateway.please refer to https://docs.pinata.cloud/gateways
#You can also use the free public gateway of IPFs,Such as https://ipfs.io.Public gateways are generally suitable for testing, and there are many restrictions.
#Pinata's gateway for accessing IPFs images
pinata_gateway = To be filled in

# carrier extranet ip address
carrier_external_ip = ${extranet IP of carrier target computer}

# carrier service port number
carrier_pprof_port = 10032
carrier_rpc_port = 10033
carrier_grpc_gateway_port = 10034
carrier_p2p_udp_port = 10035
carrier_p2p_tcp_port = 10036

# glacier2/icegrid service port number
glacier2_port = 10031
icegrid_port = 20031

# ice via extranet ip address
ice_via_external_ip = ${extranet IP of ice_via target computer}
# ice_via service port number
ice_glacier2_port = 10031   # Glacier2 service port number
ice_grid_port = 10032       # IceGrid service port number
# Ice transport protocol: tcp(default)/ssl 
ice_protocol = tcp

# data port number, set according to your deployment. 
# The number should be consistent with the number of ips in the data group.
data_port = [8700, 8701, 8702]

# compute port number, set according to your deployment. 
# The number should be consistent with the number of ips in the compute group.
compute_port = [8700, 8701, 8702]
```

### Description of Variables in `inventory.ini`

#### 1. ssh related configuration (Not compulsory. If it's not configured, you need to manually enter the password when executing the playbook)

ansible_ssh_user: ssh account name of the target computer

ansible_ssh_pass: ssh account password of the target computer

ansible_sudo_pass: sudo privilege escalation password of ssh account on the target computer



#### 2. Cluster name

cluster_name: The name of the entire organization

#### 3. Deployment service switch options

enable_deploy_ice_via: Whether to deploy, start, close, and destroy ice_via. If yes, True; if no, False.

enable_deploy_carrier: Whether to deploy, start, shut down, and destroy carrier. If yes, True; if no, False.

enable_deploy_admin: Whether to deploy, start, shut down, and destroy admin. If yes, True; if no, False.

enable_deploy_data: Whether to deploy, start, shut down, and destroy data. If yes, True; if no, False.

enable_deploy_compute: Whether to deploy, start, shut down, and destroy compute. If yes, True; if no, False.

enable_deploy_consul: Whether to deploy, start, shut down, or destroy consul. If yes, True; if no, False.



#### 4. Configure the consul service port number (it is recommended to enable the intranet port policy only)

consul_server_port: The port of the consul service, in the form of an array [8200, 8201, 8202]. The port number is set according to your deployment, and the number should be consistent with the number of members of the consul group.

consul_serf_lan_port: The port to which Serf LAN gossip communication should be bound, in the form of an array [8300, 8301, 8302]. The port number is set according to your deployment, and the number should be consistent with the number of members in the consul group.

consul_serf_wan_port: The port to which Serf WAN gossip communication should be bound, in the form of an array [8400, 8401, 8402]. The port number is set according to your deployment, and the number should be consistent with the number of members in the consul group.

consul_http_port: HTTP API port, in the form of an array [8500, 8501, 8502]. The port number is set according to your deployment, and the number should be consistent with the number of members in the consul group.

consul_dns_port: DNS server port, in the form of an array [8600, 8601, 8602]. The port number is set according to your deployment, and the number should be consistent with the number of members in the consul group.

#### 5. admin web certificate related configuration

enable_tls: Whether to enable https. If yes, True, and you need to configure the certificate and the corresponding domain name, the cipher suite in the certificate, etc. If no, False, and just ignore the following configuration.

admin_server_name: ngnix configuration item server_name, the domain name of the computer where the admin service is deployed.

admin_ssl_protocols: ngnix configuration item ssl_protocols, the tls version supported by the certificate.

admin_ssl_ciphers: ngnix configuration item ssl_ciphers, the cipher algorithm supported by the certificate.

#### 6. admin mysql related configuration (to be modified by the user)

mysql_root_password: mysql root account password.

amin_user: The common user name used by mysql to create the business database.

admin_password: The password of the mysql ordinary user.

#### 7. Configure the admin web port number (it is recommended to enable the intranet port policy only)

admin_web_port: The port that provides web services for admin.

#### 8. The extranet ip address of the carrier service

carrier_external_ip: The extranet ip opened by the carrier's p2p service (the specific extranet ip is for external organizations to discover the organization).

#### 9. Port number configuration for carrier service

carrier_pprof_port: The port on which pprof, the debugging service of golang language by which carrier is written, listens, which is for development and debugging. It is recommended to enable the intranet port policy, which can be defined by the user as needed.

carrier_rpc_port: The port on which carrier's rpc server listens. The intranet and extranet port policies are enabled, which can be defined by the user as needed.

carrier_grpc_gateway_port: The port on which the restful server of carrier's rpc api listens. The intranet and extranet port policies are enabled, which can be defined by the user as needed.

carrier_p2p_udp_port: The port on which carrier's p2p udp server listens. The intranet and extranet port policies are enabled, which can be defined by the user as needed.

carrier_p2p_tcp_port: The port on which carrier's p2p tcp server listens. The intranet and extranet port policies are enabled, which can be defined by the user as needed.

#### 10. The extranet ip address of the ice_via service

ice_via_external_ip: The IP of the external network where the ice_via(Glacier2/IceGrid) service is located, in which the iceGrid service uses this IP for the service registration of the organization and the addressing of the gladier2 service; Other organizations need to communicate with the data and compute services in the organization through the IP address of this gladier2 service.

#### 11. ice_via service port number

ice_glacier2_port: The port on which the Glacier2 service listens
ice_grid_port: The port on which the IceGrid service listens

#### 12. ice_via Service Transport protocol

ice_protocol：ice_via service transport protocol, tcp by default, also supports ssl.

#### 13. fighter(data) service port number (it is recommended to enable the intranet port policy only)

data_port: The port on which the data service listens, in the form of an array [8700, 8701, 8702]. The port number is set according to your deployment, and the number should be consistent with the number of members in the data group.

#### 14. fighter(compute) service port number (it is recommended to enable the intranet port policy only)

compute_port: The port on which the data service listens, in the form of an array [8801, 8802, 8803], the port number is set according to your own deployment, and the number should be consistent with the number of members in the compute group.

## Example of Configuration File

`inventory.ini` Configuration example: [Configuration demo example](./doc/EN/ConfigurationDemoExample.md)



## Network of Each Service in the Organization

Network requirements: [Network](./doc/EN/NetworkConfiguration.md)

## Description of Each Operation of the Script (Master Computer Preparation, Initialization, Deployment, Startup, Shutdown, Destruction, etc.)

> **Please be sure to check the above configuration before performing the following operations**

### 1. Preparations before the first deployment (the following two commands are preliminary preparations, which are executed once during the first deployment only)

```shell
# The preparation on the master computer mainly involves downloading and deploying related binary files and executing them in the root directory of the script `datum-network-deploy` (you need to enter the sudo password):

ansible-playbook --ask-sudo-pass local_prepare.yml

# Initialize each node of the cluster, mainly to check the environment information of the target computer (operating system version, python and python3 installation), and execute it in the root directory of the script `datum-network-deploy`:

ansible-playbook -i inventory.ini bootstrap.yml
```

### 2. Install related services

```shell
# Install binary files and configuration files (install corresponding services) for each target computer on the master computer, and execute it in the root directory of the script `datum-network-deploy`:

ansible-playbook -i inventory.ini deploy.yml
```

### 3. Start related services

```shell
# Start related services on each target host on the master computer (running in the daemon state in the background), and execute it in the root directory of the script `datum-network-deploy`:

ansible-playbook -i inventory.ini start.yml
```

### 4. Stop the service

```shell
# Stop related services on each target host on the master computer (gracefully stop the process with a signal), and execute it in the root directory of the script `datum-network-deploy`:

ansible-playbook -i inventory.ini stop.yml 
```

### 5. Destroy the service

> **Before performing this operation, you need to stop the service first. This operation is dangerous and will clear all the data including the admin business database in the mysql database. If the current Datum-network Node (organization) has operated in the admin console before [registered identity information] ], then you need to execute [Logout Identity] on the admin console before cleanup.**

```shell
# First stop the related services on each target host on the master computer (gracefully stop the process with a signal), and execute it in the root directory of the script `datum-network-deploy`:

ansible-playbook -i inventory.ini stop.yml 

# Then clear the data of related services and installed binary file configuration on each target computer on the master computer (clear all data), and execute it in the root directory of the script `datum-network-deploy`:

ansible-playbook -i inventory.ini cleanup.yml
```



## Instructions for Use

After the deployment of Datum-network Node is completed, please refer to: [Instructions for Use](./doc/EN/InstructionsForUseOfDatumNetwork.md)

## Task event type code comparison table

[Click to view](./doc/EN/eventCodeDetails.md)

## FAQ

For common problems and solutions in deployment, refer to: [FAQ](./doc/EN/FAQ_EN.md)

[OrganizationNetworkTopologyOfDatumNetworkNode]: ./img/OrganizationNetworkTopologyOfDatumNetworkNode.jpg
[InternalServiceNetworkTopologyOfASingleDatumNetworkNode]: ./img/InternalServiceNetworkTopologyOfASingleDatumNetworkNode.jpg
[TopologyBetweenDatumPlatformAndDatumNetworkNode]: ./img/TopologyBetweenDatumPlatformAndDatumNetworkNode.jpg
