# EKS-Terraform
The AmazonEKS_CNI_Policy is an AWS managed IAM policy designed specifically for the Amazon Elastic Kubernetes Service (EKS) Container Network Interface (CNI) plugin, also known as the Amazon VPC CNI plugin for Kubernetes.

✅ Purpose of AmazonEKS_CNI_Policy:
This policy grants necessary permissions for the CNI plugin to:

Attach and detach ENIs (Elastic Network Interfaces) to EKS worker nodes.

Assign IP addresses from your VPC to pods.

Tag network resources, like ENIs.

Describe EC2 resources, such as instances and subnets.

Without these permissions, the CNI plugin can’t manage networking for your pods, which breaks key EKS functionality like assigning pod IPs.

📌 Why it's needed:
In EKS, pods get IP addresses from your VPC CIDR, and this is handled by the CNI plugin. For this plugin to do its job, it must:

Interact with EC2 APIs to manage network interfaces.

Allocate IPs from subnets.

Clean up network resources when pods are deleted.

So, AmazonEKS_CNI_Policy ensures the plugin can do all this securely and effectively.
