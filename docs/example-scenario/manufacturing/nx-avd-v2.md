---
title: Deploy Siemens NX/X in Azure Virtual Desktop
author: Sunita Phanse
ms.date: 04/29/2025
---
Deploy Siemens NX/X in Azure Virtual Desktop 

This article defines the baseline architecture for implementing Siemens NX with Azure Virtual Desktop. Siemens NX software helps designers and manufacturers deliver better products faster using powerful, integrated CAD and CAM solutions that realize the full value of the digital twin. Consolidating NX with Azure Virtual Desktop provides a consistent and synchronized CAD /CAM experience across your enterprise.

Many customers run multiple NX solutions across the enterprise, mixing multiple instances, multiple ISV vendors, and hybrid cloud and on-premises implementations. We can host the NX on Azure Virtual Desktop.

## Architecture

## 11Your text here11Your text hereYour text hereYour text hereWorkflow

![](media/image3.png)

![](media/image3.png)

1. NX users access the NX application deployed on Azure Virtual Desktop via Remote Desktop Application (RDP). User can access the application either login into the Session Desktop or via remote application(streaming). 
1. Users' identity and credential must be set in Siemens Azure subscription to access the valid NX X licenses. Use accesses are granted access through their companies Microsoft Entra ID. Siemens cloud subscription is required. Users, who are assigned to the application group can access the Session Desktop and NX application from the workspace in Remote Desktop application
1. Azure Virtual Desktop control plane seamlessly manages the web access, Gateway, broker & diagnostic and extensibility components such as REST APIs. 
1. Azure Virtual Desktop Host Pools manages the session hosts, application groups, user assignments.
   - Session Hosts: Session hosts are GPU enabled Azure virtual machines. Administrator will deploy the NX on session hosts with the common NX product image.  
   - Application groups: AVD supports multiple Application groups. Application group is collection of Session desktop and NX remote application, MS 360 application or any other CAD applications.
   - Workspace: All these application groups should be grouped under a workspace which will be displayed in Remote desktop app
   - User Assignments: CAD user is assigned to the workspace and application groups. 

   

1. NX CAD files are stored in Azure Files or in Azure NetAPP files storage. User profiles are managed by FsLogiX which is stored in Azure storage- Fileshare/Azure Netapp file

##  Remote Desktop Client & Connection:

1. Download the [Remote Desktop client](/previous-versions/remote-desktop-client/connect-windows-cloud-services).
1. Users subscribes their ID in the Remote Desktop Application. Upon subscribing, users can see the Session Desktop and NX application.
1. Users can access Session Desktop with their credentials and separate Desktop session will be opened from where they can access NX. Users can create Part, drawing and save it in the corresponding session desktop, which will save in the corresponding VMs where the user was logged in
1. Users can access NX Remote app from Remote Desktop application after entering their credentials. NX application will be open locally, users can create part and drawing and save it to the corresponding VM from the where the NX is launched
1. Users can access NX Remote app from Remote Desktop application after entering their credentials. NX application will be open locally, users can create part and drawing and save it to the corresponding VM from the where the NX is launched

## Components

This architecture consists of the following Azure components.

