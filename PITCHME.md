## Anthony's EKS Learning Journey

---

#### What is EKS?
@ul[list-spaced-bullets text-09]
- [AWS Elastic Kubernetes Service](https://aws.amazon.com/eks/)
- A managed Kubernetes service hosted by AWS
@ulend

---

#### Why would I want to use EKS?
@ul[list-spaced-bullets text-09]
- Let AWS manage the k8s master plane, feature updates, and security patches
- ~$70 per month per master plane vs FTE staff managing underlying k8s components
- Integrates with our existing AWS infrastructure
- Accessibility to other AWS resources(S3, Lambda, Cloudfront, etc)
@ulend

---

#### How do I deploy EKS?
@ul[list-spaced-bullets text-09]
- An AWS Account and access/secret key credentials
  - There's alot of resources to be created(IAM, EC2, VPC, etc.), so admin privileges is preferable
- [Terraform](https://www.terraform.io/)
- AWS CLI installed on your workstation
@ulend

---

#### How do I start writing my terraform configs?
@ul[list-spaced-bullets text-07]
- IMO, a good habit to develop when writing terraform configs is working backwards
- You will have to do research on your hosted or local environment to determine which terraform resources to use in your configuration
- These were the questions I used in determining my EKS terraform deployment
  - What AWS components does EKS create/manage?
  - How do we want to configure the EC2 environment in which EKS utilizes?
  - How do you scale EC2 instances to address compute and load requirements?
  - How do you access storage on EKS?
  - How do you access restricted resources from campus?
  - How do you use EKS to deploy an application?
  - How do you access kubectl?
@ulend

---

#### What AWS components does EKS create/manage?
@ul[list-spaced-bullets text-07]
- EC2 instances(EKS node groups)
- K8s master plane(SaaS)
- Load Balancers - AWS ALB/ELB/NLB(through kubectl configured ingress controller)
@ulend

---

#### How do we want to configure the EC2 environment in which EKS utilizes?
@ul[list-spaced-bullets text-07]
- Secure VPC network(DO NOT USE DEFAULT!)
- Subnets in different availability zones for high availability
- Traffic ingress must traverse load balancer
- Egress access to download container images from external registries(Dockerhub)
- Remote SSH to managed EKS EC2 instances is configurable
@ulend

---

@ul[list-spaced-bullets text-07]
#### How do you scale EC2 instances to address compute and load requirements?
- AWS EKS Node Group sets desired, lower and upper bound limits for scaling
- EKS does not automatically scale your node group. AWS recommends configuring the [Kubernetes Cluster Autoscaler ](https://aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-autoscaler-setup/) to auto scale the cluster within the bound limits of the node group
- Metrics to scale are defined in the cluster autoscaler config
@ulend

---

#### How do you access restricted resources from campus?
@ul[list-spaced-bullets text-07]
- Create a dedicated NAT gateway(egress traffic routes through a fixed elastic IP)
- Create VPC route tables that matches campus networks to use the NAT gateway
- Associate route table to EKS utilized subnets
- Modify campus resources to allow for NAT IP traffic
- NAT gateways are really meant to be used to meet ACL criteria instead of transferring large sets of data
@ulend

---

#### How do you access storage on EKS?
@ul[list-spaced-bullets text-07]
- S3 / AWS EFS / AWS FSx(Luster) for persistent storage
- AWS EBS for ephemeral, high IOPS and throughput storage
@ulend

---

#### How do you use EKS to deploy an application?
@ul[list-spaced-bullets text-07]
- K8s has an API endpoint
- REST API if you want to interact directly
- kubectl is the recommended tool to interface with the API
- Requires EKS token to access API
- EKS Token generated from AWSCLI
- EKS Token has full IAM role privileges
- Alternative to token authentication: [aws-iam-authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)
@ulend

---

#### How do you access the cluster with kubectl?
@ul[list-spaced-bullets text-07]
- You can configure EKS to have a private or public endpoint. Both configurations still require access through an EKS token
- Set EKS cluster to be accessible through a private endpoint
- Create a bastion EC2 host on the same VPC network as EKS EC2 instances
- bastion EC2 instance only has SSH access from jump.library.ucla.edu
- Admins/developers/bots would access the bastion EC2 instance to execute kubectl commands to designated clusters
@ulend


---

Obtaining an EKS token
```
export TOKEN=$(aws eks get-token --cluster-name eks_cluster --profile cachemeoutside | jq -r '.status.token')
```

---

Accessing the REST API
```
curl -k -X GET https://7A1976C36017B9E61D16199121478B4B.gr7.us-west-2.eks.amazonaws.com/api --header "Authorization: Bearer $TOKEN"
```

---

#### Summary of aws terraform resources
```
- Region
- Availability zones
- VPC
- Subnets
- Route tables
- Associating route tables
- Security groups
- NAT Gateway
- Internet Gateway
  - Ingress
  - Egress
- AWS AMI
- AWS ssh key pair
- EC2
- IAM Roles
- IAM Policies
- Associating IAM roles
- EKS Cluster
- EKS Node group
```

---

#### Some other things I learned
@ul[list-spaced-bullets text-07]
- Limited available selections of which load balancers we can utilize(AWS ALB/NLB/ELB)
- Terraform is amazing for spinning up the EKS and other AWS resources, but running K8s itself heavily relies on kubectl
- Not sure what the K8s update schedule is. Latest stable is from K8s is v1.17, EKS latest stable is 1.14
- When EKS platform versions become available for a minor version, EKS automatically upgrades all existing clusters to the latest Amazon EKS platform version for their corresponding Kubernetes minor version
- You can use kubectl to get a shell onto a runner container
@ulend

---
#### Takeaways
@ul[list-spaced-bullets text-07]
- EKS is great and works as advertised
- EKS offers SaaS capabilities, but flexibility to debug and manage individual resources as needed
- K8s deployments and configurations are all done through yaml files and is portable enough to run on other k8s clusters(even minikube)
- The only lock-in appears to be the available selections of our ingress controller(load balancer)
- Easy to spin up, looking forward to moving the services team's Fargate environment to EKS
- k8s is a beast by itself, but powerful and flexible
- I <3 terraform
@ulend

