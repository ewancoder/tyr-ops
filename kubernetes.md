# Kubernetes

> `kubeadm reset` - completely erases kubernetes node :)

## Installation on a node

- `kubeadm` - official kubernetes bootstrap tool
  - `kubeadm init` - sets up control plane
  - `kubeadm join` - connects another machine
- `kubelet` - agent that runs on each node, like a docker daemon
  - requires `iptables-nft` instead of `iptables`, newer iptables, fine to replace
- `kubectl` - CLI, equivalent of `docker`

Enable kubelet:

`systemctl enable --now kubelet` - will crashloop until kubeadm has initialized

### Disable UFW

By default, ufw denies any traffic that is forwarded from one interface to another, within the same machine.
We need to allow it to do that for kubernetes networking (CNI) to work.
However, using `ufw default allow routed` actually is very bad for a server, because it turns it into a router: anything that can reach it - can forward traffic through it.

- `ufw default allow routed` - easy fix for home machine, bad for server (read above).

Specific rules (better):

```
ufw allow in on cni0 from 10.244.0.0/16
ufw allow in on flannel.1 from 10.244.0.0/16
ufw route allow in on flannel.1 out on flannel.1
```

Maybe this is also needed but not necessarily:

```
ufw route allow in on cni0
ufw route allow in on flannel.1
ufw route allow out on cni0
ufw route allow out on flannel.1
```

This allows forwarding specifically for Flannel. `cni0` is a linux bridge (like docker0) that Flannel creates. All pods have `cni0` as their bridge. This is how pods talk to each other on the same node.

`flannel.1` - VXLAN interface that Flannel creates for cross-node communication.

To check them out:

`ip link`

### Swap support

By default, the kubelet will not start on a Linux node that has swap enabled.
Kubernetes historically was designed to run on a machine that has all the memory **physical**, to properly allocate memory between pods.
Kubelet needs to be configured to be used with swap, or disable swap on the node.

### Bootstrapping

First - reboot, or load necessary modules:

`sudo modprobe br_netfilter`

Then:

`kubeadm init` --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=<server-ip>

- `/16` means `10.244.0.0` to `10.244.255.255` - 16 bits are fixed. That's 65k addresses for pods.

> CIDR - Classless Inter-Domain Routing

> The server is created on the default interface (`ip route`) and is using its IP address. Consider configuring the default interface to be of a Wireguard mesh (if we need it) same as Swarm, before initializing the node.

> We need to set the CIDR to this exact IP address, because we'll be installing **Flannel** CNI and that's what it expects. If we use another CNI - we need to specify a different CIDR. If we want a different IP address - we would need to modify Flannel configs.

Then follow instructions to:

- Create config files for the user
- Install a pod network to the cluster (CNI - container network interface, or "network fabric")

### Fresh system pods

```
NAMESPACE     NAME                           READY   STATUS    RESTARTS   AGE
kube-system   coredns-7d764666f9-v4v9w       0/1     Pending   0          7m57s
kube-system   coredns-7d764666f9-wqvsk       0/1     Pending   0          7m57s
kube-system   etcd-odin                      1/1     Running   0          8m3s
kube-system   kube-apiserver-odin            1/1     Running   0          8m3s
kube-system   kube-controller-manager-odin   1/1     Running   0          8m3s
kube-system   kube-proxy-z6rvz               1/1     Running   0          7m57s
kube-system   kube-scheduler-odin            1/1     Running   0          8m3s
```

CoreDNS pods are still **pending** - because Network is not installed. Simplest network to install - **Flannel** - pure overlay-based network without extra features. But other CNIs can encrypt traffic between pods (redundant with my Wireguard setup).

Flannel is written in Go, single `flanneld` binary on each node, leasing addresses.

`kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml`

It will apply in all pods, and coredns will start shortly.

> WIP: Describe each.

### How to check logs

`kubectl logs -n <namespace> <pod-name>`

### How to force restart

Delete the pod:

`kubectl delete pod -n <namespace> <pod-name>`

### Making the "control" node schedule work

By default, kubeadm marks the control plane node as "don't schedule regular workloads here". We need to make it a "worker". Taints are like labels in Swarm.

Check current taints:

`kubectl describe node odin | grep Taints`

Remove control-plane taint (that by default says NoSchedule):

`kubectl taint nodes odin node-role.kubernetes.io/control-plane-`

## Using kubernetes

- `kubectl create deployment nginx --image=nginx` - creates deployment
  - This operation is instantaneous, everything happens behind the scenes
- `kubectl exec -it <pod-name> -- bash` - execute bash in pod

Pod naming convention:

<app-name>-<replicaset-hash>-<pod-hash>

### Exposing the pod

`kubectl expose deployment httpbin --port=80 --target-port=8080`

1. Creates a Service resource of type ClusterIP (default) called httpbin
2. Gets a stable internal DNS name: `httpbin.default.svc.cluster.local` (or just `httpbin` within the same namespace)
2.1. And gets a virtual cluster IP
3. 80 - what the service listens to, 8080 - what the image/pod actually listen on
  - so, if http server inside the container listens to 8080, and we want to actually access it from our machine by CLUSTER-IP:80 - this is the way to go
4. Round-robin load balancing to 3 pods on 8080
5. It's a shorthand for creating the yaml:

> Istio sidecar doesn't know how many connections services have from other services, only from the current one - locally tracked connections. Disadvantage compared to Caddy or Istio Gateway, but reducing single point of failures.

```
apiVersion: v1
kind: Service
metadata:
  name: httpbin
spec:
  selector:
    app: httpbin
  ports:
    - port: 80
      targetPort: 8080
```

`kubectl get services -A` - get all services

