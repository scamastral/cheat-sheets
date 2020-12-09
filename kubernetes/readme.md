# List of kubectl commands:

In general, these are the most common commands that can be used with pods / nodes / services / deployments etc.

```bash
$ kubectl get xyz
$ kubectl describe xyz my-xyz
$ kubectl apply -f xyz-yaml-file.yml
$ kubectl delete xyz my-xyz
```

## Cluster

#### Get information about the current cluster

The context is set to this cluster in your `~/.kube/config` file.

```bash
$ kubectl cluster info
```

## Nodes

#### Get all nodes on a cluster
```bash
$ kubectl get nodes
```

## Pods

#### Get all pods on a cluster
```bash
$ kubectl get pods

# --watch flag for live reload
$ kubectl get pods --watch

# --show-labels flag displays the pod's labels
$ kubectl get pods --show-labels

# -o wide flag shows all detail information
# useful to see on which nodes the pods are running on
$ kubectl get pods -o wide
```

#### Get detailed information for a pod
```bash
$ kubectl describe pods your-pod-name
```

#### Create a pod on a cluster

You need a yaml file describing the pod as a parameter.  
`-f` flag tells kubernetes that we are creating it declaratively.

```bash
$ kubectl apply -f your-pod-yaml-file.yml
```

#### Delete a pod from a cluster

There are 2 ways to delete a pod:

```bash
$ kubectl delete pod your-pod-name

$ kubectl delete -f your-pod-yaml-file.yml
```

## Services

#### Get all services from a cluster
```bash
$ kubectl get svc
```

#### Create a service on a cluster

You need a yaml file describing the service as a parameter.  
`-f` flag tells kubernetes that we are creating it declaratively.

```bash
$ kubectl apply -f your-service-yaml-file.yml
```

## Deployments

#### Get all deployments from a cluster
```bash
$ kubectl get deploy
```

#### Make a deployment to a cluster

You need a yaml file describing the pod as a parameter.  
`-f` flag tells kubernetes that we are creating it declaratively.

```bash
$ kubectl apply -f your-deployment-yaml-file.yml
```

## Replica Sets

#### Get all replica sets from a cluster
```bash
$ kubectl get rs
```

## Endpoints

#### Get all endpoints from a cluster
```bash
$ kubectl get ep
```

#### Get detailed information for an endpoint
```bash
$ kubectl describe ep your-ep-name
```

# Kubernetes Deployments with kubectl
This is an example of how you can deploy a new version of your app with a rolling update.  
Pods will be replaced step by step until all the old pods are gone and only the new version of your app is served.

#### Deploying a new version
Now we modify our `my-deployment-yaml-file.yml` with a new version of our app for example.  
Deploy it using this command:

```bash
$ kubectl apply -f my-deployment-yaml-file.yml
``` 

#### Observe the process

We can use the following commands in 2 different terminal windows to see the live progress of the rolling update

```bash
# list all the pods in terminal 1
$ kubectl get pods --watch

# see the update
$ kubectl rollout status deploy my-deployment
```

Now you should see the progress in the before prepared terminals updated live.

#### Rollback to old version

With this command we can see that the replica set with the old app version is still here:
```bash
$ kubectl get rs
```

Take the name of the replica set from the output of the command above and use it to see additional information about the replica set.  
We can for example see, for which image version the replica set was managing replicas for.
```bash
$ kubectl describe rs my-rs-name
```

We can take a look at the rollout history using this command:
```bash
$ kubectl rollout history deploy my-deployment
```

The output from the command above shows us the different revisions we rolled out.
We can roll back to the older version in case we need to using the following commands:
```bash
# rollback to revision 1
$ kubectl rollout undo deploy my-deployment --to-revision=1
```

Now all the pods should be back to the previous version from revision 1.