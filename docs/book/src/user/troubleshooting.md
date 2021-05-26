# Troubleshooting

## Labeling nodes with reserved labels such as `node-role.kubernetes.io` fails with kubeadm error during bootstrap

Self-assigning Node labels such as `node-role.kubernetes.io` using the kubelet `--node-labels` flag
(see `kubeletExtraArgs` in the [CABPK examples](https://github.com/kubernetes-sigs/cluster-api/tree/master/bootstrap/kubeadm))
is not possible due to a security measure imposed by the
[`NodeRestriction` admission controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#noderestriction)
that kubeadm enables by default.

Assigning such labels to Nodes must be done after the bootstrap process has completed:

```
kubectl label nodes <name> node-role.kubernetes.io/worker=""
```

For convenience, here is an example one-liner to do this post installation 

```
kubectl get nodes --no-headers -l '!node-role.kubernetes.io/master' -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}' | xargs -I{} kubectl label node {} node-role.kubernetes.io/worker=''
```

## Cluster API with Docker

When provisioning workload clusters using Cluster API with Docker infrastructure,
the workload cluster might get stuck in a provisioning state if there are oprhaned Docker volumes or resources on your machine.
Docker volumes are not removed by default because they contain data, so if you stop a container sometimes the volume will hang around in your system.

The [docker system df](https://docs.docker.com/engine/reference/commandline/system_df/) command gives details about disk space consumed by Docker resources.
```
docker system df
```
The [docker volume prune](https://docs.docker.com/engine/reference/commandline/volume_prune/) command will remove all volumes that are not used by at least one container.
```
docker system prune --volumes
```