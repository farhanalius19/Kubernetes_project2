# This is the README file for this project.

# Project Statement:

Create a DevOps infrastructure for an e-commerce application to run on
high-availability mode.
Background of the problem statement:
A popular payment application, EasyPay where users add money to their wallet
accounts, faces an issue in its payment success rate. The timeout that occurs
with the connectivity of the database has been the reason for the issue.
While troubleshooting, it is found that the database server has several
downtime instances at irregular intervals. This situation compels the company
to create their own infrastructure that runs in high-availability mode.
Given that online shopping experiences continue to evolve as per customer
expectations, the developers are driven to make their app more reliable, fast,
and secure for improving the performance of the current system.
Implementation requirements:
1. Create the cluster (EC2 instances with load balancer and elastic IP in case
of AWS)
2. Automate the provisioning of an EC2 instance using Ansible or Chef
Puppet
3. Install Docker and Kubernetes on the cluster
4. Implement the network policies at the database pod to allow ingress
traffic from the front-end application pod
5. Create a new user with permissions to create, list, get, update, and delete
pods
6. Configure application on the pod
7. Take snapshot of ETCD database
8. Set criteria such that if the memory of CPU goes beyond 50%,
environments automatically get scaled up and configured
The following tools must be used:

a. EC2
b. Kubernetes
c. Docker
d. Ansible or Chef or Puppet

The following things to be kept in check:
1. You need to document the steps and write the algorithms in them.
2. The submission of your GitHub repository link is mandatory. In order to
track your tasks, you need to share the link of the repository.
3. Document the step-by-step process starting from creating test cases, then
executing them, and recording the results.
4. You need to submit the final specification document, which includes:
● Project and tester details
● Concepts used in the project
● Links to the GitHub repository to verify the project completion
● Your conclusion on enhancing the application and defining the USPs
(Unique Selling Points)

---

LAUNCHING EC2 INSTANCES IN CLUSTER:

Make sure to use t2.medium or any other instance with the following specifications

No. of vCPU: 2 or more
Free RAM: At least 540 MB
Space: 15 GB

Use Ubuntu 20.04 LTS or 22.04 LTS

Create a security group which allows SSH and HTTP/HTTPS traffic to the EC2 instances.

After instances are created, connect to the instance and check for the httpd service. By default, Apache server comes with Ubuntu 20.04 and 22.04, but to make sure the service is enabled and running, check the service manually with following command:

# sudo systemctl status apache2

If the command returns "service not available", then service is not installed on your EC2. Install it with 

# sudo apt update
# sudo apt install apache2 -y

Or 

# sudo apt-get install apache2 -y

If the service is installed and not enabled, start it with:

# sudo systemctl start apache2

Note: Check the status of httpd on each noce in your cluster

To make a cluster you need to manually add each node/EC2 to your cluster. 
For this purpose install microk8s on each node and start the service.
After installing microk8s edit the /etc/hosts file on each node. Add the IP Address and Hostname of each node.
Example:

IP_ADDRESS_OF_NODE HOSTNAME_OF_NODE

Note: Before adding node to your cluster, make sure it is publicaly reachable. If not then check the security group and allow traffic from with in the VPC.

On your Master node, run the following command:

# sudo microk8s add node

This command will output another command. Copy and run that command on each worker node. Must add '--worker' flag to make the node understand that it is a worker and not the master.

Wait for a while and then check the status of added nodes on the master node with the following command:

# sudo microk8s get nodes

This command will diplay all the nodes currently in the cluster. All the nodes should be available and connected to the internet for the master node to communicate with them.

Note: It is recommended to deploy EC2 instances in Public Subnet. If you want to use Private Subnet, make sure to attach a NAT Gateway for traffic to reach Instances.

To attach a Load Balancer with the EC2 instances, first create a Target Group for the load balancer. Use an Application Load Balancer which runs HTTP health checks on port 80 of the EC2s. Attach a Global Accelerator on the Load Balancer which will give it a static/elastic IP.

