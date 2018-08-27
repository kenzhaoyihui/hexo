title: Kubernetes-note
author: kenzhaoyihui
date: 2018-08-27 18:01:29
---
#### K8s Master components:

1. kube-apiserver: 
	* Component on the master that exposes the Kubernetes API. It is the front-end for the Kubernetes control plane
2. etcd
	* Consistent and highly-available key value store used as Kubernetes’ backing store for all cluster data
3. kube-scheduler
	* Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.
4. kube-controller-manager
	* Component on the master that runs controllers .Logically, each controller  is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.

These controllers include:

* Node Controller: Responsible for noticing and responding when nodes go down.
* Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
* Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
* Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.


#### K8s Node components:

1. kubelet
	* An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn’t manage containers which were not created by Kubernetes.
    
2. kube-proxy
	* enables the Kubernetes service abstraction by maintaining network rules on the host and performing connection forwarding

3. Container Runtime
	* The container runtime is the software that is responsible for running containers. Kubernetes supports several runtimes: Docker, rkt, runc and any OCI runtime-spec implementation.
    
    
#### Addons
1. DNS
2. Web UI(Dashboard)
3. Container Resource Monitoring
4. Cluster-level Logging
...


#### Compute, Storgae, and Networking Extensions

1. Certificates
	* easyrsa, openssl, cfssl
    * Distributing Self-Signed CA Certificate
2. Cloud  Providers
	* AWS
	* Azure
	* CloudStack
	* GCE
	* OpenStack
	* OVirt
	* Photon
	* VSphere
3. Cluster Networking
	* Docker model
	* Kubernetes model
    * How to implement the kubernetes networking model
    	* ACI
        * AOS from Apstra
        * Big Cloud Fabric from Big Switch Networks
        * Cilium
        * CNI-Genie from Huawei
        * Contiv
        * Contrail
        * Flannel
        * Google Compute Engine(GCE)
        * Knitter
        * Kube-route
        * L2 networks and linux bridging
        * Multus(a Multi Network plugin)
        * NSX-T
        * Huge Networks VCS(Virtualized Cloud Services)
        * OpenVSwitch
        * OVN(Open Virtual Networking)
        * Project Calico
        * Romana
        * Weave Net from Weaveworks
4. Logging Architecture
	* Basic logging in Kubernetes
	* Logging at the node level -- logrotate
	* Cluster-level logging architecture
5. Kubelet Garbage Collection
	* Image Collection
	* Container Collection
    * Some Garbage Collection features will be replaced by kubelet eviction in the future
6. Federation -- More than one cluster,manage multiple clusters
	* `kubefed` is the recommended way to deploy federated clusters
    * federation API resources:
    	* Cluster
        * ConfigMap
        * DaemonSets
        * Deployment
        * Events
        * Hpa
        * Ingress
        * Jobs
        * Namespaces
        * ReplicaSets
        * Secrets
        * Services
7. Proxies
	* kubectl proxy
    	* runs on a user’s desktop or in a pod
        * proxies from a localhost address to the Kubernetes apiserver
        * client to proxy uses HTTP
        * proxy to apiserver uses HTTPS
        * locates apiserver
        * adds authentication headers
	* apiserver proxy
    	* is a bastion built into the apiserver
        * connects a user outside of the cluster to cluster IPs which otherwise might not be reachable
        * runs in the apiserver processes
        * client to proxy uses HTTPS (or http if apiserver so configured)
        * proxy to target may use HTTP or HTTPS as chosen by proxy using available information
        * can be used to reach a Node, Pod, or Service
        * does load balancing when used to reach a Service
	* kube proxy
    	* runs on each node
        * proxies UDP and TCP
        * does not understand HTTP
        * provides load balancing
        * is just used to reach services
	* a proxy/load-balancer in front of apiserver(s)
    	* existence and implementation varies from cluster to cluster (e.g. nginx)
        * sits between all clients and one or more apiservers
        * acts as load balancer if there are several apiservers.
    * Cloud Load Balancers on external services
    	* are provided by some cloud providers (e.g. AWS ELB, Google Cloud Load Balancer)
        * are created automatically when the Kubernetes service has type LoadBalancer
        * use UDP/TCP only
        * implementation varies by cloud provider.
        

