# Managing Secrets at Scale (Workshop)

If you have a handful of clusters, it's fine to work with a test pipeline: build -> push -> deploy. 

However, when you have an entire set of clusters per client, so 100+ clusters, having a pipeline does not scale. To resolve this, split up the CI and the CD part. Build an artifact in the pipeline and let
GitOps handle the rest. This works great at scale.

## The problem

How do you handle secrets? In a pipeline, you can push the secrets to the cluster. However, in a pull scenario, you cannot put secrets in Git, because bad.

## SOPS with Age

[sops](https://github.com/getsops/sops) is a tool built for encrypting and decrypting secrets. [Age](https://github.com/FiloSottile/age) is an encryption tool.

```bash
# Create a keypair with age. The public part is in stdout, the private part is in ~/.age/key.txt
age-keygen --output ~/.age/key.txt

# Encrypt with age
age --encrypt --recipient <your public key> --output wordpress.yaml.bin wordpress.yaml

# Decrypt using age
age --decrypt --identity ~/.age/key.txt wordpress.yaml.bin
```

You can commit the _wordpress.yaml.bin_ to git, but Argo can't do anything with it, since it's encrypted.

```bash
# Create the sops secret with your private key
kubectl apply -f sops-age-key-file.yaml
```

```yaml
# sops secret
---
apiVersion: v1
stringData:
  key: "AGE-SECRET-KEY-1M7U0VD2TMR3KCTXKS8Q8WEMX4Z2RS6CC3XJJ6C69VT5P32WVRQ9SS0E22R"
kind: Secret
metadata:
  name: sops-age-key-file
  namespace: sops
type: Opaque
```

Deploy the [sops-secret-operator](https://isindir.github.io/sops-secrets-operator/) using Helm and Argo.

```bash
# Encrypt a secret
sops --encrypt --age <your public key> --encrypted-suffix Templates example-sops-secret-template.yaml | tee example-sops-secret.yaml
```

This creates a `SopsSecret` which you can deploy using Argo. The sops operator creates a normal `Secret` from this.

```bash
# Deploy Wordpress
age-keygen -y ~/.age/key.txt # Print public key
sops --encrypt --age age1w0yen6z758mtjmlefcwlfylg5zs77m3pjkuj9ay329w2trw9hvhqk3sjhr --encrypted-suffix Templates mysql-sops-secrets-template.yaml | tee mysql-sops-secrets.yaml
sops --encrypt --age age1w0yen6z758mtjmlefcwlfylg5zs77m3pjkuj9ay329w2trw9hvhqk3sjhr --encrypted-suffix Templates wordpress-sops-secrets-template.yaml | tee wordpress-sops-secrets.yaml
```

## External secrets

[External secrets](https://external-secrets.io/latest/) pull secrets from another location into the cluster.

Not going too deep into this. You create a `SecretStore`, for example AWS Secrets Manager or Azure Vault. You create an `ExternalSecret` that references the store and puts the values into a Kubernetes Secret.
