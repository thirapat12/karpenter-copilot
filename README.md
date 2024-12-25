# Project EKS: Installing Karpenter on Amazon Elastic Kubernetes Service (EKS)

This guide provides detailed instructions on how to install and configure Karpenter on an Amazon Elastic Kubernetes Service (EKS) cluster. Karpenter is an open-source node provisioning project built for Kubernetes.

## Prerequisites

1. **AWS CLI:** Ensure that the AWS CLI is installed and configured with the necessary permissions. You can download it [here](https://aws.amazon.com/cli/).
2. **eksctl:** Ensure that `eksctl` is installed. You can download it from the [eksctl GitHub releases page](https://github.com/weaveworks/eksctl/releases).
3. **kubectl:** Ensure that `kubectl` is installed. You can download it [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).
4. **Helm:** Ensure that Helm is installed. You can install Helm using the following command:

   ```sh
   curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
   ```

# Step 1: Create an EKS Cluster (if not already created)
If you haven't created an EKS cluster yet, you can create one using eksctl:

    eksctl create cluster --name my-cluster --region us-west-2 --nodegroup-name initial-nodes --node-type t3.medium --nodes 3 --nodes-min 1 --nodes-max 4 --managed

# Breakdown of the Command
- <mark>eksctl create cluster</mark> : This is the base command to create a new EKS cluster.
- <mark>--name my-cluster</mark> : This option sets the name of the EKS cluster to my-cluster. You can replace my-cluster with any name you prefer for your cluster.
- <mark>--region us-west-2</mark> : This option specifies the AWS region where the EKS cluster will be created. In this case, the cluster will be created in the us-west-2 region (Oregon). You can change this to any other AWS region based on your requirements.
- <mark>--nodegroup-name initial-nodes</mark> : This option names the initial node group as initial-nodes. A node group is a group of EC2 instances that will be used as worker nodes in your EKS cluster. You can name this node group anything you like.
- <mark>--node-type t3.medium</mark> : This option specifies the type of EC2 instances to be used for the worker nodes. In this case, t3.medium instances are used. You can choose other instance types based on your workload requirements and budget.
- <mark>--nodes 3</mark> : This option sets the desired number of nodes in the node group to 3. This is the initial number of worker nodes that will be created in the cluster.
- <mark>--nodes-min 1</mark> : This option sets the minimum number of nodes in the node group to 1. This means that the node group will always have at least 1 node running.
- <mark>--nodes-max 4</mark> : This option sets the maximum number of nodes in the node group to 4. This means that the node group can scale up to 4 nodes if needed.
- <mark>--managed</mark> : This option indicates that the node group should be managed by EKS. Managed node groups provide auto-scaling and simplified lifecycle management for worker nodes.

## Summary

This command will create an EKS cluster named my-cluster in the us-west-2 region with a managed node group named initial-nodes. The node group will use t3.medium EC2 instances and will initially have 3 nodes, with a minimum of 1 node and a maximum of 4 nodes.

Update your kubeconfig to use the new cluster:

    aws eks --region us-west-2 update-kubeconfig --name my-cluster

# Step 2: Install Helm and Karpenter
1. Add Karpenter Helm Repository:
    ```sh
    helm repo add karpenter https://charts.karpenter.sh
    helm repo update
    ```

2. Create IAM Role for Karpenter:

   Create an IAM policy for Karpenter:

    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "ec2:CreateLaunchTemplate",
            "ec2:CreateFleet",
            "ec2:RunInstances",
            "ec2:TerminateInstances",
            "ec2:DescribeInstances",
            "ec2:DescribeLaunchTemplates",
            "ec2:DescribeSubnets",
            "ec2:DescribeSecurityGroups",
            "ec2:DescribeInstanceTypes",
            "iam:PassRole"
          ],
          "Resource": "*"
        }
      ]
    }
    ```

# Breakdown of the Policy
  
  ## Version
  ```sh
  "Version": "2012-10-17"
  ```
  This specifies the version of the policy language. The date "2012-10-17" indicates the version of the IAM policy language being used.

  ## Statement
  
  The Statement element contains one or more individual statements. Each statement grants permissions for specific actions on specific resources.

  ## Effect
  ```sh
  "Effect": "Allow"
  ```
  The Effect element specifies whether the statement allows or denies access. In this case, it is set to "Allow", meaning the actions listed in the Action element are permitted.

  ## Action
    JSON
    "Action": [
      "ec2:CreateLaunchTemplate",
      "ec2:CreateFleet",
      "ec2:RunInstances",
      "ec2:TerminateInstances",
      "ec2:DescribeInstances",
      "ec2:DescribeLaunchTemplates",
      "ec2:DescribeSubnets",
      "ec2:DescribeSecurityGroups",
      "ec2:DescribeInstanceTypes",
      "iam:PassRole"
    ]

  The Action element specifies the list of actions that are allowed. Each action corresponds to a specific API operation in AWS services. Here are the actions included in this policy:

  - <mark>ec2:CreateLaunchTemplate</mark> : Allows creating EC2 launch templates.
  - <mark>ec2:CreateFleet</mark> : Allows creating EC2 fleets.
  - <mark>ec2:RunInstances</mark> : Allows launching new EC2 instances.
  - <mark>ec2:TerminateInstance</mark> : Allows terminating EC2 instances.
  - <mark>ec2:DescribeInstances</mark> : Allows describing EC2 instances (retrieving information about EC2 instances).
  - <mark>ec2:DescribeLaunchTemplates</mark> : Allows describing EC2 launch templates.
  - <mark>ec2:DescribeSubnets</mark> : Allows describing subnets (retrieving information about subnets).
  - <mark>ec2:DescribeSecurityGroups</mark> : Allows describing security groups (retrieving information about security groups).
  - <mark>ec2:DescribeInstanceTypes</mark> : Allows describing instance types (retrieving information about EC2 instance types).
  - <mark>iam:PassRole</mark> : Allows passing an IAM role to an AWS service (needed for services like EC2 to assume a role).

  ## Resource
  ```sh
  "Resource": "*"
  ```
  The Resource element specifies the resources to which the actions apply. The asterisk (*) means that the actions are allowed on all resources. In other words, this policy grants the specified actions on any EC2 or IAM resource in the AWS account.

## Summary

This IAM policy grants permissions to perform several EC2-related actions (such as creating and managing instances, launch templates, and fleets, as well as describing various EC2 resources) and allows passing IAM roles to AWS services. The policy applies to all resources (*), meaning the permissions are not restricted to specific EC2 instances, subnets, or security groups. This kind of policy is typically used by applications or services that need to dynamically manage EC2 resources and requires broad permissions to do so.