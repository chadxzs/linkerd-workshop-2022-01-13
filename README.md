# linkerd-workshop

Notes and whatnot from a Linkerd workshop: "Locking down your Kubernetes cluster with Linkerd"

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
kubectl get -n emojivoto deploy -o yaml | linkerd inject - > installation/emojivoto.yaml
```