8. Controller manager metrics
	* These metrics include common Go language runtime metrics such as go_routine count and controller specific metrics such as etcd request latencies or Cloudprovider (AWS, GCE, OpenStack) API latencies that can be used to gauge the health of a cluster.
    * http://localhost:10252/metrics

#### Kubernetes Architecture

1. Node -- status
2. Master-Node communication
	* Cluster -> Master
    * Master -> Cluster
3. Cloud Controller Manager
	* Functions of the CCM
    	* Kubernetes controller manager
        	* Node Controller
            * Route Controller
            * Service Controller
            * PersistentVolumeLabels Controller
        * Kubelet
        * Kubernetes API server
    * Plugin mechanism
    * Authorization
    * Vendor Implementations
    * Cluster Administration
    
#### Extending Kubernetes

1. Extending the Kubernetes Cluster
2. Extending the Kubernetes API
	* Extending the Kubernetes API with the aggregation layer
    * Custom Resources
    	* Kubernetes provides two ways to add custom resources to your cluster:
        	* CRDs(CustomResourceDefinitions) are simple and can be created without any programming.
            * API Aggregation requires programming, but allows more control over API behaviors like how data is stored and conversion between API versions.
3. Compute, Storage,and Networking Extensions
	* Network plugins
    	* CNI plugins: adhere to the appc/CNI specification, designed for interoperability.
        	* Support hostPort -- offical `portmap` plugin
            * Support traffic shaping -- offical `bandwidth` plugin
        * Kubenet plugin: implements basic cbr0 using the bridge and host-local CNI plugins
        	* The standard CNI bridge, lo and host-local plugins are required, at minimum version 0.2.0. Kubenet will first search for them in /opt/cni/bin. Specify cni-bin-dir to supply additional search path. The first found match will take effect.
            * Kubelet must be run with the --network-plugin=kubenet argument to enable the plugin
            * Kubelet should also be run with the --non-masquerade-cidr=`clusterCidr` argument to ensure traffic to IPs outside this range will use IP masquerade.
            * The node must be assigned an IP subnet through either the --pod-cidr kubelet command-line option or the --allocate-node-cidrs=true --cluster-cidr=`cidr` controller-manager command-line options.

	* Device plugins
4. Service Catalog
	* Service Catalog is an extension API that enables applications running in Kubernetes clusters to easily use external managed software offerings, such as a datastore service offered by a cloud provider.
    
    
#### Containers
1. Images
	* Updating Images
    * Using a Private Registry
    	* Using Google Container Registry(GCR)
        * Using AWS EC2 Container Registry(ECR)
        * Using Azure Container Registry(ACR)
        * Configuring Nodes to Authenticate to a Private Registry
        * Pre-pulling Images
        * Specifying ImagePullSecrets on a Pod
    * Use Cases
    	* Cluster running only non-proprietary(e.g. open-source) images.No need to hide images.(Use public images on the Docker hub)
        * Cluster running some proprietary images which should be hidden to those outside the company, but visible to all cluster users.
        * Cluster with a proprietary images, a few of which require stricter access control.
        * A multi-tenant cluster where each tenant needs own private registry.
        
2. Container Environment Variables
	* Container environment
    	* A filesystem, which is a combination of an image and one or more volumes.
        * Information about the Container itself.
        * Information about other objects in the cluster
        
3. Container Lifecycle Hooks
	* This page describes how kubelet managed Containers can use the Container lifecycle hook framework to run code triggered by events during their management lifecycle.
    * Container hooks
    	* PostStart
    	* PreStop
    * Hook handler implementations
    	* Exec - Executes a specific command, such as pre-stop.sh, inside the cgroups and namespaces of the Container. Resources consumed by the command are counted against the Container
        * HTTP - Executes an HTTP request against a specific endpoint on the Container.
        
        
