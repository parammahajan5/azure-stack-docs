---
title: Upgrade a Kubernetes cluster on Azure Stack Hub 
description: Learn how to upgrade a Kubernetes cluster on Azure Stack Hub. 
author: sethmanheim

ms.topic: article
ms.date: 3/4/2021
ms.author: sethm
ms.reviewer: waltero
ms.lastreviewed: 3/4/2021

# Intent: Notdone: As a < type of user >, I want < what? > so that < why? >
# Keyword: Notdone: keyword noun phrase

---


# Upgrade a Kubernetes cluster on Azure Stack Hub

The AKS engine allows you to upgrade the cluster that was originally deployed using the tool. You can maintain the clusters using the AKS engine. Your maintenance tasks are similar to any IaaS system. You should be aware of the availability of new updates and use the AKS engine to apply them.
## Upgrade a cluster

The upgrade command updates the Kubernetes version and the base OS image. Every time that you run the upgrade command, for every node of the cluster, the AKS engine creates a new VM using the AKS Base Image associated to the version of **aks-engine** used. You can use the `aks-engine upgrade` command to maintain the currency of every master and agent node in your cluster. 

Microsoft doesn't manage your cluster. But Microsoft provides the tool and VM image you can use to manage your cluster. 

For a deployed cluster upgrades cover:

-   Kubernetes
-   Azure Stack Hub Kubernetes provider
-   Base OS

When upgrading a production cluster, consider:

-   Are you using the correct cluster specification (`apimodel.json`) and resource group for the target cluster?
-   Are you using a reliable machine for the client machine to run the AKS engine and from which you are performing upgrade operations?
-   Make sure that you have a backup cluster and that it is operational.
-   If possible, run the command from a VM within the Azure Stack Hub environment to decrease the network hops and potential connectivity failures.
-   Make sure that your subscription has enough space for the entire process. The process allocates new VMs during the process.
-   No system updates or scheduled tasks are planned.
-   Set up a staged upgrade on a cluster that is configured exactly as the production cluster and test the upgrade there before doing so in your production cluster

## Steps to upgrade to a newer Kubernetes version

> [!NOTE]  
> The AKS base image will also be upgrade if you are using a newer version of the aks-engine and the image is available in the marketplace.

The following instructions use the minimum steps to perform the upgrade. If would like more detail, see the article [Upgrading Kubernetes Clusters](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping).

1. You need to first determine the versions you can target for the upgrade. This version depends on the version you currently have and then use that version value to perform the upgrade. The Kubernetes versions supported by your AKS Engine can be listed by running the following command:
    
    ```bash
    aks-engine get-versions --azure-env AzureStackCloud
    ```
    
    For a complete mapping of AKS engine, AKS Base Image and Kubernetes versions see [Supported AKS Engine Versions](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping).

2. Collect the information you will need to run the `upgrade` command. The upgrade uses the following parameters:

    | Parameter | Example | Description |
    | --- | --- | --- |
    | azure-env | AzureStackCloud | To indicate to AKS engine that your target platform is Azure Stack Hub use `AzureStackCloud`. |
    | location | local | The region name for your Azure Stack Hub. For the ASDK, the region is set to `local`. |
    | resource-group | kube-rg | Enter the name of a new resource group or select an existing resource group. The resource name needs to be alphanumeric and lowercase. |
    | subscription-id | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | Enter your Subscription ID. For more information, see [Subscribe to an offer](./azure-stack-subscribe-services.md#subscribe-to-an-offer) |
    | api-model | ./kubernetes-azurestack.json | Path to the cluster configuration file, or API model. |
    | client-id | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | Enter the service principal GUID. The Client ID identified as the Application ID when your Azure Stack Hub administrator created the service principal. |
    | client-secret | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | Enter the service principal secret. This is the client secret you set up when creating your service. |
    | identity-system | adfs | Optional. Specify your identity management solution if you are using Active Directory Federated Services (AD FS). |

3. With your values in place, run the following command:

    ```bash  
    aks-engine upgrade \
    --azure-env AzureStackCloud \
    --location <for an ASDK is local> \
    --resource-group kube-rg \
    --subscription-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --api-model kube-rg/apimodel.json \
    --upgrade-version 1.18.15 \
    --client-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --client-secret xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --identity-system adfs # required if using AD FS
    ```

4.  If for any reason the upgrade operation encounters a failure, you can rerun the upgrade command after addressing the issue. The AKS engine will resume the operation where it failed the previous time.

## Steps to only upgrade the OS image

1. Review [the supported-kubernetes-versions table](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping) and determine if you have the version of aks-engine and AKS base Image that you plan for your upgrade. To view the version of aks-engine run: `aks-engine version`.
2. Upgrade your AKS engine accordingly, in the machine where you have installed aks-engine run: `./get-akse.sh --version vx.xx.x` replacing **x.xx.x** with your targeted version.
3. Ask your Azure Stack Hub operator to add the version of the AKS Base Image you need in the Azure Stack Hub Marketplace that you plan to use.
4. Run the `aks-engine upgrade` command using the same version of Kubernetes that you are already using, but add the `--force`. You can see an example in [Forcing an upgrade](#forcing-an-upgrade).


## Steps to update cluster to OS version Ubuntu 18.04

With AKS engine version 0.60.1 and above you can upgrade your cluster VMs from Ubuntu 16.04 to 18.04. Follow these steps:

1. Locate and edit the `api-model.json` file that was generated during deployment. This should be the same file used for any upgrade or scale operation with `aks-engine`.
2. Locate the sections for `masterProfile` and `agentPoolProfiles`, within those sections change the value of `distro` to `aks-ubuntu-18.04`.
2. Save the `api-model.json` file and use the `api-model.json` file in your` aks-engin upgrade` command as you would in the [Steps to upgrade to a newer Kubernetes version](#steps-to-upgrade-to-a-newer-kubernetes-version)

## Forcing an upgrade

There may be conditions where you may want to force an upgrade of your cluster. For example, on day one you deploy a cluster in a disconnected environment using the latest Kubernetes version. The following day Ubuntu releases a patch to a vulnerability for which Microsoft generates a new **AKS Base Image**. You can apply the new image by forcing an upgrade using the same Kubernetes version you already deployed.

```bash  
aks-engine upgrade \
--azure-env AzureStackCloud   
--location <for an ASDK is local> \
--resource-group kube-rg \
--subscription-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
--api-model kube-rg/apimodel.json \
--upgrade-version 1.18.15 \
--client-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
--client-secret xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
--force
```

For instructions, see [Force upgrade](https://github.com/Azure/aks-engine/blob/master/docs/topics/upgrade.md#force-upgrade).

## Next steps

- Read about the [The AKS engine on Azure Stack Hub](azure-stack-kubernetes-aks-engine-overview.md)
- [Scale a Kubernetes cluster on Azure Stack Hub](azure-stack-kubernetes-aks-engine-scale.md)
