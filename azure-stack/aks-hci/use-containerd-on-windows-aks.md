---

title: Use containerd for Windows nodes in Azure Kubernetes Service on Azure Stack HCI and Windows Server (preview)
description: Use containerd as the container runtime for Windows Server node pools on Azure Kubernetes Service on Azure Stack HCI and Windows Server.
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.date: 03/07/2022
ms.lastreviewed: 03/07/2022
ms.reviewer: crwilhit

# Intent: As a Kubernetes cluster administrator, I want to use containerd as my runtime so that my cluster is ready for the deprecation of dockershim.
# Keyword: containerd Windows AKS HCI

---

# Use containerd for Windows nodes in Azure Kubernetes Service on Azure Stack HCI and Windows Server (preview)

> Applies to: Azure Stack HCI, versions 21H2 and 20H2; Windows Server 2022 Datacenter, Windows Server 2019 Datacenter

Beginning in Kubernetes version v1.22.1, you can use `containerd` as the container runtime for Windows Server node pools. The use of the `containerd` runtime for Windows on Azure Kubernetes Service (AKS) on Azure Stack HCI and Windows Server is currently in **preview**. While dockershim remains the default runtime for now, it's deprecated and will be removed in Kubernetes v1.24.

> [!IMPORTANT]  
> The `containerd` runtime for Windows on AKS on Azure Stack HCI and Windows Server is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Prerequisites

Verify you have the following requirements ready:

- You've prepared your machine [for deployment](/azure-stack/aks-hci/prestage-cluster-service-host-create#step-2-prepare-your-machines-for-deployment).
- You have the [AksHci PowerShell module](./kubernetes-walkthrough-powershell.md#install-the-akshci-powershell-module) installed.
- You can run commands in this article from an elevated PowerShell session.
## Set the configuration

Before your deploy your AKS cluster on Azure Stack HCI, set the cluster to be deployed with the `containerd` runtime for Windows nodes. To set the runtime to `containerd`, you'll use the **Set-AksHciConfig** PowerShell cmdlet with the `-ring wincontainerd` and `-catalog aks-hci-stable-catalogs-ext` flags.

In the following steps, the values of the parameters are given, but you'll need to update these values for your specific environment.

1. Run Windows PowerShell as an Administrator.
1. You'll need the following parameters to run the **[Set-AksHciConfig](./reference/ps/set-akshciconfig.md)** cmdlet:
    1. **workingDir**  
        Specify a working directory for the module to use for storing small files, for example: `c:\ClusterStorage\Volume1\ImageStore`.
    1. **cloudConfigLocation**  
        Specify where the cloud agent will store its configuration, for example: `c:\clusterstorage\volume1\Config`
    1. **Version**  
        The version of AKS on Azure Stack HCI and Windows Server that you want to deploy, for example `v1.22.1`.
    1. **vnet**  
        The name of the **AksHciNetworkSetting** object created with **New-AksHciNetworkSetting** command. For an example of the cmdlet that stores the result in the `$vnet` variable, see [Create a virtual network](./kubernetes-walkthrough-powershell.md#step-2-create-a-virtual-network).
    1. **imageDir**  
        The path to the directory where AKS on Azure Stack HCI and Windows Server will store its VHD images, for example `c:\clusterstorage\volume1\Images`
1. Run the following cmdlet. The values given in this example command will need to be customized for your environment.
    ```powershell
    Set-AksHciConfig -workingDir c:\ClusterStorage\Volume1\ImageStore -Version v1.22.1 -vnet $vnet -imageDir $c:\clusterstorage\volume1\Images -skipHostLimitChecks -ring wincontainerd -catalog aks-hci-stable-catalogs-ext
    ```
## Deploy a cluster

Deploy your AKS on Azure Stack HCI and Windows Server cluster.
1. Run Windows PowerShell as an Administrator on any node in your Azure Stack HCI or Windows Server cluster.
1. Run the following cmdlet:
    ```PowerShell
    Install-AksHCI $VerbosePreference = "Continue"
    ```
1. Verify that `containerd` is the container runtime with kubectl. Run the following command from a PowerShell or bash prompt:
    ```PowerShell
    kubectl get nodes -o wide
    ```
    Kubectl returns the nodes for your cluster. For example:
    ```output
    PS C:\Users\azureuser> kubectl get nodes -o 
    NAME                                STATUS   ROLES                  AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION    CONTAINER-RUNTIME
    moc-llnr3iekoz5                     Ready    control-plane,master   21m     v1.22.2   192.168.0.5   <none>        CBL-Mariner/Linux                5.10.78.1-2.cm1   containerd://1.4.4
    moc-wh4rlkcxn1n                     Ready    <none>                 4m38s   v1.22.2   192.168.0.6   <none>        Windows Server 2019 Datacenter   10.0.17763.2565   containerd://1.6.0-beta.3-82-ga95a8b8ff+azure
    ```
1. Verify your runtime is `containerd` in the `container-runtime` column.

## Known issues

You may encounter the following issues when using `containerd` on AKS on Azure Stack HCI and Windows Server.

### Issues accessing SMB shares from a pod configured with GMSA

When a Windows pod is configured with GMSA and the runtime is `containerd`, pods sometimes have difficulty accessing SMB shares. If this occurs, you can work around this by opening an elevated PowerShell session on the target node and run the following:

```powershell  
reg add "HKLM\SYSTEM\CurrentControlSet\Services\hns\State" /v EnableCompartmentNamespace /t REG_DWORD /d 1
```

Reboot the node after setting this reg key in order to apply the change.

> [!IMPORTANT]  
> The `containerd` preview doesn't currently support Flannel Overlay networking. You must use Calico.
## Next steps

- [Deploy .NET applications](deploy-windows-application.md).
- [Monitor AKS on Azure Stack HCI and Windows Server clusters](monitor-logging.md).