#### Workloads
1. Pods
	* Pods
	* Pod Lifecycle
    	* Pod phase
    		* Pending
        	* Running
        	* Succeeded
        	* Failed
        	* Unknown
        * Pod conditions
        	* lastProbeTime
            * lastTransitionTime
            * message
            * reason
            * status
            * type
            	* PodScheduled
                * Ready
                * Initialized
                * Unschedulable
                * ContainerReady
        * Container probes
        	* ExecAction: Executes a specified command inside the Container. The diagnostic is considered successful if the command exits with a status code of 0.
            * TCPSocketAction: Performs a TCP check against the Container’s IP address on a specified port. The diagnostic is considered successful if the port is open.
            * HTTPGetAction: Performs an HTTP Get request against the Container’s IP address on a specified port and path. The diagnostic is considered successful if the response has a status code greater than or equal to 200 and less than 400.
        * Restart policy: -- restartPolicy
        	* Always
            * OnFailure
            * Never
        
	* Init Containers
    	* A Pod can have multiple Containers running apps within it, but it can also have one or more Init Containers, which are run before the app Containers are started.
        * They always run to completion.
        * Each one must complete successfully before the next one is started.
	* Pod Preset
    	* This page provides an overview of PodPresets, which are objects for injecting certain information into pods at creation time. The information can include secrets, volumes, volume mounts, and environment variables.
	* Disruptions
    	* This guide is for application owners who want to build highly available applications, and thus need to understand what types of Disruptions can happen to Pods.
        * Voluntary and Involuntary Disruptions
        * Dealing with Disruptions
        	* Ensure your pod requests the resources it needs.
            * Replicate your application if you need higher availability. (Learn about running replicated stateless and stateful applications.)
            * For even higher availability when running replicated applications, spread applications across racks (using anti-affinity) or across zones (if using a multi-zone cluster.)
        * Separating Cluster Owner and Application Owner Roles
        * How to perform Disruptive Actions on your Cluster
        	* Accept downtime during the upgrade.
            * Fail over to another complete replica cluster.
            * Write disruption tolerant applications and use PDBs(PodDisruptionBudgets)
        

