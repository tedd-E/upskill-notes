# Networking with Kubernetes

## OSI Model

OSI (open systems interconnection) model breaks network into 7 layers. Each layer uses data from the layer below it to create its own **PDU** (protocol data unit), then sends it to the next layer until the communication is received.
1. Application: the top layer of the OSI model, and is what the end user sees. Responsible for displaying the data sent over the network.
   1. PDU: data
   2. Examples: HTTP, DNS, SSH, SMTP
2. Presentation: translates data between application layer and lower layers. Responsible for formatting and encoding/decoding data.
   1. PDU: data
   2. Examples: SSL/TLS, data format conversions
3. Session: manages and controls the communication sessions between a local and remote application. Establishes procedures for checkpointing, suspending, and terminating a session.
   1. PDU: data
   2. Examples: NetBIOS, PPTP (point-to-point tunneling protocol) 
4. Transport: manages end-to-end communication services for applications, and ensure data is delivered properly
   1. PDU: data segments (TCP) or datagrams (self-contained data packet) (UDP)
   2. Examples: TCP (transmission control protocol), UDP (user datagram protocol)
5. Network: manages data transfer between devices on different networks, and handles routing, forwarding, addressing, and packet fragmentation.
   1. PDU: packet. Packets attach IP addresses and routing information to data so that it can travel across networks.
   2. Examples: IP & IP addresses, routers
6. Data link: establishes and manages a link between two hosts on the same network.
   1. PDU: Data link frame. Frames contain both data and addressing information (such as MAC addresses), and don't cross local network boundaries.
   2. Examples: ethernet, wifi, MAC addresses
7. Physical: hardware level communication. Defines elements such as data transmission rates, voltage levels, etc.
   1. PDU: bits
   2. Examples: ethernet cables, fiber optics, radio frequencies


## Container Networking 
Processes are isolated using network namespaces, by giving the process and its children a virtual IP stack (including interfaces, routes, IP tables, etc). Running a container is essentially putting a set of processes within a private network namespace. 

Kubernetes is based around pod networking. Each pod is a set of containers that share the same network namespace. All pods within a Kubernetes cluster should be able to communicate with each other. This can be restricted with additional networking communication policies, but without interference, any pod should be able to talk to any other pod in a cluster.

### 3 Core Kubernetes Plugins
Kubernetes relies on 3 core plugins to extend its functionality, and integrate with different environments. 

1. CRI: Container runtime interface (Docker, containerd, etc)
2. CNI: Container networking interface. This is a specification that allows Kubernetes systems to integrate with SDN (software defined networking) solutions.
   1. CNI Plugins provide pod network wiring, pod IP addresses, network policy implementation, etc.
   2. CNI plugins manage CIDR block allcoations.
   2. One solution is Cilium-  a CNI plugin that provides pod networking features.
3. CSI: Container storage interface

## Cluster Network Architectures
### Isolated 
In an isolated cluster network, nodes and the Kubernetes API server are routable (ie they can receive external traffic), but pods are not. This allows multiple clusters to use the same IP address space, because each cluster is not routable from the broader network. Any clusters that need to receive external traffic will have load balancers and proxies.

### Flat
In a flat network, all pods have an IP address that are routable from the broader network (ie they can receive external traffic), regardless of which node the communication comes from. A drawback of this network architecture is that it requires a large continuous IP address space, because the pods cannot have any repeating addresses. It is achievable with a private subnet, but harder and more expensive with public IPs. 

### Island (most common)
Island networks are a combination of isolated and flat networks. Nodes have connectivity with the broader network, but pods don't. Pods send and receive traffic through node proxies, such as `iptables`, so that outbound packets appear to be from the node, rather than the pod. This also masks the pod's IP address from outside the cluster.


### Services
Kubernetes Services have an internal IP address, called a ClusterIP. ClusterIPs are used for internal communications with in the Kubernetes cluster only, are the default k8s service type, and provides load balancing by distributing communication requests across different pods. Cluster IPs are implemented by `kube-proxy` or by other CNI plugins. 

Kubernetes `Services` have kind `Ports` to forward, and kind `EndPoints` to forward to. Selectors identify pods to generate endpoints from. Endpoints can be generated automatically for services, or created manually through manifests.


| Types of K8s Services | Description                                                                |
|-----------------------|----------------------------------------------------------------------------|
| Headless              | A special case of ClusterIP. Supports name resolution, but not forwarding. |
| ClusterIP             | For load balanced internal service communications in a cluster.            |
| NodePort              | For external services across a universal port.                             |
| LoadBalancer          | Uses a plugin to enable an external load balancer                          |
| ExternalName          | Aliases this service to a specified externalName                           |


### Controllers
Kubernetes controllers are software that watches and synchronizes resources in Kubernetes. Kubernetes has multiple controllers, each responsible for a specific resource type or operation.

`kube-controller-manager` includes multiple controllers that manage the network stack. Admins set the cluster CIDR here. 

`kubelet` is a single binary that runs on every worker node in a cluster. It is responsible for managing any pods scheduled to the node, and tracking the status of the nodes and pods. The networking functionality in the kubelet comes from API interactions with a CNI plugin on the node.


## Conclusion 


### Resources
* [CNCF - Kubernetes Networking 101](https://www.youtube.com/watch?v=cUGXu2tiZMc)
* [O'Reilly - Networking and Kubernetes](https://learning.oreilly.com/library/view/networking-and-kubernetes/9781492081647/preface01.html)

[//]: # (* [CNCF - Kubernetes Networking Deep Dive]&#40;https://www.youtube.com/watch?v=tq9ng_Nz9j8&#41;)

