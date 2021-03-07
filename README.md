# minikube-nfs-test

## Purpose
Various resources to install an NFS server / client on minikube.
This NFS server should be made available as PV and PVC to be used by pods. On this example we just write every 5 seconds on a `file.txt` inside the NFS share. 

## Setup
There are 2 options for the nfs server, to run on minikube and to run on docker-compose on the host machine.

### Setup with NFS on minikube
Verify minikube is running:
```
labros@beastx:~$ minikube status -p minikube-test
minikube-test
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
timeToStop: Nonexistent
```

Create namespace:
```shell
kubectl create namespace nfs
or
kubectl create -f nfs/nfs-namespace.yml
```

Create the nfs server
```shell
kubectl apply -f nfs/nfs-server.yml -n nfs
```

Create the nfs client,
```shell
kubectl apply -f nfs/nfs-client.yml -n nfs
```
> Make sure you find and update the IP of the PV using `kubectl get svc -l name=nfs-server -n nfs` prior to creating the client

Then you can check that the client is indeed writing to the NFS server by exec into the server pod and checking the file `/data/nfs/file.txt`, i.e.
```shell
kubectl get pod -n nfs | grep nfs-server
nfs-server-68d7965f59-7n9vb
kubectl exec -it -n nfs nfs-server-68d7965f59-7n9vb -- bash
tail -f /data/nfs/file.txt
```

### Setup with NFS on docker-compose
Goto `dc` folder and run the docker-compose there.
In this case the NFS server is exposed on localhost:2049/

```shell
cd dc
docker-compose up -d
# To see the logs
docker-compose logs -f
```


## References
* The image of nfs server, https://hub.docker.com/r/itsthenetwork/nfs-server-alpine
* How to create nfs server, https://stackoverflow.com/questions/64954206/k8s-persistentvolume-shared-among-multiple-persistentvolumeclaims-for-local-test
* For the pod as nfs client, https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-volumes-example-nfs-persistent-volume.html