2. Controllers
	* ReplicaSet
    	* ReplicaSet is the next-generation Replication Controller. The only difference between a ReplicaSet and a Replication Controller right now is the selector support. ReplicaSet supports the new set-based selector requirements as described in the labels user guide whereas a Replication Controller only supports equality-based selector requirements.
        * Alternatives to ReplicaSet:
        	* Deployment(Recommended) -- Deployment is a higher-level API object that updates its underlying ReplicaSets and their Pods in a similar fashion as kubectl rolling-update. Deployments are recommended if you want this rolling update functionality, because unlike kubectl rolling-update, they are declarative, server-side, and have additional features. For more information on running a stateless application using a Deployment, please read Run a Stateless Application Using a Deployment.
            
            * Bare Pods -- Unlike the case where a user directly created pods, a ReplicaSet replaces pods that are deleted or terminated for any reason, such as in the case of node failure or disruptive node maintenance, such as a kernel upgrade. For this reason, we recommend that you use a ReplicaSet even if your application requires only a single pod. Think of it similarly to a process supervisor, only it supervises multiple pods across multiple nodes instead of individual processes on a single node. A ReplicaSet delegates local container restarts to some agent on the node (for example, Kubelet or Docker).
            
            * Job -- Use a Job instead of a ReplicaSet for pods that are expected to terminate on their own (that is, batch jobs).
            
            * DaemonSet -- Use a DaemonSet instead of a ReplicaSet for pods that provide a machine-level function, such as machine monitoring or machine logging. These pods have a lifetime that is tied to a machine lifetime: the pod needs to be running on the machine before other pods start, and are safe to terminate when the machine is otherwise ready to be rebooted/shutdown.
            
	* ReplicationController
    	* A ReplicationController ensures that a specified number of pod replicas are running at any one time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available.
        * Common usage pattern
        	* Rescheduling
            * Scaling
            * Rolling updates
            * Multiple release tracks
            * Using ReplicationControllers with Services
        * Alternatives to ReplicationController
        	* ReplicaSet
            * Deployment(Recommended)
            * Bare Pods
            * Job
            * DaemonSet
	* Deployments
    	* A Deployment controller provides declarative updates for Pods and ReplicaSets.You describe a desired state in a Deployment object, and the Deployment controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.
        * Use Case
        	* Create a Deployment to rollout a ReplicaSet
            * Declare the new state of the Pods
            * Rollback to an earlier Deployment revision
            * Scale up the Deployment to facilitate more load
            * Pause the Deployment
            * Use the status of the Deployment
            * Clean up older ReplicaSets
        * Alternative to Deployments
        	* kubectl rolling update
	* StatefulSets
    	* StatefulSet is the workload API object used to manage stateful applications.
        * Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods; These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.
        * Using StatefulSets:
        	* Stable, unique network identifiers.
            * Stable, persistent storage.
            * Ordered, graceful deployment and scaling.
            * Ordered, graceful deletion and termination.
            * Ordered, automated rolling updates.
        * Pod Identity
        	* Ordinal Index
            * Stable Network ID
            * Stable Storage
            * Pod Name Label
        * Pod Management Policies
        	* OrderedReady Pod Management
            * Parallel Pod Management
        * Update Strategies
        	* On Delete
            * Rolling Updates
	* DaemonSet
    	* A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.
        * Typical uses of a DaemonSet:
        	* running a cluster storage daemon, such as glusterd, ceph, on each node.
            * running a logs collection daemon on every node, such as fluentd or logstash.
            * running a node monitoring daemon on every node, such as Prometheus Node Exporter, collectd, Datadog agent, New Relic agent, or Ganglia gmond.
        * Communicating with Daemon Pods
        	* Push: Pods in the DaemonSet are configured to send updates to another service, such as a stats database. They do not have clients.
            * NodeIP and Known Port: Pods in the DaemonSet can use a hostPort, so that the pods are reachable via the node IPs. Clients know the list of node IPs somehow, and know the port by convention.
            * DNS: Create a headless service with the same pod selector, and then discover DaemonSets using the endpoints resource or retrieve multiple A records from DNS
            * Service: Create a service with the same Pod selector, and use the service to reach a daemon on a random node. (No way to reach specific node.)
        * Alternatives to DaemonSet
        	* Init Scripts
            * Bare Pods
            * Static Pods
            * Deployments
	* Garbage Collection
    	* The role of the Kubernetes garbage collector is to delete certain objects that once had an owner, but no longer have an owner.
        * Controlling how the garbage collector deletes dependents
        	* Foreground cascading deletion
            * Background cascading deletion
            * Setting the cascading deletion policy
            * Additional note on Deployments
	* Jobs - Run to Completion
    	* A job creates one or more pods and ensures that a specified number of them successfully terminate. As pods successfully complete, the job tracks the successful completions. When a specified number of successful completions is reached, the job itself is complete. Deleting a Job will cleanup the pods it created.A simple case is to create one Job object in order to reliably run one Pod to completion. The Job object will start a new Pod if the first pod fails or is deleted (for example due to a node hardware failure or a node reboot).A Job can also be used to run multiple pods in parallel.
        * Handing Pod and Container Failures
        	* Pod Backoff failure policy
        * Job Termination and Cleanup
        * Job Patterns
        	*  One Job object for each work item, vs. a single Job object for all work items. The latter is better for large numbers of work items. The former creates some overhead for the user and for the system to manage large numbers of Job objects.
            * Number of pods created equals number of work items, vs. each pod can process multiple work items. The former typically requires less modification to existing code and containers. The latter is better for large numbers of work items, for similar reasons to the previous bullet.
            * Several approaches use a work queue. This requires running a queue service, and modifications to the existing program or container to make it use the work queue. Other approaches are easier to adapt to an existing containerised application.
        * Advanced Usage
        	* Specifying your own pod selector
        * Alternatives
        	* Bare Pods
            * Replication Controller
            * Single Job starts Controller Pod
	* CronJob


