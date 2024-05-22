---
layout: post
title: "CNI Overlay for Azure Kubernetes Service"
date: 2024-03-30 15:18:00 +0300
categories: Azure Kubernetes
tags: azure aks kubernetes cni network
---

One of the critical components influencing AKS performance and functionality is its networking architecture, which plays a pivotal role in enabling seamless communication between pods, services, and external resources. When it comes to AKS networking, operators are presented with multiple options, each tailored to address specific use cases and requirements. At the forefront of these options are Container Network Interface (CNI) and Kubenet, each with its strengths and considerations.

In this post, we will explore the CNI overlay networking model, a (fairly) new but popular choice for AKS networking, and discuss its benefits, use cases, and considerations.

## Kubernetes Networking

Kubernetes networking is a complex topic that involves multiple components and configurations. At its core, Kubernetes networking is responsible for enabling communication between pods, services, and external resources. To achieve this, Kubernetes networking relies on a set of plugins and configurations that define how network traffic is routed, load balanced, and secured within the cluster. **Kubernetes nodes** are connected to a **virtual network**, so that pods (basic units of deployment in Kubernetes) can have both inbound and outbound connectivity. **Kube-proxy** runs on each node and is responsible for providing the necessary network features. In Azure you can implement the AKS networking with one of CNI options or the Kubenet.

### Kubenet

