# linkerd-workshop

Notes and whatnot from a Linkerd workshop: "Locking down your Kubernetes cluster
with Linkerd"

```
# install linkerd cli
brew install linkerd

# configure kubectl and linkerd cli for minikube
export KUBECONFIG=~/.minikube/config/minikube

# do a pre-flight check prior to install linkerd
linkerd check --pre

# write the linkerd install file to this project
linkerd install > installation/linkerd-install.yaml

# apply the linkerd installation yaml
kubectl apply -f installation/linikerd-install.yaml

# generate linkerd viz on-cluster metrics dashboard
linkerd viz install > installation/viz-install.yaml

# apply viz installation yaml
kubectl apply -f installation/viz-install.yaml

# download emojivoto
curl -sL run.linkerd.io/emojivoto.yml > installation/emojivoto.yaml

# install emojivoto
kubectl apply -f installation/emojivoto.yaml

# test emojivoto out
kubectl -n emojivoto port-forward svc/web-svc 8080:80
open https://localhost:8080

# inject linkerd into emojivoto
kubectl get -n emojivoto deploy -o yaml | linkerd inject - > installation/emojivoto-injected.yaml

# apply injected app
kubectl apply -f installation/emojivoto-injected.yaml
```

Notes:

Blog
post: https://buoyant.io/2021/12/14/locking-down-network-traffic-between-kubernetes-namespaces/

* Authorization policy in 2.11
    * control over communication allowed in cluster
    * Linkerd (and Kubernetes) allows all communication between pods. Now it has
      the ability to deny communication
    * In place today: server-side policies. server enforcing traffic. Only
      restricting connections today, not individual requests
    * 2.12 will work towards client-side policies and more fine-grained policies
* AuthorizationPolicy meant to be a replacement for Kubernetes NetworkPolicy
    * Uses workload identity (ServiceAccount)
    * built on top of mTLS (same as Linkerd in general)
    * enforced at pod level (important for zero trust)
    * will be able to capture L7 semantics in 2.12+
    * NetworkPolicy is still better for things like UDP traffic
* AuthorizationPolicy is the first possibility using Linkerd to "shoot yourself
  in the foot"
* Mechanics:
    * There's a default policy, typically set through
      `config.linkerd.io/default-inbount-policy` annotation.
        * Every cluster has a cluster-wide default policy. By default it's "
          all-unauthenticated" (changing nothing)
        * Default policy can be overriden at namespace or workload level
        * default policy fixed at startup time. when changed, need to restart
          pod
    * Two new CRDs: Server and ServerAuthorization
* Default policies
    * all-unauthenticated - allow all
    * cluster-unauthenticated - allow from clients with source IPs in cluster
    * all-authenticated - allow from clients with Linkerd's mTLS
    * cluster-authenticated - combination of previous two
    * deny
* CRDS
    * Server CRD
        * selects a port over a set of pods in a namespace
        * can match multiple workloads
        * by themselves servers deny all traffic!
    * ServerAuthorization CRD
        * selects over one or more servers and describes the type of traffic
          allowed to those servers
        * can match multiple servers
        * can allow based on what ServiceAccount the client is using
* Cluster-wide default policy is the only way to apply a cluster-wide policy
* When rejected
    * if it's a gRPC connection: `grpc-status: PermissionDenied`
    * if it's an HTTP connection: `403 Forbidden` response code
    * otherwise denial = refused TCP connection
* Gotchas
    1. Kubelet probes need to be authorized
        - if building a deny by default setup, need to make sure kubeletes are
          authorized or they won't start
    2. Default policies are not read dynamically
        - can change with `linkerd update` cli tool command
    3. Ports must be in the pod spec
        - if a server references a port, it must be in the pod spec

new commands:

```
 curl -sL run.linkerd.io/emojivoto-policy.yml > installation/emojivoto-policy.yaml
 kubectl apply -f installation/emojivoto-policy.yaml
```

At this point he was showing some of the same stuff that was being shown in the
blog post above