Global Accelerator is a new service offered by AWS which attaches a Static/Elastic IP to your resource. In addition to the Elastic IP, Global Accelerator increases the performance and availability of applications.

---

PROVISIONING EC2 INSTANCE WITH ANSIBLE:

To achieve this task, Ansible, Python, pip, AWS CLI and boto should be installed and configured properly. First update the package manager and then install the packages with the following commands:

# sudo apt update 		(Update the package manager)
# sudo apt install ansible -y	(Install Ansible)
# sudo apt install awscli -y	(Install the AWS CLI)
# sudo apt install python-is-python3 -y 	(Install Python)
# sudo apt install python3-pip -y	(Install Python package manager 'pip')

# sudo pip install boto		(Install boto with pip)

After installing all the packages, first configure AWS. Run the following command:

# aws configure

Then enter AWS Access and Secret Key. Additionally, it is preffered to enter a region as well. Skip the 4th option by leaving it empty and press enter. Afterwars create the ansible playbook which contains all the configuration files for launching an EC2. An examplary file is present in the repository.

Save the file and quit VI Editor.

Run the playbook as follows:

# ansible-playbook <FILE_NAME>.yaml

If the configuration file is correct, an EC2 will be launched in the specified region.

---

INSTALLING DOCKER AND KUBERNETES:

We have previously used Kubernetes to make a cluster. Install Docker with the following command:

# sudo apt install docker.io

Then,

# sudo snap install docker

As Kubernetes is already installed, it is a good practice to alias a Microk8s command for our ease. This ease is carried out throughout the project from here. If you don't want to alias the command, it will not effect the project as long as you enter the whole command which is 'microk8s kubectl'.

To shorten/alias the command run the following command:

# sudo snap alias microk8s.kubectl kubectl

Here we have aliased the command to 'kubectl' which will be used in the further steps.

Note: If you are going to skip this step, add 'microk8s' to all the commands where 'kubectl' is used

---

IMPLEMENTING INGRESS TRAFFIC FROM NETWORK POLICY:

An emaplary Network Policy file is present in the repository. Make sure the labels are correct both for target and source pods.

We can also ingress traffic from IP CIDRs. If you're using IP CIDRs then there is no need to add labels or namespaces for source pods. Kubernetes will only use IP CIDRs for ingressing traffic.

---

CREATING A NEW USER WITH PERMISSIONS:


To add an actual human with access to the Kubernetes cluster, first make a new user on the system with any preferred authentication method. To add the user to your cluster use the manifest files present in the repository as an example. The examplary manifests are for creating users on the cluster level to "get", "create", "update", "list" and "delete" pods. There are two files, one for defining the role and the second for binding the role to the user. Be sure to apply both manifests, otherwise the user will not be created.

Apply the manifests with the following commands:

# sudo kubectl apply -f <FILE_NAME_FOR_ROLE_DEFINITION>.yaml 

and, 

# sudo kubectl apply -f <FILE_NAME_FOR_ROLE_BINDING>.yaml

---

CONFIGURE APPLICATION ON THE POD:

To configure an application on the pod, we are going to use an examplary application which uses Wordpress as the front-end and MySQL as the database of the application. Wordpress first checks if MySQL service is present or not. Wordpress container will only be launched once MySQL service is present. To launch MySQL we need to create a Secret which will contain the MYSQL ROOT PASSWORD and successfully launch the container. We also need Persistent Volume to store data of the application. The Persistent Volume will be static, of 10GBs. The Persistent Volume Claim will be of 3GBs. At last all the configurations will be entered in the Deployment manifest and pod will be launched. ConfigMaps are an optional resource used in this project. Their absence will not effect the deployment of the pod in any way. To use ConfigMaps, uncomment the lines in the deployment manifest.

All the manifests are present in the repository as an example. 

Note: Take extra care while labeling your pods, persistent volumes, persistent volume claims. They will be used by other resources for communication. 

To apply any manifest, run the following command:

# sudo kubectl apply -f <FILE_NAME>.yaml

This command will launch the resource in the default namespace. If you want to launch the resource in a specific namespace, use the above command with '-n' flag as follows:

# sudo kubectl -n <NAMESPACE_NAME> apply -f <FILE_NAME>.yaml

Persistent Volume does not require a namespace, it is launched above the namespace level. However, Persistent Volume Claim requires a namespace. If no namespace is mentioned while applying PVC manifest, it is deployed in the 'default' namespace.

---

TAKE SNAPSHOT OF ETCD DATABASE:

Note: For ETCD to take proper action, it is recommended to install it on the system before installing microk8s or any Kubernetes deployment.

To take a snapshot of ETCD Database, first you need to install 'etcdctl' utility on your master node.

First install the following utilities, if not already available:

1): wget
2): curl

Install the utilities with

# sudo apt install wget curl


Install the etcdctl with the following commands:

# export RELEASE=$(curl -s https://api.github.com/repos/etcd-io/etcd/releases/latest|grep tag_name | cut -d '"' -f 4)

# wget https://github.com/etcd-io/etcd/releases/download/${RELEASE}/etcd-${RELEASE}-linux-amd64.tar.gz

Extract the files with:

# tar xvf etcd-${RELEASE}-linux-amd64.tar.gz

Change to the unzipped directory

# cd etcd-${RELEASE}-linux-amd64

Move the etcd and etcdctl binary files to local binary directory with the following command:

# sudo mv etcd etcdctl etcdutl /usr/local/bin

Now create a ETCD configuration file and data directory. Run the following commands:

# sudo mkdir -p /var/lib/etcd
# sudo mkdir /etc/etcd

Create a ETCD system user with the following command:

# sudo groupadd --system etcd
# sudo useradd -s /sbin/nologin --system -g etcd etcd

Now we will set the directory ownership of etcd directory to 'etcd' user

# sudo chown -R etcd:etcd /var/lib/etcd/

Creating a systemd service file for etcd

# sudo vi /etc/systemd/system/etcd.service

After entering the vi editor, paste the following lines to the file

[Unit]
Description=etcd database
Documentation=https://github.com/etcd-io/etcd
After=network.target

[Service]
User=etcd
Type=notify
Environment=ETCD_DATA_DIR=/var/lib/etcd
Environment=ETCD_NAME=%m
ExecStart=/usr/local/bin/etcd
Restart=always
RestartSec=10s
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target

Save the file and exit the vi editor

After exiting the editor, start the etcd service with the following command:

# sudo systemctl daemon-reload
# sudo systemctl start etcd.service

Check the status of the service with

# sudo systemctl status etcd.service

After the service is configured and running, you can take the snapshot with the following commands:

ETCDCTL_API=3 etcdctl \
--endpoints=https://[127.0.0.1]:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/snapshot-pre-boot.db

This command will save the snapshot of the ETCD database. Use the next command to print the snapshot in a tabular form to confirm that snapshot has been taken.

ETCDCTL_API=3 etcdctl \
--endpoints=https://[127.0.0.1]:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot status /opt/snapshot-pre-boot.db -w table

--- 

SETTING THE CRITERIA FOR THE ENVIRONMENT:

To set criteria for the environments to scale up and down depending upon the utilization of CPU, we will use Horizontal Pod Autoscaler (HPA). 

To use HPA, first enable the metrics server of microk8s with the following command:

# sudo microk8s enable metrics-server

After enabling the service, run the following command to attach an HPA to our deployment

# sudo kubectl autoscale deployment <NAME_OF_DEPLOYMENT> --cpu-percent=50 --min=1 --max=10

Name of deployment should be the same as mentioned in the 'metadata' module of the deployment manifest. Minimum and Maximum are administrator's choice. 

Note: If your deployment is not in default namesapce, use '-n' flag with the above command as follows:

# sudo kubectl -n <NAMESPACE_NAME> autoscale deployment <NAME_OF_DEPLOYMENT> --cpu-percent=50 --min=1 --max=10

Check the HPA with the following command:

# sudo kubectl get hpa

Or

# sudo kubectl -n <NAMESPACE_NAME> get hpa 

Use this command if HPA and deployment are in another namespace other than 'default'