[Kubenet](https://learn.microsoft.com/en-us/azure/aks/configure-kubenet) is the default network option for AKS clusters. It is a simple and straightforward networking model that relies on the Azure Virtual Network (VNet) and Network Security Groups (NSG) to provide network connectivity to pods. In the Kubenet model, each node in the AKS cluster gets an IP address from the Azure VNet subnet, and pods are assigned IP addresses from a private address range. The kube-proxy component running on each node is responsible for routing traffic to the appropriate pods. While Kubenet is easy to set up and manage, it has some limitations, such as lack of support for network policies and limited scalability.

### CNI

[CNI](https://learn.microsoft.com/en-us/azure/aks/configure-azure-cni?tabs=configure-networking-portal) is a more advanced networking model that provides a flexible and extensible way to manage network connectivity in Kubernetes clusters. In the CNI model, each pod gets its IP address, and network traffic is routed through a set of plugins that define how traffic is handled. CNI plugins can be used to implement advanced networking features such as network policies, load balancing, and encryption. CNI is a popular choice for AKS networking due to its flexibility and scalability.

## Selecting the Right Networking Model

When choosing a networking model for your AKS cluster, it is essential to consider your specific requirements and use cases. Here are some factors to consider when selecting a networking model:

- **Scalability**: If you need to scale your cluster to hundreds or thousands of nodes, you should consider using a _CNI model_ that can handle the increased traffic and workload.
- **Security**: If you need to enforce network policies, encryption, or other security features, you should choose a _CNI model_ that supports these features.
- **Performance**: If you need low latency and high throughput for your applications, you should choose a _CNI model_ that can provide the necessary performance.
- **Ease of Use**: If you are looking for a simple and easy-to-manage networking model, you may want to consider using _Kubenet_.
- **Support for Windows node pools**: If you need to run Windows containers in your AKS cluster, you should choose a _CNI model_ that supports Windows node pools.

In reality apart from ease of use the biggest advantage of kubenet against CNI is that it requires far smaller number of IP addresses, because each pod gets an IP address from the subnet of the node it is running on. In CNI each pod gets an IP address from the subnet of the virtual network. This means that in CNI you need to have a larger subnet to accommodate the pods. Imagine an AKS cluster that is configured to support 30 pods per node, and consists of 10 nodes. In Kubenet you would need a subnet that can accommodate 10 IP addresses (one for every node), while in CNI you would need a subnet that can accommodate 300 IP addresses for the pods plus 10 for the nodes. This is a significant difference and unfortunately in many real world scenarios I have seen customer to opt in for kubenet just for that reason.

However, you should have in mind that while _kubenet_ is the default networking option for an AKS cluster to create a virtual network and subnet, it is **not recommended** for production
deployments. For most [production deployments(https://learn.microsoft.com/en-us/azure/aks/concepts-network#kubenet-basic-networking)], you should plan for and use Azure CNI networking due to its superior scalability and performance characteristics. But what if you need to use CNI but you want to have the same IP address management as in kubenet? This is where the CNI overlay networking comes into play.

## CNI Overlay Networking

[CNI overlay](https://learn.microsoft.com/en-us/azure/aks/azure-cni-overlay?tabs=kubectl) networking is a specific implementation of the CNI model that uses overlay networks to provide network connectivity to pods. In the overlay networking model, each pod gets an IP address from a private address range, and network traffic is encapsulated and routed through an overlay network. Network Address Translation (NAT) uses the node's IP address to reach resources outside the cluster. This solution saves a significant amount of VNet IP addresses and enables you to scale your cluster to large sizes. An extra advantage is that you can reuse the private CIDR in different AKS clusters, which extends the IP space available for containerized applications in Azure Kubernetes Service (AKS).

![AKS CNI Overlay](/images/aks-cni/cni-overlay-overview.jpg){: width="400" }

### Basic Differences between Kubenet and Azure CNI Overlay

CNI Overlay has the same advantage as kubenet against plain CNI regarding the POD CIDR space requirements, but this is all what they share in common. The Azure CNI Overlay is a superior networking model and provides the following benefits over Kubenet:

|                              | Azure CNI Overlay                      | Kubenet                                                                            |
| ---------------------------- | -------------------------------------- | ---------------------------------------------------------------------------------- |
| Cluster scale                | 5000 nodes and 250 pods/node           | 400 nodes and 250 pods/node                                                        |
| Network policies             | Azure Network Policies, Calico, Cilium | Calico                                                                             |
| OS support                   | Linux and Windows                      | Linux                                                                              |
| Pod connectivity Performance | High                                   | Lower - extra hop adds minor latency                                               |
| Network configuration        | Simple                                 | More complex - requires route tables and UDRs on cluster subnet for pod networking |

### IP Address Planning Considerations and other limitations

When planning your AKS cluster, you should consider the following IP address requirements:

- The same pod CIDR space can be used on multiple independent AKS clusters in the same VNet. This is because the overlay network is isolated from the VNet and does not require a unique IP address space. Hence, don't save one the POD CIDR, it is recommended to use a /16 for pod CIDR. By default the CNI Overlay will give [each node a /24 subnet](https://learn.microsoft.com/en-us/azure/aks/azure-cni-overlay?tabs=kubectl#ip-address-planning), so you need a large enough POD CIDR to accommodate appropriate scaling.
- Pod CIDR space must not overlap with the cluster subnet range
- Pod CIDR space must not overlap with directly connected networks (like VNet peering, ExpressRoute, or VPN)

Other limitations that you might need to consider are:

- You can't use [Application Gateway as an Ingress Controller (AGIC)](https://learn.microsoft.com/en-us/azure/application-gateway/ingress-controller-overview)] for a CNI Overlay cluster - you can however use [Application Gateway for Containers-AGC](https://learn.microsoft.com/en-us/azure/application-gateway/for-containers/) (hope to write a blog soon about this)
- The name of the subnet, virtual network and resource group which contains the network, must be 63 characters or less. This comes from the fact that these names will be used as labels in AKS worker nodes, and are therefore subjected to Kubernetes label syntax rules.

## Set up CNI Overlay AKS Cluster

To create an AKS cluster with CNI overlay networking, you need to specify the `--network-plugin azure` and `--network-plugin-mode overlay` options when creating the cluster. You also need to specify the `--pod-cidr` option to define the IP address range for pods in the cluster. Here is an example of how to create an AKS cluster with CNI overlay networking:

```bash
MY_SUBSCRIPTION_ID=<subscription>
CLUSTERNAME="aks-cni-01"
RG="rg-aks-cni"
LOCATION="northeurope"
VNET_NAME="vnet-aks"
AKS_SUBNET_ID="/subscriptions/$MY_SUBSCRIPTION_ID/resourceGroups/$RG/providers/Microsoft.Network/virtualNetworks/$VNET_NAME/subnets/aks-subnet"


az aks create -n $CLUSTERNAME -g $RG \
            --location $LOCATION \
            --network-plugin azure \
            --network-plugin-mode overlay \
            --pod-cidr 192.168.0.0/16 \
            --generate-ssh-keys \
            --node-count 2 \
            --node-vm-size Standard_DS2_v2 \
            --enable-cluster-autoscaler \
            --min-count 2 \
            --max-count 10 \
            --vnet-subnet-id $AKS_SUBNET_ID \

```

Alternatively, you can use Bicep to create the AKS cluster. The repo below contains an example of a Bicep file that creates an AKS cluster with CNI overlay networking:
[AKS Helper](https://github.com/thotheod/aks-deployment-helper/blob/main/private-cni-overlay/deploy.azcli)

## References

- [Azure Kubernetes Service networking overview](https://learn.microsoft.com/en-us/azure/aks/concepts-network)
- [Configure Azure CNI networking in Azure Kubernetes Service (AKS)](https://learn.microsoft.com/en-us/azure/aks/configure-azure-cni)
- [Kubenet](https://learn.microsoft.com/en-us/azure/aks/configure-kubenet)
- [CNI overlay](https://learn.microsoft.com/en-us/azure/aks/azure-cni-overlay?tabs=kubectl)
- [Networking concepts for applications in Azure Kubernetes Service (AKS)](https://learn.microsoft.com/en-us/azure/aks/concepts-network)
- [AKS Helper](https://github.com/thotheod/aks-deployment-helper/blob/main/private-cni-overlay/deploy.azcli)
