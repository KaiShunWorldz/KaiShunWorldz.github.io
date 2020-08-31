Services as Dispatchers

That’s how Kubernetes handles containers and pods at the management level. But as we mentioned above, it also abstracts functionally related/identical pods into services, and it is at the service level that external clients and other elements of the application interact with pods. Services have IP addresses (used internally by Kubernetes) which are relatively stable. When a program element needs to make use of the functions abstracted by the service, it makes a request to the service, rather than an individual pod. The service then acts as a dispatcher, assigning a pod to handle the request.

Dispatching and Load Distribution

If by now, you’re thinking, \“Hey, shouldn’t load balancing happen at the dispatching level?\“— You’re right. A service in Kubernetes is a bit like a heavy-equipment pool, sending functionally identical machines into the field as needed. And as part of the dispatching process, it needs to manage availability and prevent resource bottlenecks.

Let kube-proxy Do It
The most basic type of load balancing in Kubernetes is actually load distribution, which is easy to implement at the dispatch level. Kubernetes uses two methods of load distribution, both of them operating through a feature called kube-proxy, which manages the virtual IPs used by services.

The default mode for kube-proxy is called iptables, which allows fairly sophisticated rule-based IP management. The native method for load distribution in iptables mode is random selection— an incoming request goes to a randomly chosen pod within a service. The older (and former default) kube-proxy mode is userspace, which uses round-robin load distribution, allocating the next available pod on an IP list, then rotating (or otherwise permuting) the list.

Genuine Load Balancing: Ingress

As we mentioned above, however, neither of these methods is really load balancing. For true load balancing, the most popular, and in many ways, the most flexible method is Ingress, which operates by means of a controller in a specialized Kubernetes pod. The controller includes an Ingress resource—a set of rules governing traffic—and a daemon which applies those rules. The controller has its own built-in features for load balancing, with some reasonably sophisticated capabilities. You can also include more complex load-balancing rules in an Ingress resource, allowing you to take into account load-balancing features and requirements for specific systems or vendors.

LoadBalancer as an Alternative

As an alternative to Ingress, you can also use a service of the LoadBalancer type, which uses a cloud service-based, external load balancer. LoadBalancer can only be used with specific cloud service providers, such as AWS, Azure, OpenStack, CloudStack, and Google Compute Engine, and the capabilities of the balancer are provider-dependent. Other load-balancing methods may be available from service providers, as well as third parties.

In the Balance, It’s Ingress

Currently, however, Ingress is the load-balancing method of choice. Since it is essentially internal to Kubernetes, operating as a pod-based controller, it has relatively unencumbered access to Kubernetes functionality (unlike external load balancers, some of which may not have good access at the pod level). The configurable rules contained in an Ingress resource allow very detailed and highly granular load balancing, which can be customized to suit both the functional requirements of the application and the conditions under which it operates.
