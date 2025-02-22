---
type: docs
title: "Integrate Azure Defender with Cluster API as an Azure Arc Connected Cluster using Kubernetes extensions"
linkTitle: "Integrate Azure Defender with Cluster API as an Azure Arc Connected Cluster using Kubernetes extensions"
weight: 1
description: >
---

## Integrate Azure Defender with Cluster API as an Azure Arc Connected Cluster using Kubernetes extensions

The following README will guide you on how to enable [Azure Defender](https://docs.microsoft.com/en-us/azure/security-center/defender-for-kubernetes-introduction) for a Cluster API that is projected as an Azure Arc connected cluster.

In this guide, you will hook the Cluster API to Azure Defender by deploying the [Azure Defender extension](https://docs.microsoft.com/es-es/azure/security-center/defender-for-kubernetes-azure-arc?tabs=k8s-deploy-cli%2Ck8s-verify-cli%2Ck8s-remove-arc) on your Kubernetes cluster in order to start collecting security related logs and telemetry.  

> **Note: This guide assumes you already deployed a Cluster API and connected it to Azure Arc. If you haven't, this repository offers you a way to do so in an automated fashion using a [Shell script](https://azurearcjumpstart.io/azure_arc_jumpstart/azure_arc_k8s/cluster_api/capi_azure/).**

Kubernetes extensions are add-ons for Kubernetes clusters. The extensions feature on Azure Arc enabled Kubernetes clusters enables usage of Azure Resource Manager based APIs, CLI and portal UX for deployment of extension components (Helm charts in initial release) and will also provide lifecycle management capabilities such as auto/manual extension version upgrades for the extensions.

## Prerequisites

* Clone the Azure Arc Jumpstart repository

    ```shell
    git clone https://github.com/microsoft/azure_arc.git
    ```

* [Install or update Azure CLI to version 2.15.0 and above](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest). Use the below command to check your current installed version.

  ```shell
  az --version
  ```

* Create Azure service principal (SP)

    To be able to complete the scenario and its related automation, Azure service principal assigned with the “Contributor” role is required. To create it, login to your Azure account run the below command (this can also be done in [Azure Cloud Shell](https://shell.azure.com/)).

    ```shell
    az login
    az ad sp create-for-rbac -n "<Unique SP Name>" --role contributor
    ```

    For example:

    ```shell
    az ad sp create-for-rbac -n "http://AzureArcK8s" --role contributor
    ```

    Output should look like this:

    ```json
    {
    "appId": "XXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "displayName": "AzureArcK8s",
    "name": "http://AzureArcK8s",
    "password": "XXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "tenant": "XXXXXXXXXXXXXXXXXXXXXXXXXXXX"
    }
    ```

    > **Note: The Jumpstart scenarios are designed with as much ease of use in-mind and adhering to security-related best practices whenever possible. It is optional but highly recommended to scope the service principal to a specific [Azure subscription and resource group](https://docs.microsoft.com/en-us/cli/azure/ad/sp?view=azure-cli-latest) as well considering using a [less privileged service principal account](https://docs.microsoft.com/en-us/azure/role-based-access-control/best-practices)**

## Automation Flow

For you to get familiar with the automation and deployment flow, below is an explanation.

* User has deployed Kubernetes using Cluster API and has it connected as Azure Arc enabled Kubernetes cluster.

* User is editing the environment variables on the Shell script file (1-time edit) which then be used throughout the extension deployment. 

* User is running the shell script. The script will use the extension management feature of Azure Arc to deploy the Azure Defender extension on the Azure Arc connected cluster.

* User is veryfing the cluster is shown in Azure Defender and that the extension is deployed.

* User is simulating a security alert on the Kubernetes cluster that will cause Azure Defender to detect this activity and trigger a security alert.

## Create Azure Defender extensions instance

To create a new extension Instance, we will use the _k8s-extension create_ command while passing in values for the mandatory parameters. This scenario provides you with the automation to deploy the Azure Defender extension on your Azure Arc enabled Kubernetes cluster.

* Before integrating the cluster with Azure Defender, click on the "Security (preview)" blade for the connected Arc cluster to show how the cluster is not currently being assessed by Azure Defender.

    ![Screenshot showing Azure Portal with Azure Arc enabled Kubernetes resource](./01.png)

    ![Screenshot showing Azure Portal with Azure Arc enabled Kubernetes resource extensions](./02.png)

* Edit the environment variables in the script to match your environment parameters followed by running the ```. ./azure_defender_k8s_extension.sh``` command.

    ![Screenshot parameter examples](./03.png)

    > **Note: The extra dot is due to the shell script has an *export* function and needs to have the vars exported in the same shell session as the rest of the commands.**

   The script will:

  * Login to your Azure subscription using the SPN credentials
  * Add or Update your local _connectedk8s_ and _k8s-extension_ Azure CLI extensions
  * Create Azure Defender k8s extension instance

* You can see that Azure Defender is enabled once you visit the security tab section of the Azure Arc enabled Kubernetes cluster resource in Azure.

![Screenshot extension deployment security tab](./04.png)

* Also verify under the Extensions section of the Azure Arc enabled Kubernetes cluster that the Azure Defender extension is correctly installed.

![Screenshot extension deployment](./05.png)

* You can also verify the deployment by running the command below:

```bash
kubectl get pods -n azuredefender --kubeconfig <cluster-name>.kubeconfig
```

![Screenshot extension deployment on cluster](./06.png)

## Simulate a security alert

To verify that Azure Defender is working properly and alerting on security threats, run the below command to simulate an alert:

```bash
kubectl get pods --namespace=asc-alerttest-662jfi039n --kubeconfig <cluster-name>.kubeconfig
```

Within 30 minutes Azure Defender will detect this event and trigger a security alert that you wil see in the Azure Portal under Azure Security Center's security alerts and also on the security tab of your Azure Arc enabled cluster.

![Screenshot security alert in Azure Security Center](./07.png)

![Screenshot security alert in Azure Security Center](./08.png)

![Screenshot security alert in Azure Security Center](./09.png)

### Delete extension instance

The following command only deletes the extension instance, but doesn't delete the Log Analytics workspace or disables Azure Defender.

```bash
az k8s-extension delete --name microsoft.azuredefender.kubernetes --cluster-type connectedClusters --cluster-name <cluster-name> --resource-group <resource-group>
```