#### Configuration
1. Configuration Best Practices
2. Managing Compute Resources for Containers
3. Assigning Pods to Nodes
4. Taints and Tolerations
	* Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints. Tolerations are applied to pods, and allow (but do not require) the pods to schedule onto nodes with matching taints.
5. Secrets
	* Objects of type secret are intended to hold sensitive information, such as passwords, OAuth tokens, and ssh keys. Putting this information in a secret is safer and more flexible than putting it verbatim in a pod definition or in a docker image. 
    * Usage of Secrets
    	* as files in a volume mounted on one or more of its containers, 
        * or used by kubelet when pulling images for the pod.
    * Using Secrets
    	* Using Secrets as Files from a Pod
        	* Projection of secret keys to specific paths
            * Secret files permissions
            * Consuming Secret Values from Volumes
            * Mounted Secrets are updated automatically
        * Using Secrets as Environment Variables
        	* Consuming Secret Values from Environment Variables
        * Using imagePullSecrets
        	* Manually specifying an imagePullSecret
    * Use cases
    	* Pod with ssh keys
        * Pods with prod/test credentials
        * Dotfiles in serect volume
        * Secret visible to one container in pod
    * Best practices
    	* Clients that use the secrets API
    * Security Properties
    	* Protections
        * Risks
        
6. Organizing Cluster Access Using kubeconfig Files
	* Use kubeconfig files to organize information about clusters, users, namespaces, and authentication mechanisms. The kubectl command-line tool uses kubeconfig files to find the information it needs to choose a cluster and communicate with the API server of a cluster.
7. Pod Priority and Preemption
	* Pods can have priority. Priority indicates the importance of a Pod relative to other Pods. If a Pod cannot be scheduled, the scheduler tries to preempt (evict) lower priority Pods to make scheduling of the pending Pod possible.
    * How to use priority and preemption
    	* Add one or more PriorityClasses
        * Create Pods withpriorityClassName set to one of the added PriorityClasses. Of course you do not need to create the Pods directly; normally you would add priorityClassName to the Pod template of a collection object like a Deployment.


#### Services, Load Balancing,and Networking
1. Services
	* Services with selectors
    * Services without selectors, for example:
    	* You want to have an external database cluster in production, but in test you use your own databases.
        * You want to point your service to a service in another Namespace or on another cluster.
        * You are migrating your workload to Kubernetes and some of your backends run outside of Kubernetes.
    * Virtual IPs and service proxies
    	* Proxy-mode:
        	* userspace
            * iptables
            * ipvs -- based on netfilter hook function,but uses hash table as the underlying data structure and works in the kernel space.That means ipvs redirects traffic much faster, and has much better performance when syncing proxy rules. Furthermore, ipvs provides more options for load balancing algorithm, such as: round-robin, least connection, destination hashing, source hashing, shortest expected delay, never queue
        * Multi-Port Services
        * Choosing your own IP address
        * Discovering services
        	* Environment variables
            * DNS
        * Headless services
        	* Sometimes you don’t need or want load-balancing and a single service IP. In this case, you can create “headless” services by specifying "None" for the cluster IP (.spec.clusterIP).
        * Publishing services - service types
        	* Types:
            	* ClusterIP -- Exposes the service on a cluster-internal IP. Choosing this value makes the service only reachable from within the cluster. This is the default ServiceType
                * NodePort -- Exposes the service on each Node’s IP at a static port (the NodePort). A ClusterIP service, to which the NodePort service will route, is automatically created. You’ll be able to contact the NodePort service, from outside the cluster, by requesting NodeIP:NodePort
                * LoadBalancer -- Exposes the service externally using a cloud provider’s load balancer. NodePort and ClusterIP services, to which the external load balancer will route, are automatically created.
                * ExternalName -- Maps the service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value. No proxying of any kind is set up. This requires version 1.7 or higher of kube-dns