Now, you can go to pod's CLUSTER-IP:80, and it will open the page (round-robin).

### Load balancing

Kubernetes always uses round-robin, for least-conn we need a service mesh (or ingress controllers etc).

The alternative (like `dnsrr` in Swarm) is `clusterIP: None`. This is HEADLESS service. Returns list of IP addresses of all pods.

```
apiVersion: v1
kind: Service
metadata:
  name: httpbin
spec:
  clusterIP: None
  selector:
    app: httpbin
  ports:
    - port: 8080
```

### Playing with the cluster

```
kubectl create deployment httpbin --image=mccutchen/go-httpbin --replicas=3
kubectl expose deployment httpbin --port=80 --target-port=8080
```

Checking the logs of a single pod:

`kubectl logs podname`

Checking a logs of an app:

`kubectl logs app=httpbin -f --prefix`

- `--prefix` - shows which logs belong to which pod
- `-f` - follows live
- `-l` - selector, filters by label
- `--all-containers` - include all containers, we have only one so it's fine (no service mesh installed)

Debugging stuff!! same as docker debug but free:

`kubectl debug -it full-podname --image=busybox --target=containername`

Where containername is the name of the target container, for example `go-httpbin` for httpbin, even if you named the app lolbin.

!! Without a Service (exposed deployment), you cannot reach the pod even from another pod. It is only reachable after you expose it in some way.

Service is accessible by:

- `servicename`
- `servicename.namespace`
- `servicename.namespace.svc`
- `servicename.namespace.svc.cluster.local`

### Deleting stuff

(service can be shorthanded to svc)

`kubectl delete service servicename` - deletes SERVICE (so basically exposure of an app)
`kubectl delete deployment httpbin` - deletes DEPLOYMENT (scales down and deletes pods)

## Secrets

Swarm secrets are encrypted at rest, at /run/secrets/, only specific app can read them.
Kubernetes secrets are stored in etcd, NOT encrypted by default, just base64-encoded.
So, we need to enable encryption for etcd (rather complex, not done for now).

ETCd is used across all services & replicated across nodes by kubernetes. It is a single source of truth for the whole cluster (pod specs, deployments, services - everything is stored there).
- ETCd runs only on control plane nodes, so only 1 instance if others are workers. If multiple control planes - etcd is replicated - contains copy of all the info about the whole cluster. Same with Swarm Raft store.

Helm is just a cli tool, it uses kubernetes API to store rollback history of services etc etc - in the kubernetes secrets itself.
Helm is like docker compose / stack for docker - can describe everything and deploy as a single stack.

### Other stuff (unsorted)

`expose deployment --type=NodePort` - exposes a random port (or specific, if you specify) to the host machine, same as docker `-p` ports.

To apply all yaml files from a folder:

`kubectl apply -f folder`

Restart containers/images etc:

`kubectl rollout restart deployment deploymentname1 deploymentname2`

or

`kubectl rollout restart deployment/deploymentname1 deployment/deploymentname2`

> Rollout restart makes kube to re-pull the image if needed, I guess.

Import locally built images from docker to k8s:

`docker save imagename:tagname | sudo ctr -n k8s.io images import -`

> `docker save` creates tar, `ctr` imports image to containerd for kubernetes.

Get logs of a deployment:

`kubectl logs deployment/deploymentname`

Scaling the deployment:

`kubectl scale deployment/deploymentname --replicas=3`

When testing different pods load balancing - browser caches live TCP connection. We need to use curl.

Quick script to test pod distribution:

`for i in $(seq 1 300); do curl -s http://localhost:30080/api/v1/latest; done | grep -o 'pod=[^ ]*' | sort | uniq -c`

> Golang TCP reuses connections even between requests, globally cached.

ServiceMesh solves one more problem: We don't need to disable persistent connections anymore (TCP handshakes are expensive, keep them cached) because sidecars connect to each other with persistent connections, BUT under the sidecar level it decides where traffic goes.

Kubectl describe pod - shows pod details including its containers.

Kubectl has init containers (exit right at start) and long-running init containers (start before our container, exit after our container). This is what servicemeshes like Istio use. It also prevents us from seeing its logs until actually asked for :)

## Istio

Installation:

```
pacman -S istio
istioctl install --set profile=minimal
istioctl install --set profile=default # Default (can be omitted) with ingress/egress.
```

> Minimal profile istall only IstioD - minimal thing - control plane without ingress/egress gateways.

Enable sidecar injection:

`kubectl label namespace default istio-injection=enabled`

Check mtls status:

`istioctl proxy-config secret deployment/service2-deployment`
`kubectl get peerauthentication -A` - if this returns nothing, we are in permissive mode.

To use strict mode, add to k8s folder:

```
spec:
  mtls:
    mode: STRICT
```

### Gateway & VirtualService

When we use MTLS in strict mode - we cannot access endpoints from outside because they won't trust us. We need ingress/egress.

Gateway - ISTIO resource (not k8s). Configures INGRESS gateway pod. Tells which ports to listen, which hostnames to accept. It doesn't route traffic anywhere, it just says "accept traffic" on a particular port.

`kubectl get svc istio-ingressgateway -n istio-system` - gets istio's ingress gateway that we need to use when we want to access services via NodePort. It is created by istio and has a random port.

In order to have static IP mapping to istio-ingressgateway, we need to have a load balancer type instead of nodeport - MetalLB.

### Visualization

```
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.21/samples/addons/prometheus.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.21/samples/addons/grafana.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.21/samples/addons/jaeger.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.21/samples/addons/kiali.yaml
istioctl dashboard kiali
istioctl dashboard grafana
istioctl dashboard jaeger
```
