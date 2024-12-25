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

3.