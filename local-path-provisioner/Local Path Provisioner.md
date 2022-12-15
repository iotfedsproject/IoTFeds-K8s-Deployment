[[_TOC_]]

For better management of the PersistentVolume we installed a provisioner. With this tool we only have to create a PersistentVolumeClaim

## Installation

The [GitHub page](https://github.com/rancher/local-path-provisioner)

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

## Default StorageClass

Make the storageclass of the provisioner default for easier mangement of storages

Male the the local-path storageclass the default one.

```bash
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## Test

To test that the provisioner is working run the below commands

### Creation of pods and pvc

```bash
kubectl create -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/examples/pvc/pvc.yaml
kubectl create -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/examples/pod/pod.yaml
```
### Save data

With the below commands we are creating some data to be saved and we delete the pod and create a new one to check the persisence.

```bash
kubectl exec volume-test -- sh -c "echo local-path-test > /data/test"
kubectl delete -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/examples/pod/pod.yaml
kubectl create -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/examples/pod/pod.yaml
kubectl exec volume-test -- cat /data/test
```
### Remove testing data

```bash
kubectl delete -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/examples/pod/pod.yaml
kubectl delete -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/examples/pvc/pvc.yaml
```