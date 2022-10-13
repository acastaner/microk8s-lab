==This file is still a work in progress==

# microk8s-lab
A repo with some files needed for microk8s/Kubernetes lab hand-ons.

# Prerequisites

+ This lab requires Ubuntu Server 20.04. Do not use 22.04 as some of packages are not officially available for that version.
+ The lab will be a single node, single cluster environment.
+ To make this easy we will use [microk8s](https://microk8s.io/). This allows the cluster to be in a Virtual Environment, unlike **minikube**.
+ Basic understanding of Linux is expected (command line, text editor, sudo, etc.).

## Ubuntu Server ##

Install [Ubuntu Server 20.04](https://ubuntu.com/download/server) in a Virtual Machine (the Hypervisor type doesn't matter). Make sure to enable OpenSSH Server during install, otherwise go for a minimal installation.

I suggest to take a snapshot of the VM at this step.

## MicroK8s ##

Once the server is set up, install MicroK8s:

`sudo snap install microk8s --classic`

Enable some addons. While only the `ingress` addon is required, it doesn't hurt to enable the other ones as well in case you need to expand on that lab later on. This is up to you.

`microk8s enable dns dashboard storage ingress`

Ingress is absolutely mandatory as it'll allow us to route traffic from outside of the cluster (eg: from external clients, such as yourself or a traffic generator).

With `microk8s status` you can see the list of available addons and the ones currently enabled.

### List of the most important addons ###
* dns: Deploy DNS. This addon may be required by others, thus we recommend you always enable it.
dashboard: Deploy kubernetes dashboard.
storage: Create a default storage class. This storage class makes use of the hostpath-provisioner pointing to a directory on the host.
* ingress: Create an ingress controller.
* gpu: Expose GPU(s) to MicroK8s by enabling the nvidia-docker runtime and nvidia-device-plugin-daemonset. Requires NVIDIA drivers to be already installed on the host system.
* istio: Deploy the core Istio services. You can use the microk8s istioctl command to manage your deployments.
* registry: Deploy a docker private registry and expose it on localhost:32000. The storage addon will be enabled as part of this addon.

**Once this is done, I suggest to take a snapshot of your VM so you can easily roll-back in case anything breaks horribly.**

## Deploying an app, service and ingress ##

We need an application that'll run in a _container_ (that's a pod). We can't access that application directly, we need to go through a _service_ (sometimes shortened to "svc"). To access that service from the outside of cluster, we need an _ingress_.

The templates for each of these components are the *.yaml files stored under the _/echo_server_ directory of this repository.

The application we'll be running is an _echo server_ - it returns the details of the requests. To deploy it, use this command:

`microk8s kubectl apply -f echo-deployment.yaml`

To access this application we need a service. The command is the same but points to the service template:

`microk8s kubectlk apply -f echo-service.yaml`

**Important note:** Note that the service `type` is set to `NodePort`. This is required in order to access the service from outside of cluster.

And finally, create the ingress:

`microk8s kubectl apply -f echo-ingress.yaml`



