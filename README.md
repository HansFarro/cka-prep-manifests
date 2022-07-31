# cka-prep-manifests
Practical guide with manifests to help you to prepare for CKA (Certified Kubenetes Administrator) exam.

## Tips

Create a deployment without creating in the cluster

`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

Force replace of container

`kubectl replace --force -f ubuntu-sleeper-3.yaml`
## Chapter 2 : Scheduling
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

## Chapter 3: Monitoring
### Logs - Kubernetes

`kubectl logs -f <pod_name> <container_name>`

## Chapter 4: Application Lifecycle Management

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
```bash
env:
  - name: APP_COLOR
    value:
```

2. Config Map
```bash
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
```

3. Secret
```bash
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
```

Create a secret in the imperative way

`kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123`

Use envFrom to define all of the Secret's data as container environment variables.

      envFrom:
      - secretRef:
          name: mysecret