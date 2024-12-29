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
  
  ### Version
  ```sh
  "Version": "2012-10-17"
  ```
  This specifies the version of the policy language. The date "2012-10-17" indicates the version of the IAM policy language being used.

  ### Statement
  
  The Statement element contains one or more individual statements. Each statement grants permissions for specific actions on specific resources.

  ### Effect
  ```sh
  "Effect": "Allow"
  ```
  The Effect element specifies whether the statement allows or denies access. In this case, it is set to "Allow", meaning the actions listed in the Action element are permitted.

  ### Action
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

  ### Resource
  ```sh
  "Resource": "*"
  ```
  The Resource element specifies the resources to which the actions apply. The asterisk (*) means that the actions are allowed on all resources. In other words, this policy grants the specified actions on any EC2 or IAM resource in the AWS account.

## Summary

This IAM policy grants permissions to perform several EC2-related actions (such as creating and managing instances, launch templates, and fleets, as well as describing various EC2 resources) and allows passing IAM roles to AWS services. The policy applies to all resources (*), meaning the permissions are not restricted to specific EC2 instances, subnets, or security groups. This kind of policy is typically used by applications or services that need to dynamically manage EC2 resources and requires broad permissions to do so.

### Save the above JSON to a file named karpenter-iam-policy.json.
Create the policy in AWS:

The first command creates an IAM policy in AWS using the AWS CLI. The policy will be used to define the permissions for the Karpenter controller.

```sh
aws iam create-policy --policy-name KarpenterControllerPolicy --policy-document file://karpenter-iam-policy.json
```

- <mark>aws iam create-policy</mark>: This is the command to create a new IAM policy.
- <mark>--policy-name KarpenterControllerPolicy</mark>: This specifies the name of the policy you are creating, in this case, "KarpenterControllerPolicy".
- <mark>--policy-document file://karpenter-iam-policy.json</mark>: This specifies the path to the JSON file that contains the policy document. The file:// prefix indicates that the policy document is located in a file named karpenter-iam-policy.json on your local machine.

Create an IAM Role for Karpenter and Attach the Policy

The second set of commands creates an IAM service account for Karpenter in your EKS cluster and attaches the newly created policy to it.

```sh
eksctl create iamserviceaccount \
  --cluster=my-cluster \
  --namespace=karpenter \
  --name=karpenter \
  --attach-policy-arn=arn:aws:iam::<YOUR_ACCOUNT_ID>:policy/KarpenterControllerPolicy \
  --approve
```

- <mark>eksctl create iamserviceaccount</mark>: This is the command to create an IAM service account using eksctl, a CLI tool for managing Amazon EKS clusters.
- <mark>--cluster=my-cluster</mark>: This specifies the name of the EKS cluster where the IAM service account will be created. Replace my-cluster with the actual name of your EKS cluster.
- <mark>--namespace=karpenter</mark>: This specifies the namespace within the EKS cluster where the IAM service account will be created. In this case, it's the karpenter namespace.
- <mark>--name=karpenter</mark>: This specifies the name of the IAM service account. In this case, it's karpenter.
- <mark>--attach-policy-arn=arn:aws:iam::<YOUR_ACCOUNT_ID>:policy/KarpenterControllerPolicy</mark>: This specifies the ARN (Amazon Resource Name) of the policy to attach to the IAM service account. Replace <YOUR_ACCOUNT_ID> with your actual AWS account ID.
- <mark>--approve</mark>: This flag automatically approves the creation of the IAM service account without prompting for confirmation.

## Summary

- Create the IAM Policy: This policy document defines the permissions that the Karpenter controller will have.
- Create IAM Service Account for Karpenter: This service account is created in the specified EKS cluster and namespace, and the previously created policy is attached to it.

3. Install Karpenter using Helm:

```sh
helm install karpenter karpenter/karpenter --namespace karpenter --create-namespace \
  --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=arn:aws:iam::<YOUR_ACCOUNT_ID>:role/<IAM_ROLE_NAME> \
  --set settings.aws.clusterName=my-cluster \
  --set settings.aws.defaultInstanceProfile=KarpenterNodeInstanceProfile \
  --set settings.aws.interruptionQueueName=karpenter-interruption
```

    1 vulnerability detected
    Hardcoded Credentials
    Embedding credentials in source code risks unauthorized access

Explanation:
- helm install karpenter karpenter/karpenter: 
- This is the Helm command to install a chart. The chart name is karpenter, and it is located in the karpenter repository.

- --namespace karpenter: Specifies the namespace in which to install Karpenter. If the namespace does not exist, it will be created.

- --create-namespace: This flag tells Helm to create the namespace if it doesn't already exist.

- --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=arn:aws:iam::<YOUR_ACCOUNT_ID>:role/<IAM_ROLE_NAME>: This sets the annotation for the Karpenter service account to associate it with an IAM role. Replace <YOUR_ACCOUNT_ID> with your AWS account ID and <IAM_ROLE_NAME> with the name of the IAM role you created for Karpenter.

- --set settings.aws.clusterName=my-cluster: This sets the clusterName configuration for Karpenter to the name of your EKS cluster. Replace my-cluster with your actual EKS cluster name.

- --set settings.aws.defaultInstanceProfile=KarpenterNodeInstanceProfile: This sets the default instance profile that Karpenter will use for the nodes it provisions. The instance profile should be named KarpenterNodeInstanceProfile.

- --set settings.aws.interruptionQueueName=karpenter-interruption: This sets the name of the interruption queue that Karpenter will use to handle node interruptions. The queue should be named karpenter-interruption.

Summary:
This Helm command installs Karpenter in the karpenter namespace of your Kubernetes cluster. It configures the Karpenter service account with the necessary IAM role, sets the EKS cluster name, specifies the default instance profile for nodes, and configures the interruption queue. Replace the placeholders in the command with your actual AWS account ID, IAM role name, and EKS cluster name.