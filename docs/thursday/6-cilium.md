# Become a Cilium Badass (Workshop)

## Setting up

```bash
# Install k3s
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.27.5+k3s1" INSTALL_K3S_EXEC="--flannel-backend=none --disable servicelb --disable traefik --disable-network-policy" sh -

# Install Cilium
helm repo add cilium https://helm.cilium.io/
helm install cilium cilium/cilium --version 1.14.1 -n kube-system -f /tmp/cilium-values.yaml

# Verify Cilium
watch kubectl get pods -n kube-system
```

```yaml
# cilium-values.yaml
operator:
  replicas: 1
```

## Cilium CLI

```bash
# Check connectivity
cilium connectivity test
```

## Hubble

```bash
# Deploy a demo app. Just deployment and service, nothing fancy
kubectl create namespace demo
kubectl apply -f /tmp/demo-app -n demo
```

```bash
# Enable hubble and hubble UI
helm upgrade cilium cilium/cilium -f /tmp/hubble-ui-values.yaml -n kube-system
```

```yaml
# hubble-ui-values.yaml
operator:
  replicas: 1
hubble:
  relay:
    enabled: true
  ui:
    enabled: true
    service:
      type: NodePort
      nodePort: 31235
```

## Network policies

Cilium network policies can go down to layer 4, which gives more control over allowing and blocking traffic.

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: noyoucant
  namespace: netpolicy
spec:
  endpointSelector:
    matchLabels:
      app: nginx
  ingress:
    - fromEndpoints:
        - matchLabels:
            cani: yesyoucan
            io.kubernetes.pod.namespace: default
      toPorts:
        - ports:
            - port: "80"
              protocol: TCP
```
