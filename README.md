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

Do you want a full walkthrough using Terraform or AWS Console?

AWS Console:
=========== Here's a quick step-by-step to attach the AmazonEKS_CNI_Policy to your EKS node IAM role using the AWS Management Console:

🧭 Steps to Attach AmazonEKS_CNI_Policy in Console:
✅ Step 1: Find your EKS Node IAM Role
Go to the IAM Console.

In the Roles section, search for the role associated with your EKS nodes.

It’s often named like: eksctl-your-cluster-NodeInstanceRole-XYZ or similar.

If you're unsure, go to EKS Console > Your Cluster > Compute > Node Groups and check the IAM Role for the node group.

✅ Step 2: Attach the Policy
Click on the role name for your node group.

In the Permissions tab, click the "Add permissions" button.

Choose "Attach policies".

Search for AmazonEKS_CNI_Policy.

Check the box next to it.

Click "Next" and then "Add permissions".

That’s it! 🎉 Your node role now has permission to let the VPC CNI plugin do its thing.

Want to verify it's working or check logs to confirm it's active?

Let’s make sure everything’s wired up right. You can verify the Amazon VPC CNI plugin is working and using the policy correctly by checking logs and pod status in your EKS cluster.

🔍 How to Verify Amazon VPC CNI Plugin is Working
✅ 1. Check the aws-node DaemonSet Pods
The VPC CNI plugin runs as a DaemonSet called aws-node in the kube-system namespace.

Run this in your terminal:

bash
Copy
Edit
kubectl get pods -n kube-system -l k8s-app=aws-node
You should see one aws-node pod per node, and they should all be in the Running or Completed state.

✅ 2. Check Logs for Errors
Pick one pod (replace with actual pod name):

bash
Copy
Edit
kubectl logs -n kube-system <aws-node-pod-name>
What you don’t want to see:

Permission errors like UnauthorizedOperation or AccessDenied

Failed calls to EC2 APIs

What you do want to see:

Logs showing ENI/IP attachment

Messages like: "Successfully assigned IPs" or "Added ENI" etc.

✅ 3. (Optional) Confirm Pod IPs Match VPC
You can also run:

bash
Copy
Edit
kubectl get pods -o wide
Check the IP column — those IPs should belong to your VPC CIDR range, not some internal or made-up range. That confirms the CNI is assigning real VPC IPs to pods.

using Terraform:
===============

 let’s do it Terraform-style. Here's how you can attach the AmazonEKS_CNI_Policy to your EKS worker node IAM role using Terraform.

🛠️ Step-by-Step: Attach AmazonEKS_CNI_Policy in Terraform
✅ 1. Find or Define Your Node IAM Role
If you're using eksctl, it might already be created. But in Terraform, you likely have something like this:

hcl
Copy
Edit
resource "aws_iam_role" "eks_node_role" {
  name = "eks-node-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect = "Allow",
        Principal = {
          Service = "ec2.amazonaws.com"
        },
        Action = "sts:AssumeRole"
      }
    ]
  })
}
✅ 2. Attach the AmazonEKS_CNI_Policy
Use the built-in managed policy ARN:

hcl
Copy
Edit
resource "aws_iam_role_policy_attachment" "cni_policy_attachment" {
  role       = aws_iam_role.eks_node_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
}
You can place this right after your node role resource.

✅ 3. Apply Your Terraform
Run:

bash
Copy
Edit
terraform apply
This will attach the policy to the IAM role. If the nodes are already running, the change takes effect right away. No reboot needed.