2. DNS for Services and Pods
	* Kubernetes DNS schedules a DNS Pod and Service on the cluster, and configures the kubelets to tell individual containers to use the DNS Service’s IP to resolve DNS names.
    * Support DNS mode
    	* Service
        	* A records
            * SRV records
        * Pods
        	* A records
            * Pod’s hostname and subdomain fields
            * Pod’s DNS Policy -- dnsPolicy
            	* Default
                * ClusterFirst -- If dnsPolicy is not explicitly specified, then “ClusterFirst” is used.
                * ClusterFirstWithHostNet
                * None
            * Pod’s DNS Config -- dnsPolicy is set to None, then it can be specified
            	* nameservers
                * searches
                * options
3. Connecting Application with Services
4. Ingress
	* An API object that manages external access to the services in a cluster, typically HTTP.
    * Ingress can provide load balancing, SSL termination and name-based virtual hosting.
    * Ingress controllers
    	* Kubernetes currently supports and maintains GCE and nginx controllers.
        * F5 Networks provides support and maintenance for the F5 BIG-IP Controller for Kubernetes.
        * Kong offers community or commercial support and maintenance for the Kong Ingress Controller for Kubernetes
        * Traefik is a fully featured ingress controller (Let’s Encrypt, secrets, http2, websocket…), and it also comes with commercial support by Containous
        * NGINX, Inc. offers support and maintenance for the NGINX Ingress Controller for Kubernetes
    * Type of Ingress
    	* Single Service Ingress
        * Simple fanout
        * Name based virtual hosting
        * TLS
        * Loadbalancing
5. Network Policies
	* A network policy is a specification of how groups of pods are allowed to communicate with each other and other network endpoints.
    * NetworkPolicy resources use labels to select pods and define rules which specify what traffic is allowed to the selected pods.
    * Default policies
    	* Default deny all ingress traffic
        * Default allow all ingress traffic
        * Default deny all egress traffic
        * Default allow all egress traffic
        * Default deny all ingress and all egress traffic
6. Adding entries to Pod /etc/hosts with HostAliases


#### Storage
1. Volumes
	* On-disk files in a Container are ephemeral, which presents some problems for non-trivial applications when running in Containers. First, when a Container crashes, kubelet will restart it, but the files will be lost - the Container starts with a clean state. Second, when running Containers together in a Pod it is often necessary to share files between those Containers. The Kubernetes Volume abstraction solves both of these problems.
    * Kubernetes supports types of volumes:
    	* awsElasticBlockStore
        * azureDisk
        * azureFile
        * cephfs
        * configMap
        * downwardAPI
        * emptyDir
        * fc
        * flocker
        * gcePersistentDisk
        * gitRepo (deprecated)
        * glusterfs
        * hostPath
        * iscsi
        * local
        * nfs
        * persistentVolumeClaim
        * projected
        * portworxVolume
        * quobyte
        * rdb
        * scaleIO
        * secret
        * storageos
        * vsphereVolume
2. Persistent Volumes
	* Lifecycle of a volume and claim
    	* Provisoning
        	* Static
            * Dynamic
        * Binding
        * Using
        * Storage Object in Use Protection
        * Reclaiming
        	* Retain
            * Delete
            * Recycle
        * Expanding Persistent Volumes Claims
        	* Resizing a volume containing a file system
            * Resizing an in-use PersistentVolumeClaim
    * Types of Persisient Volumes
    	* GCEPersistentDisk
        * AWSElasticBlockStore
        * AzureFile
        * AzureDisk
        * FC
        * Flexvolume
        * Flocker
        * NFS
        * iSCSI
        * RBD (Ceph Block Device)
        * CephFS
        * Cinder (OpenStack block storage)
        * Glusterfs
        * VsphereVolume
        * Quobyte Volumes
        * HostPath (Single node testing only – local storage is not supported in any way and WILL NOT WORK in a multi-node cluster)
        * Portworx Volumes
        * ScaleIO Volumes
        * StorageOS
3. Storage Classes
4. Dynamic Volume Provisioning
5. Node-specific Volume Limits


#### Policies
1. Pesource Quotas
2. Pod Security Policies