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

Would you like to see the actual JSON of the policy or an example of how it's attached to a node role?

Yes:
📄 AmazonEKS_CNI_Policy – IAM Policy JSON
This is the current AWS-managed version (as of the latest knowledge update):

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:AssignPrivateIpAddresses",
        "ec2:AttachNetworkInterface",
        "ec2:CreateNetworkInterface",
        "ec2:DeleteNetworkInterface",
        "ec2:DescribeInstances",
        "ec2:DescribeTags",
        "ec2:DescribeNetworkInterfaces",
        "ec2:DescribeInstanceTypes",
        "ec2:DetachNetworkInterface",
        "ec2:ModifyNetworkInterfaceAttribute",
        "ec2:UnassignPrivateIpAddresses",
        "ec2:DescribeSubnets",
        "ec2:DescribeSecurityGroups",
        "ec2:CreateTags"
      ],
      "Resource": "*"
    }
  ]
}
🔐 Note: The wide "Resource": "*" is necessary because the plugin needs to dynamically manage and discover ENIs across nodes in your VPC.
🔧 How to Attach AmazonEKS_CNI_Policy to a Node Role
When you launch worker nodes (or use managed node groups), the EC2 instances running your pods need an IAM role with this policy attached. You can do this through the AWS Console, CLI, or IaC (like Terraform).

Example using AWS CLI:

bash
Copy
Edit
aws iam attach-role-policy \
  --role-name yourNodeInstanceRoleName \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
This assumes you already have a role (yourNodeInstanceRoleName) used by your EKS worker nodes.


