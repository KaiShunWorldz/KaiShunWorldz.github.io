# Load Balancing

## NGINX as Load Balancer for Opensim Robust Services ( adapted from NGINX.com )
It is possible to use nginx as a very efficient HTTP load balancer to distribute traffic to several application servers and to improve performance, scalability and reliability of web applications with nginx.Since Robust services talk to each other using HTTP requests, we should be able to use NGINX as a load balancer.

### Load balancing methods
The following load balancing mechanisms (or methods) are supported in nginx:

- *round-robin* — requests to the application servers are distributed in a round-robin fashion,
- *least-connected* — next request is assigned to the server with the least number of active connections,
- *ip-hash* — a hash-function is used to determine what server should be selected for the next request (based on the client’s IP address).

### Default load balancing configuration
The simplest configuration for load balancing a Robust service with nginx may look like the following:
```NGINX
http {
    upstream AssetService {
        server localhost:8005;
        server localhost:8007;
        server localhost:8009;
    }

    server {
        listen 8003;

        location / {
            proxy_pass http://AssetService;
        }
    }
}
```
In the example above, there are 3 instances of the same application running on localhost listening on different ports. When the load balancing method is not specifically configured, it defaults to *round-robin*. All requests are proxied to the server group AssetService, and nginx applies HTTP load balancing to distribute the requests.

### Least connected load balancing
Another load balancing discipline is *least-connected*. Least-connected allows controlling the load on application instances more fairly in a situation when some of the requests take longer to complete.

With the least-connected load balancing, nginx will try not to overload a busy application server with excessive requests, distributing the new requests to a less busy server instead.

Least-connected load balancing in nginx is activated when the least_conn directive is used as part of the server group configuration:
```NGINX
    upstream AssetService {
        least_conn;
        server localhost:8005;
        server localhost:8007;
        server localhost:8009;
    }
```    
   
### Session persistence
Please note that with *round-robin* or *least-connected* load balancing, each subsequent client’s request can be potentially distributed to a different server. There is no guarantee that the same client will be always directed to the same server.

If there is the need to tie a client to a particular application server — in other words, make the client’s session “sticky” or “persistent” in terms of always trying to select a particular server — the ip-hash load balancing mechanism can be used.

With ip-hash, the client’s IP address is used as a hashing key to determine what server in a server group should be selected for the client’s requests. This method ensures that the requests from the same client will always be directed to the same server except when this server is unavailable.

To configure ip-hash load balancing, just add the ip_hash directive to the server (upstream) group configuration:
Least-connected load balancing in nginx is activated when the least_conn directive is used as part of the server group configuration:
```NGINX
upstream AssetService {
    ip_hash;
        server os-asset-usa:8005;
        server os-asset-eur:8005;
        server os-asset-apac:8005;
}
```
Least-connected load balancing in nginx is activated when the least_conn directive is used as part of the server group configuration:

### Health checks
Reverse proxy implementation in nginx includes in-band (or passive) server health checks. If the response from a particular server fails with an error, nginx will mark this server as failed, and will try to avoid selecting this server for subsequent inbound requests for a while.

The max_fails directive sets the number of consecutive unsuccessful attempts to communicate with the server that should happen during fail_timeout. By default, max_fails is set to 1. When it is set to 0, health checks are disabled for this server. The fail_timeout parameter also defines how long the server will be marked as failed. After fail_timeout interval following the server failure, nginx will start to gracefully probe the server with the live client’s requests. If the probes have been successful, the server is marked as a live one.

### Weighted load balancing
It is also possible to influence nginx load balancing algorithms even further by using server weights.

In the examples above, the server weights are not configured which means that all specified servers are treated as equally qualified for a particular load balancing method.

With the round-robin in particular it also means a more or less equal distribution of requests across the servers — provided there are enough requests, and when the requests are processed in a uniform manner and completed fast enough.

When the weight parameter is specified for a server, the weight is accounted as part of the load balancing decision.

```NGINX
    upstream AssetService {
        server os-asset-01 weight=3;
        server os-asset-02;
        server os-asset-03;
    }
```

With this configuration, every 5 new requests will be distributed across the application instances as the following: 3 requests will be directed to os-asset-01, one request will go to os-asset-02, and another one — to os-asset-03.

It is similarly possible to use weights with the least-connected and ip-hash load balancing in the recent versions of nginx.

## Services as Dispatchers ( taken from Rancher )

