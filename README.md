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

Update your kubeconfig to use the new cluster:

    aws eks --region us-west-2 update-kubeconfig --name my-cluster

# Step 2: Install Helm and Karpenter
1. Add Karpenter Helm Repository:

    ```sh
    helm repo add karpenter https://charts.karpenter.sh
    helm repo update
    ```