[Azure Virtual Network](https://azure.microsoft.com/services/virtual-network/): Azure Virtual Network is a service that facilitates secure communication between Azure resources, the internet, and on-premises networks. In NX AVD, you can use it to create a secure network infrastructure for the NX AVD, allowing safe and reliable communication between them.

[Virtual machines](https://azure.microsoft.com/services/virtual-machines/): Azure Virtual Machines is an IaaS that provides on-demand, scalable computing resources without the need for physical hardware maintenance. Virtual machines provide the computing infrastructure that hosts the NX applications.

[Azure Files](https://azure.microsoft.com/products/storage/files): Azure Files is a service that offers shared storage and allows you to create a hierarchical folder structure to upload files. In a NX AVD deployment, it allows to store the FS Logix Session profiles

[Azure NetApp Files](https://azure.microsoft.com/services/netapp): Azure NetApp Files is a file-storage service developed jointly by Microsoft and NetApp. You can use Azure NetApp Files to allows to store the FS Logix Session profiles

[Microsoft Entra ID](https://azure.microsoft.com/products/active-directory): Microsoft Entra ID provides on-premises directory synchronization and single sign-on features. You can use Microsoft Entra ID to manage and authenticate users, providing seamless access to NX AVD from Remote Desktop App

[Network security groups](/azure/virtual-network/network-security-groups-overview): Network security groups are used to limit access to subnets within the Azure network. For NX AVD deployment, you use network security groups to secure the network infrastructure, ensuring that only authorized traffic can access the NX AVD and Session Desktop

[Azure Application Gateway](https://azure.microsoft.com/services/application-gateway/): Azure Application Gateway is a web traffic load balancer that manages traffic to web applications. You use it to manage and distribute traffic to the NX AVD services, improving performance and reliability.

[Azure Virtual Desktop](https://azure.microsoft.com/services/virtual-desktop): Azure Virtual Desktop is a desktop and app virtualization service that runs on Azure. Deliver a full Windows experience with Windows 11, Windows 10, or Windows Server. Use single session to assign devices to a single user or use multi-session for scalability. You use it to provide users with a virtualized desktop environment for CAD workstation, facilitating access NX from anywhere.

[Azure Virtual Desktop for the enterprise - Azure Architecture Center | Microsoft Learn](/azure/architecture/example-scenario/azure-virtual-desktop/azure-virtual-desktop)

[Remote Desktop Client](/previous-versions/remote-desktop-client/connect-windows-cloud-services) : Remote Desktop enables you to connect to Windows desktops and apps on a remote computer over a network connection using the Remote Desktop Protocol. You can connect to Windows hosted in the cloud from Azure Virtual Desktop along with Remote Desktop Services on-premises, and point-to-point connections with remote PCs.

[Azure Firewall](https://azure.microsoft.com/products/azure-firewall): Azure Firewall is a cloud-native network firewall security service that provides threat protection for cloud workloads. For a NX AVD deployment, Azure Firewall can be used to protect the NX AVD services from threats.

[Host pool](/azure/virtual-desktop/terminology) :A host pool is a collection of Azure virtual machines that are registered to Azure Virtual Desktop as session hosts. All session host virtual machines in a host pool should be sourced from the same image for a consistent user experience. 

[Session host Configuration:](/azure/virtual-desktop/host-pool-management-approaches) A session host configuration is a sub-resource of the session host configuration management approach that specifies the configuration of session hosts in the host pool. The session host configuration persists throughout the lifecycle of the host pool and is aligned with the session hosts in the host pool.

[Create Image](/azure/virtual-machines/capture-image-portal) :  Cloud administrator creates an image of the OS with required NX software, user profile data and any other necessary files. This image will be used to create session hosts. An image can be created from a VM and then used to create multiple VMs. For images stored in an Azure Compute Gallery (formerly known as Shared Image Gallery), you can use VMs that already have accounts created on them (specialized) or you can generalize the VM before creating the image to remove machine accounts and other machines specific information. To generalize a VM, see [Generalized a VM](/azure/virtual-machines/generalize).

[Appplication group](/azure/virtual-desktop/terminology): An application group controls access to a full desktop or a logical grouping of applications that are available on session hosts in a single host pool. Users can be assigned to multiple application groups across multiple host pools, which enable you to vary the applications and desktops that users can access.  When you create an application group, it can be one of two types:** **

- **Desktop**: users access the full Windows desktop from a session host**       **
  - **RemoteApp**: users access individual applications you select and publish to the application group. Available with pooled host pools only.

[Workspace](/azure/virtual-desktop/terminology): A workspace is a logical grouping of application groups. Each application group must be associated with a workspace for users to see the desktops and applications published to them. An application group can only be assigned to a single workspace.

[FSLogix](/fslogix/overview-what-is-fslogix) enhances and enables user profile management for Windows remote computing environments. It allows users to roam between remote computing session hosts, minimize sign-in times for virtual desktop environments, and optimize file I/O between the host/client and the remote profile store. For information about FSLogix Profile Container, Azure Files, and Azure NetApp Files best practices, see FSLogix configuration examples

## Potential use cases

. The global reach of Microsoft Azure cloud combined with leading technology investments in areas including GPUs means that ‘power user' workstation experiences can be leveraged by engineers wherever you are, on whatever device you use. Cloud-hosted engineering workstations on Azure brings high-end workstation desktop power and their apps to engineers - securely - no matter where they are to improve productivity, all the while reducing costs and giving a consistent user experience. 

1. NX AVD solution provides easy access to NX users. User an connect seamlessly NX from anywhere 
1. No need to install NX application locally in all User workstation. They can access from Remote Desktop application
1. All the NX data are stored in the Azure VM storage. It avoids users storing the NX data locally in their workstation
1. Based on the user load, NX AVD can scale up or scale down to meet the requirement
1. Modern and diverse GPU based compute options to align to CAD workload's needs
1. The flexibility of virtualization without the need to buy and maintain physical hardware
1. Rapid provisioning Auto Scaling - Increase or decrease capacity based on needs
1. NX- Multi-session is an exclusive feature for Azure Virtual Desktop and drives cost efficiency without impacting user experience
1. Reduce costs with pooled, multi-session resources. 
1. With the new Windows 11 and Windows 10 Enterprise multi-session capability, exclusive to Azure Virtual Desktop, or Windows Server, you can greatly reduce<br>the number of virtual machines and operating system overhead while still providing the same resources to<br>your users.

   

- 

##    Testing & Workloads

     User can  the NX application by [NXATS](https://support.sw.siemens.com/) (NX Automated test suit )and [NXCP](https://support.sw.siemens.com/) (NX Certification Pack) to validate the NX application in AVD.  NX ATS test suit runs both in manual and automated mode. NXCP runs in Auto and Interactive mode. Contact Siemens support [GTAC](https://support.sw.siemens.com/) to download the Test suits and installation instructions.

## Workloads

Users can run different types of workloads on the session host virtual machines. The following table shows examples of a range of workload types to help you estimate what size your virtual machines need to be. After you set up your virtual machines, continually monitor their actual usage and adjust their size accordingly. If you end up needing a bigger or smaller virtual machine, scale your existing deployment up or down.

AVD multi-Session example Scenarios:

![A white sheet with black text  AI-generated content may be incorrect.](media/image4.png)

## User Preference setting

Users can set their NX preference such as Appearance, Units, Graphics preference, Modelling Preference, Toolbars and Menus and can be saved and imported in the session. User roles can be assigned and loaded for the session. Additionally, user Defaults, Profiles, options can be defined through environment variables as well.

## Licensing

Local and cloud (NX X)

NX licensing can be obtained from Local Siemens licensing server for NX. License server can be hosted in Azure environment or On-prem, NX obtains the license in both the cases.  For NX X, the cloud application, license is obtained from Siemens cloud.

## Mapped Drive installation

   Storage drive can be mapped in the AVD setup which allows multiple users to save, access the files from the storage instead from local PC. Designers need a collaborative workspace drive to access their files. All the model, drawing files can be stored in the central mapped drive 

## Considerations

These considerations align to the pillars of the Azure Well-Architected Framework. A set of guiding tenets that can be used to improve the quality of a workload. For more information, see [Microsoft Azure Well-Architected Framework](/azure/architecture/framework).

<br>

## Performance

Performance efficiency is the ability of your workload to scale to meet the demands placed on it by users in an efficient manner. For more information, see [Performance efficiency pillar overview](/azure/architecture/framework/scalability/overview). GPU accelerated solutions to optimize cost and performance with cloud-hosted engineering workstations. Cloud-hosted engineering workstations are available on-demand, globally and securely accessible through Azure Virtual Desktop VDI. 

The latency between the end user and the RDP session needs to be measured prior to production go-live.  This latency helps to ensure that, when NX AVD users interact with maps and perform measurements or edits, the interactive edits and the tooltips appear quickly enough. The [Azure Virtual Desktop Experience Estimator](https://azure.microsoft.com/services/virtual-desktop/assessment) can provide a quick assessment of connection round-trip time (RTT) from your location, through the Azure Virtual Desktop service, and to each Azure region in which you can deploy virtual machines.

When you use a remote Windows session, your network's available bandwidth greatly affects the quality of your experience. The following table lists the minimum recommended bandwidths for a smooth user experience. These recommendations are based on the guidelines in [Remote Desktop workloads](/windows-server/remote/remote-desktop-services/remote-desktop-workloads).

## Scalability

You can scale this architecture in many ways. You can scale the VMs for the back end or the desktops (both CPU and GPUs) in, scale out, scale up, or scale down. You can also deploy Azure Virtual Desktop on individual VMs or multi-session VMs. Azure Virtual Desktop can scale hundreds or thousands of VMs. For more information, see [Windows 10 or Windows 11 Enterprise multi-session remote desktops](/mem/intune/fundamentals/azure-virtual-desktop-multi-session).

- 

## Reliability

Reliability ensures your application can meet the commitments you make to your customers. For more information, see [Overview of the reliability pillar](/azure/architecture/framework/resiliency/overview). In general, consider Availability zones or Availability sets based on requirements of multisite implementations. For more information, see [High availability and disaster recovery for IaaS apps](/azure/architecture/example-scenario/infrastructure/iaas-high-availability-disaster-recovery)

1. Multi host session where multiple VMs are running under host pool and session load is distributed breadth first or Depth first pools

## Security

           NX on AVD provides access control only for the authorised users through MultiFactor authentication, Role-Based Access Control. Only user with specific roles and permissions to manage AVD resources and NX software. Data Protection prevent sensitive data from leaving the AVD environment. Minimize the risk of users accessing unauthorized files. Robust Data backup and recovery strategy for NX data and models.

Azure Security provides assurances against deliberate attacks and the abuse of your valuable data and systems. For more information, see [Overview of the security pillar](/azure/architecture/framework/security/overview). Only the Assigned users in Microsoft Entra ID can access the NX AVD application

## Cost Optimization

Cost optimization is about looking at ways to reduce unnecessary expenses and improve operational efficiencies. For more information, see [Overview of the cost optimization pillar](/azure/architecture/framework/cost/overview).

**Consider constrained vCPU virtual machines.** If your workload requires more memory and fewer CPUs, consider using one of [constrained vCPU virtual machine](/azure/virtual-machines/constrained-vcpu) sizes to reduce software licensing costs that are charged per vCPU.

**Use the right virtual machine SKUs.** You should use the virtual machine SKUs in the following table. Contact Siemens support team for the latest NX on Azure certification matrix and SKU recommendations.

The Azure calculator can help you estimate and optimize cost. For an estimated cost of the baseline architecture, see [estimated cost](https://azure.com/e/625cea91d4aa43bca73e0a8235817ba7). Your estimates might differ based on your NX AVD implementation

## SKU Recommendations

| **SKU Name** | **vCPUs** | **Memory** | **Suitable For** |
|:---:|:---:|:---:|:---:|
| **Standard_NV6ads_A10_v5** | 6 | 55 GB | GPU-intensive CAD applications |
| **Standard_NV12ads_A10_v5** | 12 | 110 GB | GPU-intensive CAD applications |
| **Standard NV24s v3** | 24 | 224 GB | Demanding CAD workloads |

**Single-session recommendations**

_Single-session_ scenarios are when there's only one user assigned to a session host virtual machine at any one time. For example, if you use personal host pools in Azure Virtual Desktop, you're using a single-session scenario.

The following table provides examples for single-session NX AVD scenarios:

| **Workload type** | **Example Azure virtual machine SKU** | **Activity** |
|---|---|---|
| **Light** | NV6ads_A10_v5 | Viewing |
| **Medium** | Standard_NC8as_T4_v3,Standard_NC16as_T4_v3 | Editing |
| **Heavy** | Standard_NV12ads_A10_v5 | Visualizing |

**Multi-session recommendations**

_Multi-session_ scenarios are when there's more than one user signed in to a session host at any one time. For example, when you use pooled host pools in Azure Virtual Desktop with the Windows 11 Enterprise multi-session operating system (OS), that's a multi-session deployment.

The following table provides examples for multi-session NX AVD scenarios:

| **Workload type** | **Example Azure virtual machine SKU** | **Maximum users per VM** | **Activity** |
|---|---|---|---|
| **Light** | Standard_NV12ads_A10_v5, NVv4 Series | 5 | Viewing |
| **Medium** | Standard_NV12ads_A10_v5, NVv4 Series | 5 | Editing |
| **Heavy** | Standard_NV18ads_A10_v5, NVv4 Series | 5 | Visualizing |

| **          **<br> | | |
|---|---|---|
