## Setting up a new cluster (one time job)

1. Setup the correct env variables in `ansible/vars/environments` for the cluster
2. Change to the `ansible` folder

   ```
   cd ansible
   ```

3. Run ansible-notebook with the correct env and pass it with the `dp` flag.

   ```
   ansible-playbook inflation.yml -e dp=some-cluster-1 -vvvv
   ```

4. Add the SSH public key from the Flux Operator to your Git account

# Minikube

>Recommended Minikube configuration __if istio is enabled__
>```
>minikube config set cpus 6
>minikube config set memory 16384
>minikube config set disk-size 64g
>```

Run the Ansible playbook to set up the minikube cluster. Either with istio enabled or nginx controller.

```
# Istio enabled
ansible-playbook inflation.yml -e dp=minikube-istio -vvvv

# Nginx enabled
ansible-playbook inflation.yml -e dp=minikube -vvvv
```

## 1. Create minikube Flux ssh private key

Create a Kubernetes secret

```
kubectl -n flux create secret generic flux-ssh --from-file=./identity
```

>__If not provided or you want to create a new one__, generate a SSH key named identity (and create the secret from the last step).
>
>```
>ssh-keygen -q -N "" -f ./identity
>```
>
>Add ./identity.pub to your Git account under SSH keys.

## 2. Replace sealed secrets private key

Obtain the private sealed secrets key

```
kubectl replace -f sealed-secrets-key.yaml
```

Restart the sealed secrets controller by deleting the pod

```
kubectl delete pod -n kube-system "$(kubectl get -n kube-system pod -l 'app.kubernetes.io/name=sealed-secrets'  -o jsonpath='{.items..metadata.name}')"
```

## 3. Setup MetalLB (bare metal load-balancer) on Minikube

Check that the MetalLB pods are running

```
kubectl get pods -n metallb-system
```

Get the minikube ip

```
minikube ip
```

Result example:

```
192.168.99.100
```

Set minikube ip as a variable

```
MINIKUBE_IP=$(minikube ip)
```

Apply this configuration:

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: custom-ip-space
      protocol: layer2
      addresses:
      - ${MINIKUBE_IP}/28
EOF
```