When a program element needs to make use of the functions abstracted by the service, it makes a request to the service, rather than an individual pod. The service then acts as a dispatcher, assigning a pod to handle the request.

## Dispatching and Load Distribution

If by now, you’re thinking, \“Hey, shouldn’t load balancing happen at the dispatching level?\“— You’re right. A service in Kubernetes is a bit like a heavy-equipment pool, sending functionally identical machines into the field as needed. And as part of the dispatching process, it needs to manage availability and prevent resource bottlenecks.

### Let kube-proxy Do It
The most basic type of load balancing in Kubernetes is actually load distribution, which is easy to implement at the dispatch level. Kubernetes uses two methods of load distribution, both of them operating through a feature called kube-proxy, which manages the virtual IPs used by services.

The default mode for kube-proxy is called iptables, which allows fairly sophisticated rule-based IP management. The native method for load distribution in iptables mode is random selection— an incoming request goes to a randomly chosen pod within a service. The older (and former default) kube-proxy mode is userspace, which uses round-robin load distribution, allocating the next available pod on an IP list, then rotating (or otherwise permuting) the list.

### Genuine Load Balancing: Ingress

As we mentioned above, however, neither of these methods is really load balancing. For true load balancing, the most popular, and in many ways, the most flexible method is Ingress, which operates by means of a controller in a specialized Kubernetes pod. The controller includes an Ingress resource—a set of rules governing traffic—and a daemon which applies those rules. The controller has its own built-in features for load balancing, with some reasonably sophisticated capabilities. You can also include more complex load-balancing rules in an Ingress resource, allowing you to take into account load-balancing features and requirements for specific systems or vendors.

### LoadBalancer as an Alternative

As an alternative to Ingress, you can also use a service of the LoadBalancer type, which uses a cloud service-based, external load balancer. LoadBalancer can only be used with specific cloud service providers, such as AWS, Azure, OpenStack, CloudStack, and Google Compute Engine, and the capabilities of the balancer are provider-dependent. Other load-balancing methods may be available from service providers, as well as third parties.

### In the Balance, It’s Ingress

Currently, however, Ingress is the load-balancing method of choice. Since it is essentially internal to Kubernetes, operating as a pod-based controller, it has relatively unencumbered access to Kubernetes functionality (unlike external load balancers, some of which may not have good access at the pod level). The configurable rules contained in an Ingress resource allow very detailed and highly granular load balancing, which can be customized to suit both the functional requirements of the application and the conditions under which it operates.

## Google Cloud (taken from Google)

Google Kubernetes Engine (GKE) offers integrated support for two types of Cloud Load Balancing for a publicly accessible application:

- When you specify type:*LoadBalancer* in the resource manifest, GKE creates a Service of type LoadBalancer. GKE makes appropriate Google Cloud API calls to create either an external network load balancer or an internal TCP/UDP load balancer. GKE creates an internal TCP/UDP load balancer when you add the cloud.google.com/load-balancer-type: "Internal" annotation; otherwise, GKE creates an external network load balancer.

Although you can use either of these types of load balancers for HTTP(S) traffic, they operate in OSI layers 3/4 and are not aware of HTTP connections or individual HTTP requests and responses. Another important characteristic is that the requests are not proxied to the destination.

```
Note: When GKE creates an external network load balancer, it does not configure any load balancer health check for 
the target pool. (External network load balancers using target pools do not require health checks.) When GKE creates 
an internal TCP/UDP load balancer, it creates a health check for the load balancer's backend service based on the 
readiness probe settings of the workload referenced by the GKE Service. For more information, see to Internal 
TCP/UDP Load Balancing.
```

- When you specify type:*Ingress* in the resource manifest, you instruct GKE to create an Ingress resource. By including annotations and supporting workloads and Services, you can create a custom Ingress controller. Otherwise, GKE makes appropriate Google Cloud API calls to create an external HTTP(S) load balancer. The load balancer's URL map's host rules and path matchers reference one or more backend services, where each backend service corresponds to a GKE Service of type NodePort, as referenced in the Ingress. The backends for each backend service are either instance groups or network endpoint groups (NEGs). NEGs are created when you configure container-native load balancing as part of the configuration for your Ingress. For each backend service, GKE creates a Google Cloud health check, based on the readiness probe settings of the workload referenced by the corresponding GKE Service.

If you are exposing an HTTP(S) service hosted on GKE, HTTP(S) load balancing is the recommended method for load balancing.
