## __KUBERNETES SETUP WITH KUBERNETES OPERATIONS (KOPS)__

__Kops__ - __"Kubernetes Operations"__, is a command-line tool used for deploying, managing, and operating Kubernetes clusters on various cloud platforms. Kubernetes is an open-source container orchestration platform that helps manage the deployment, scaling, and operation of containerized applications.

Kops specifically focuses on simplifying the process of setting up and managing Kubernetes clusters on cloud infrastructure providers like Amazon Web Services (AWS), Google Cloud Platform (GCP), and others. It allows users to define and customize their Kubernetes clusters using configuration files, which makes it easier to maintain consistent and reproducible cluster configurations.

__Key features of Kops include:__

__Cluster Provisioning:__ Kops helps automate the provisioning of infrastructure resources required for a Kubernetes cluster, such as virtual machines, networking, and storage.

__Cluster Upgrades:__ Kops simplifies the process of upgrading Kubernetes clusters to newer versions, ensuring that the update process is seamless and minimizes downtime.

__Scaling:__ Kops allows you to scale your cluster by adding or removing nodes, adjusting the resources allocated to nodes, and managing node groups.

__High Availability:__ Kops supports configuring high availability features of Kubernetes, such as distributing master nodes across multiple availability zones, which enhances the resiliency of the cluster.

__Customization:__ Users can define various cluster configurations, including node instance types, networking settings, add-ons, and other parameters, according to their requirements.

__Validation and Verification:__ Kops includes tools for verifying the correctness of your cluster configuration and diagnosing potential issues before deploying the cluster.

In this instance, I will be configuring a Kubernetes cluster by employing the Kops tool.

__Prerequisites:__

- A domain name for Kubernetes DNS record - __dybran.com__.
- AWS account

__Task:__

- Log into AWS account:
  - Create an ubuntu Instance.
  - Create __S3 bucket__.
  - Create an __IAM user__ for awscli.
  - Create __Route 53__ hosted zone.

- SSH into the instance
- Setup the following:
   - kops
   - kubectl
   - awscli
   - ssh keys

Purchase a domain name from a domain name provider. eg godaddy.com and namesilo.com etc.

Create an ubuntu EC2 instance for Kops

![](./images/kops-inst.PNG)

Create S3 bucket

![](./images/state.PNG)

Kops will establish a cluster and utilize awscli for interaction with AWS services. To enable access to these services via awscli commands, valid credentials are essential.

Create an IAM role that will be endowed with administrator-level privileges for the kops instance. This IAM role is essential as it will interact with a wide range of services, including S3 buckets and Route53. Once this role is in place, the next step involves setting up the kops instance to utilize the associated access keys.

![](./images/21.PNG)
![](./images/22.PNG)
![](./images/23.PNG)
![](./images/24.PNG)
![](./images/25.PNG)
![](./images/26.PNG)

Create Hosted zone in Route53

![](./images/hz1.PNG)

This creates the __ns servers url__ which we will add to the domain servers register in the DNS provider - __www.godaddy.com__

Log into your DNS provider account and update the name servers with the __ns server url__ that was created earlier

![](./images/hz2.PNG)

SSH into the EC2 instance and complete the setup.

Generate __ssh keys__

![](./images/key1.PNG)

Install __awscli__

`$ sudo apt update && sudo apt install awscli -y`

![](./images/key2.PNG)

Configure it to use the access keys of the __IAM user__ that was created.

`$ aws configure`

![](./images/key3.PNG)

__Setup kubectl and kops__

To setup __kubectl__ we will refer to the [kubernetes documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux).

Install __kubectl__

`$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"`

Make kubectl executable

`$ sudo chmod +x kubectl`

To make __kubectl__ globally accessible

`$ sudo mv kubectl /usr/local/bin/`

Verify

`$ kubectl version --client`

![](./images/dsaq.PNG)

To setup __kops__ we will refer to the [kops documentation](https://github.com/kubernetes/kops/releases) for various versions og kops.

I will be installing version __1.26.4__

![](./images/amd1.PNG)
![](./images/amd2.PNG)

Copy the above link and download the binary

`$ wget https://github.com/kubernetes/kops/releases/download/v1.26.4/kops-linux-amd64`


Make kops executable

`$ chmod +x kops-linux-amd64`

To make __kops__ globally accessible

`sudo mv kops-linux-amd /usr/local/bin/kops`

`$ kops version`

![](./images/dsaw.PNG)


Verify domain __kubekops.dybran.com__

`$ nslookup -type=ns kubekops.dybran.com`

![](./images/nsv.PNG)

Create configurations for the cluster and store in the S3 bucket using the command:

`$ kops create cluster --name=kubekops.dybran.com \
--state=s3://project-kops-state --zones=us-east-1a,us-east-1b \
--node-count=2 --node-size=t3.small --master-size=t3.medium --dns-zone=kubekops.dybran.com \
--node-volume-size=8 --master-volume-size=8`

![](./images/lfc.PNG)


Make sure to specify the `--node-volume-size` and `--master-volume-size` if not it will create huge volume for the `etcd`.

Configure and create the cluster by running the command

`$  kops update cluster --state=s3://project-kops-state --name kubekops.dybran.com --yes --admin`

![](./images/vcs.PNG)

![](./images/cp.PNG)


Wait for __15 minutes__ for the cluster to create.

On every execution of the __'kops'__ command, inclusion of the associated bucket name is mandatory. In the absence of this S3 bucker name the command will not work.

Validate the cluster

`$ kops validate cluster --state=s3://project-kops-state`

![](./images/qaq.PNG)


Upon the installation of __kops__, a configuration file named __.kube/config__ was generated within the home directory for the purpose of executing __kubectl__ commands. 

`$ cat .kube/config`

![](./images/kube.PNG)

To see the nodes we run the command

`$ kubectl get nodes`

![](./images/gets.PNG)


To delete the cluster

`$ kops delete cluster --name=kubekops.dybran.com --state=s3://project-kops-state --yes`

![](./images/dele.PNG)
