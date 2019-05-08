# Instructions for Setting up an Exchange with CoinXP Public Blockchain

Setting up an Exchange with CoinXP Public Blockchain involves becoming a block producer of CoinXP Public Blockchain.
To enable user engagement and participate in the voting for crosschain digital assets custody, additional ui side server
and client as well as crosschain gateway service have to be set up and deployed. 

Fortunately, the services are all packaged up as kubernetes services at CoinXP and version managed by Helm. Therefore 
requires minimum to no effort to get them running in a real production environment. 

## Why NOT letting CoinXP Set Up EVERYTHING? Centralized v.s. Decentralized
I would like to elaborate on the purpose and motivation of this article before venturing into the detailed technology discussions. One question we frequently got from our partners is why not letting CoinXP set up everything for them. 
It is a valid question and desires some further discussion. 

The root motivation for this setup is to preserve all 
credentials/permissions/architectures under complete control of yourself. The only thing that couples your exchange with CoinXP public blockchain should be the producer account name and block producer endpoints for inter-node communications. CoinXP is a public blockchain infrastructure service. One of its core values is to build and contribue to the decentralized blockchain world. You can certainly ask CoinXP to setup everything for you. In that case, you are placing your fate fully under the control of CoinXP. As a pioneer of the decentralized blockchain world, we strongly suggest you against this setup. 
In comparision, you can think of this set up as setting up your own [Mastodon](https://mastodon.social/about) instance (open source, decentralized version of twitter) instead of opening an account with twitter.

## Become a Block Producer
In order to become a CoinXP Public Blockchain producer, you have to get the approval from the committee. 
This is an offline process. The following instructions assume that you have applied to become a CoinXP block producer and 
received the approval as required. 

### Set Up a Secure and Highly Available Kubernetes Cluster
Since all CoinXP's services are packaged as kubernetes services, you will need a physical cluster to deploy the services. 
Setting up this services should require minimum efforts. We recommend using [AWS (Amazon Web Services)](https://aws.amazon.com/). 
You have two options if you choose AWS to spin up the physical kubernetes cluster,

1. Use the [AWS EKS (Elastic Kubernetes Service)](https://aws.amazon.com/eks/) to setup the cluster. 
This is potentially the most straightforward way to setup a kubernetes cluster. 
It is easy to setup and manage and comes with the AWS supported [eks cli](https://docs.aws.amazon.com/cli/latest/reference/eks/index.html). 
However, we do find a couple of limitations that prevented us from adopting it directly:
    - AWS EKS is still in beta phase. It is only supported in 3 regions, namely Northern Virginia, Ohio, Oregon. 
    - Dependent on the basic features supported by AWS EKS. 

2. Use self managed kubernetes cluster with [kops](https://github.com/kubernetes/kops). 
Kops is a mature tool for deploying and managing kubernetes cluster on AWS. 
There are many online resources for setting up a basic cluster with kops on AWS. 
There is also one tutorial from CoinXP for [Deploying Secure and Highly Available Block Producers on AWS with Kops](https://medium.com/@weijia.che/deploying-secure-and-highly-available-block-producers-on-aws-with-kops-c6956831c048)

### Deploy Block Producer as a Service
The second step of becoming a CoinXP Public Blockchain producer is to deploy the block producer as a service and join the producer schedule. 
Again, we will provide instructions for each phase. 

1. Deploy the producer as a service. Since the block producer is already packaged as a kubernetes service,
 the only thing required is simple deployed similar to install a software on a computer. 
 Only, in this case, you will deploy a kubernetes service on a kubernetes service using helm. 
 The kubernetes service is available as a software package in CoinXP's public repository [coinxp/helm](https://github.com/coinxp/helm). 
    - Install helm client. Instructions can be found here [nstalling Helm](https://helm.sh/docs/using_helm/#installing-helm).
    - Clone the [coinxp/helm](https://github.com/coinxp/helm) repository. 
        ```
        cd helm/charts/nodeosd
        ```
        You will see a list of configuration files, e.g. bpa.yaml - bpg.yaml. Copy the file and create your own configuration file, 
        e.g. _bph_.yaml. The only thing you need to do is to update the name field in the configuration file to e.g. _bph_. 
        Then execute the helm command to deploy service
        ```
        helm install ./chart -f ./chart/bph.yaml --name coinxp-bph
        ```
        Later, if you want to upgrade the version of the service. You can simply execute
        ```
        helm upgrade coinxp-bph ./chart -f ./chart/bph.yaml
        ```
        with a new version number provided in the configuration file. 
        
        **NOTE:** _Deploying the block producer service will automatically create credentials files such as 
        wallet password, producer public key, producer private key in the [AWS Secret Manager Service](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html). 
        You will have to grant the kubernetes pod's role (AWS role) the permission to access [AWS Secret Manager Service](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html).
        In this way, only in your private VPC cluster, and execute as with kubernetes pod's role, the secrets are accessible. 
        This is to ensure the maximum security of cluster secrets. 
        In no place, the secrets are revealed in any manual process. 
        The secrets are created/managed inside the VPC cluster with strict RBAC control (role based access control)._ 
        
 2. Join the CoinXP Public Blockchain as a producer. 
 When first started, the producer node is in standby mode. 
 It will only receive blocks from existing block producers but not producing anything. 
 In order to join as a block producer, as stated in the previous section, you will have to gain approval from the CoinXP 
 public blockchain committee. Once the approval is granted, the committee can elect the producer to start producing blocks. 
 This only thing you need to provide in the step is the name of the producer account, e.g. _bph_ in the example. 
 
    **NOTE:** Again, we recommend using AWS to deployment your physical kubernetes cluster. 
    And CoinXP will pair up your cluster with existing CoinXP VPC clusters with [VPC peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html). 
    Otherwise, additional steps will be required to guarantee maximum performance and highest level of security. 
    
    
## Participate In the Voting of Crosschain Digital Assets Custody
CoinXP's crosschain digital assets custody and block producers are packaged as different services. 
This is to guarantee the highest availability for block producers and decouple the design concerns. 
An article for all the voting take place in CoinXP's crosschain digital assets custody can be found here 
[Building Secure and Highly Available Bridges Between Crypto Islands](https://medium.com/@weijia.che/building-secure-and-highly-available-bridges-between-crypto-islands-a209555a387c).
In a nutshell, every operation in the crosschain digital assets custody will require the majority of the block producers to reach consensus.

   - To participate in the voting process, again, you will need approval from CoinXP public blockchain committee. 
    Once approved, you will be added the a list of approved voters. In this process, all you need to provide to the committee is your block producer account name. 
   
   - To deploy the cross-chain gateway service. A similar process compared to deploying block procuder service  need to be performed.
        ```
        cd helm/charts/gateway
        ```
        You will see a list of configuration files, e.g. bpa.yaml - bpg.yaml. Copy the file and create your own configuration file, 
        e.g. _bph_.yaml. The only thing you need to do is to update the name field in the configuration file to e.g. _bph_. 
        Then execute the helm command to deploy service
        ```
        helm install ./chart -f ./chart/bph.yaml --name coinxp-gateway-bph
        ```
        Later, if you want to upgrade the version of the service. You can simply execute
        ```
        helm upgrade coinxp-gateway-bph ./chart -f ./chart/bph.yaml
        ```
        with a new version number provided in the configuration file. 
        
        
## Bring up UI for Exchange and Personal Bank
CoinXP team also provides a set of packaged services for partners to quickly bring up their own exchanges with personal bank. 
There are 5 services involved with bring up the whole set of service. 

   - Web UI (exchange): This is the service that user interact with. It has the marketplace. User can check prices, orders, buy/sell their crypto assets. 
   - Web Link (exchange): This is the service that powers the WebUI. It connects the Web UI with CoinXP blockchain and the user account system. 
   - Bank UI: This is the personal bank UI. User can use it as a code wallet, whereas, the account system in WebUI as hot wallet. 
   - Bank Server: This is the service that powers Bank UI. It connects Bank UI with Web Link, CoinXP blockchain, and the user account system.
   - Socket: This is the service that powers the market with the latest price for each crypto asset. 
   
To bring up the services are fairly simple. All the services are already packaged and managed by kubernetes and helm. 
    
    cd helm/charts/webui (weblink, bankui, bankserver, or socket)
    helm install ./chart -f ./chart/values.yaml --name coinxp-webui (weblink, bankui, bankserver, or socket)
   Later, if you want to upgrade the version of the service. You can simply execute
   
    helm upgrade coinxp-webui (weblink, bankui, bankserver, or socket) ./chart -f ./chart/bph.yaml
    
**NOTE:** Some environment variables probably need to be updated. Contact CoinXP if you encounter any problem. 
    
