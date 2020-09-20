# istio-boilerplate

This project is a Istio configuration files template that can be used to kickstart an istio based Kubernates system.
This boilerplate supports the most important entities that can be used like gateways, virtual services, databases services, deployments and volumes ... etc.
Also it supports a basic template for implementing authentication interceptor for specific backend services.

## Some Concepts and Definitions

### Service

A service definition is the configuration file that generates a Kubernates service inside the Kubernates cluster, we can define a name for a service, a port which the service works on and much more. Each service we has a deployment that configure how we are fethcing the container instances to deploy the service, how many pods we want to define for each service and much more.
A volumes is defined in case there is a service that wants to store a presistant data.
You can return to Kubernates website for further info [https://kubernetes.io/docs/home/]

### Virtual Service

A Virtual service in Istio defines a set of traffic routing rules to the internal services.

### Service Entries

ServiceEntry enables adding additional entries into Istio’s internal service registry, so that auto-discovered services in the mesh can access/route to these manually specified services.
A service entry describes the properties of a service (DNS name, VIPs, ports, protocols, endpoints). These services could be external to the mesh (e.g., web APIs) or mesh-internal services that are not part of the platform’s service registry (e.g., a set of VMs talking to services in Kubernetes).

### Gateway

Istio gateway is a load balancer that describes a set of ports to be exposed, the type of protocol to use and SNI configuration.

### Authentication Interceptor

Istio supports authentication through Envoy filters which uses JSON Web Token (JWT) as defined by RFC 7519. For more info about authentication policy in Istio you can return to Istio docs from [https://istio.io/docs/reference/config/istio.authentication.v1alpha1/].
Envoy filter can be configured with custom Lua code that may connect to any RESTful end point to authenticate the request and extend the headers of the request going to the service with some info like the user id, company id, roles name ... etc.
Inside auth folder there is a template to configure the Envoy filter for custom authentication.

## How to use these configuration files

After setting up the mesh and installing Istio on it, the configurations in this repository can be applied by using kubectl command like next:

```
kubectl apply gateway.yaml
```

The best order to execute the commands is to define gateway first, auth filter, database services and then virtual services. However apply commnad can be called again to updates the instance whenever you did changes on it.

Whenever you want to list all resources in your mesh you can use the next command:

```
kubectl api-resources -o wide
```

Whenever you want to list the resources under specific resource type, you can use the next command (The command for listing all Virtual Services):

```
kubectl get virtualservice
```

Whenever you want to delete un-needed resoruce you can use the next command (For deleting a virtual service):

```
kubectl delete virtualservice <virtual-service-name>
```

And whenever you need to updated a service you can just apply it again and it will be automatically configured using your last applied configuration.

### Gateway configuration

In this boilerplate I used a simple gateway that only allows inbound packets on port 80, and uses HTTP protocol, and will accepts connections from any hosts (The usage of star in host field)

### Virtual service configuration

In this boilerplate I created two virtual services, one for books service. The virtual service will route any packets coming to gate way on port 80 with /books prefix to our books service. I have put in k8 examples folder some deployment code examples for different types of projects.
The other virtual service I have created is for a React base front-end project accroding to the routes I have used (Suitable for the default build output of a React project).

### Service Entries

I added services entries with virtual services definitions for each service, where I am defining the allowed outside hosts for each virtual service. Please becareful that in case of using a wildcard pattern to select a group of domains then the resolution field should be NONE and not DNS.
One of the ServiceEntry I added to allow communications with Amazon SES. That needed also a VirtualService for that output service to correctly route the packets.

### The authentication interceptor

The authentication interceptor is an Envoy filter that will make a request to the auth service and its in her turn should append the response headers with set_header or unset_header to the filter will add these header or remove them from the request that will continue to the target service (That of course if the auth service didn't returned any status code other than 200 or that response from auth service will return to the caller as it is).

### Database instances

Sometimes if you want to buildup a project that can be installed on-premise too, then it would be better to install databases inside the cluster, here in this boilerplate I created yaml files to configure Mongo, MySQL, Redis databases inside the cluster. You can find the configuration files inside databases folder. And I put some inline comments describing each section.
