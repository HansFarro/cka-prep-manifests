# cka-prep-manifests

Practical guide with manifests to help you to prepare for CKA (Certified Kubenetes Administrator) exam.

## Table of Contents

[Tips](#tips)

[Chapter 2 Scheduling](#chapter-2-scheduling)

[Chapter 3 Monitoring](#chapter-3-monitoring)

[Chapter 4 Application Lifecycle Management](#chapter-4-application-lifecycle-management)

[Chapter 5 Cluster maintenance](#chapter-5-cluster-maintenance)

[Chapter 6 Security](#chapter-6-security)

[Chapter 7 Storage](#chapter-7-storage)

[Chapter 8 Networking](#chapter-8-networking)

[Chapter 9 Troubleshooting](#chapter-9-troubleshooting)
## Tips

Create a deployment without creating in the cluster

`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

Make an export variable for that using

`export do='--dry-client -o yaml'`

Force replace of container

`kubectl replace --force -f ubuntu-sleeper-3.yaml`

Organize files per questions : Create a directory and change to it

`mkcd(){ mkdir $@ && cd @; }`

Query ApiVersion and shortnames

`kubectl api-resources`

## Chapter 2 Scheduling

### Node Affinity Types

Available

- requiredDuringSchedulingIgnoredDuringExecution
- preferedDuringSchedulingIgnoredDuringExecution

Planned:

- requiredDuringSchedulingRequiredDuringExecution

|      |DuringScheduling|DuringExecution|
|------|----------------|---------------|
|Type 1|    Required    |    Ignored    |
|Type 2|    Preferred   |    Ignored    |
|Type 3|    Required    |   Required    |

## Chapter 3 Monitoring

### Logs - Kubernetes

`kubectl logs -f <pod_name> <container_name>`

## Chapter 4 Application Lifecycle Management

### Rollback

`kubectl rollout status deployment/<deployment_name>`
`kubectl rollout history deployment/<deployment_name>`
`kubectl rollout undo deployment/<deployment_name>`

Change container image

`kubectl set image deployment/<deployment_name> <container_name>=<new_container_image>`

Docker vs Kubernetes

```bash
ENTRYPOINT <Array> - command: <Array>
CMD <Array> - args: <Array>
```

ENV Value Types

1. Key value

```yaml
env:
  - name: APP_COLOR
    value:
```

2. Config Map

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
```

3. Secret

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
```

### Create a secret in the imperative way

`kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123`

Use envFrom to define all of the Secret's data as container environment variables.

```yaml
envFrom:
  - secretRef:
    name: mysecret
```

## Chapter 5 Cluster maintenance

### OS Upgrades

Drain the node means pods are terminated and node is marked as unschedulable

`kubectl drain node-1`

To be unschedulable

`kubectl cordon node-1`

To be schedulable a node :

`kubectl uncordon node-1`

### Upgrades

Verify kubeadm version

`kubeadm version`

Verify the upgrade plan

`kubeadm upgrade plan`

Upgrade kubelet to specific version, don't add -00

`sudo kubeadm upgrade apply v1.24.0`

### Backup and Restore Methods

Help etcdctl

`etcdctl -h`

Take a snapshot of etcd

`export ETCDCTL_API=3`

`etcdctl snapshot save -h`

Since our ETCD database is TLS-Enabled, the following options are mandatory:

--cacert verify certificates of TLS-enabled secure servers using this CA bundle

--cert   identify secure client using this TLS certificate file

--endpoints=[127.0.0.1:2379] This is the default as ETCD is running on master node and exposed on localhost 2379.

--key    identify secure client using this TLS key file

Example:

`etcdctl snapshot save --endpoints="https://127.0.0.1:2379" --key="/etc/kubernetes/pki/etcd/server.key" --cert="/etc/kubernetes/pki/etcd/server.crt" --cacert="/etc/kubernetes/pki/etcd/server.crt" /opt/snapshot-pre-boot.db`

Snapshot restore

`etcdctl snapshot restore -h`

Example

`etcdctl snapshot restore --data-dir /var/lib/etcd-from-backup /opt/snapshot-pre-boot.db`

Edit this file /etc/kubernetes/manifests/etcd.yaml

```yaml
hostPath:
  path: <change_this_to_datadir>
```

## Chapter 6 Security

Get content of csr (Certificate Signing Request) for a new user

`cat /root/<file_name>.csr | base64 | tr -d "\n"`

Approve a CSR with kubectl

`kubectl certificate approve <certificate_signing_request_name>`

Deny a CSR with kubectl

`kubectl certificate deny <certificate_signing_request_name>`

Check access

`kubectl auth can-i <command>`
`kubectl auth can-i create deployments`

Check access as administrator

`kubectl auth can-i <command> --as <role>`

Get the API groups and resource names from command

`kubectl api-resources`

Create a Secret by providing credentials on the command line

`kubectl create secret docker-registry <secret_name> --docker-server=<your_registry_server> --docker-username=<your_name> --docker-password=<your_pword> --docker-email=<your_email>`

Security contexts at pod level

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
```

Security contexts at container level

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-4
spec:
  containers:
  - name: sec-ctx-4
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
```

Get Network Policies

`kubectl get netpol`

[Real example of a network policy definition](security/2-network-policy.yaml)

## Chapter 7: Storage

### Persistent Volume and Persistent Volume Claim

Get PV and PVC

`kubectl get pv,pvc`

[Example of PV and PVC](storage/pv-pvc.yaml)

### Storage Class

Get storage classes

`kubectl get sc`

[Example of StorageClass and PVC](storage/sc-pvc.yaml)

## Chapter 8: Networking

- Inside the control plane run the following command to see which network plugin is used:

`ps aux | grep -e "--network-plugin"`

- The CNI binaries are located under */opt/cni/bin* by default.

- To identify which binaries are present in CNI plugin run `ls /opt/cni/bin`

- Run the command: `ls /etc/cni/net.d/` and identify the name of the plugin.

- See executable file `cat /etc/cni/net.d/10-flannel.conflist`.

- See kube-system namespace to see which network solution is used for. `kubectl get po -n kube-system -o wide`

- To know which IP Range is configured for the services within the cluster inspect the following: `cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range`

- How the Kubernetes cluster ensure that a kube-proxy pod runs on all nodes?  `k get ds -n kube-system`

### Core DNS

CoreDNS is DNS server that can server as Kubenetes cluster DNS.You can see more info in.

`kubectl describe cm coredns -n kube-system`

[ip_address or hostname of pods or services].[namespace].[type: pod or svc].[dns_server_name]

### Ingress

Format - kubectl create ingress <ingress-name> --rule="host/path=service:port"

Example - kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"

## Chapter 9 Troubleshooting

## Application Failure

To check the Application/Service status of the webserver `curl http://web-service-ip:node-port`

To check the endpoint of the service and compare it with the selectors `kubectl describe service web-service`

## Control Plane Failure

To check status of controlplane services

```bash
service kube-apiserver status
service kube-controller-manager status
service kube-scheduler status
service kubelet status
service kube-proxy status
```

To view the kube-apiserver logs
`sudo journalctl -u kube-apiserver`

## Scale resources

`kubectl scale --replicas=<number> rs/<release_name>`

`kubectl scale --replicas=<number> deployment/<deployment_name>`