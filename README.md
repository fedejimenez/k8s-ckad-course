# Kubernetes Certified Application Developer (CKAD) with Tests

## Content
- [Preparation](#preparation)
  - [Official k8s Docs](#official-k8s-docs)
  - [Useful Aliases](#useful-aliases)
- [Introduction](#introduction)
- [Section 2: Core Concepts](#section-2-core-concepts)
  - [Pods](#pods)
  - [Replication](#replication)
    - [Replication Controller](#replication-controller)
    - [Replica Set](#replica-set)
  - [Deployments](#deployments)
  - [Formatting Output](#formatting-output)
  - [Namespaces](#namespaces)
  - [Resource Quota](#resource-quota)
  - [Conventions](#conventions)
  - [Imperative Commands](#imperative-commands)
- [Section 3: Configuration](#section-3-configuration)
  - [Docker](#docker)
  - [Commands and Arguments](#commands-and-arguments)
  - [Editing Pods and Deployments](#editing-pods-and-deployments)
    - [Editing Pods](#editing-pods)
    - [Edifing Deployments](#editing-deployments)
  - [Environment Variables](#environment-variables)
  - [ConfigMaps](#configmaps)
  - [Secrets](#secrets)
  - [Encrypting Secret Data at Rest](#encrypting-secret-data-at-rest)


## Preparation

### Official k8s Docs:

- [Overview](https://kubernetes.io/docs/reference/kubectl/overview/)
- [Cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

### Useful Aliases

Add to your `~/.bashrc` / `~/.zshrc`

```bash
# kubernetes aliases
alias k='kubectl'
alias kc='k config view --minify | grep name'
alias kdp='kubectl describe pod'
alias krh='kubectl run --help | more'
alias ugh='kubectl get --help | more'
alias c='clear'
alias kd='kubectl describe pod'
alias ke='kubectl explain'
alias kf='kubectl create -f'
alias kga='kubectl get all'
alias kgp='kubectl get pods'
alias kgpl='kubectl get pods --show-labels'
alias kr='kubectl replace -f'
alias kh='kubectl --help | more'
alias krh='kubectl run --help | more'
alias ks='kubectl get namespaces'
alias l='ls -lrt'
alias ll='vi ls -rt | tail -1'
alias kga='k get pod --all-namespaces'
alias kgaa='kubectl get all --show-labels'
```

Then `source ~/.bashrc` / `source ~/.zshrc` to reload current terminal if needed.

# Introduction

> WIP

# Section 2: Core Concepts

## Pods

Each container is encapsulated in PODS. Definition example:

`pod-definition.yaml`

```bash
apiVersion: v1
kind: Pod
metadata:
 name: simple-webapp-color
spec:
 containers:
 - name: simple-webapp-color
   image: simple-webapp-color
```

- `kubectl run nginx  –-image nginx`
- `kubectl get pods`
- `kubectl get pods -o wide`
- `kubectl describe pod <pod-name>`
- `kubectl delete pod <pod-name>`


If you are not given a pod definition file, you may extract the definition to a file using the below command:

`kubectl get pod <pod-name> -o yaml > pod-definition.yaml`

To modify the properties of the pod, you can utilize the
`kubectl edit pod <pod-name>`
command. Please note that only the properties listed below are editable.

```bash
  spec.containers[*].image
  spec.initContainers[*].image
  spec.activeDeadlineSeconds
  spec.tolerations
  spec.terminationGracePeriodSeconds
```

Another way to define and create a pod:
- `kubectl run <pod-name> --image=<image-name> --dry-run=client -o yaml > <file-name>.yaml`

- `cat <file-name>.yaml`
- `kubectl create -f <file-name>.yaml`


## Replication
Helps balancing out demand and availability of the app.

- Replication Controller: older technology.
- Replica Set: new way to set up replication

#### Replication Controller

We can use pods templates

- `kubectl create -f rc-definition.yaml`
- `kubectl get replicationcontroller`


#### Replica Set
Need to define selector (which pods fall under it becasue it can use other pods)

```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx

replicas: 3
selector: 
  matchtLabels:
    type: front-end
```

- `kubectl create -f replicaset-definition.yaml`
- `kubectl get replicaset`
- `kubectl get rs`

Replace replicaset

`kubectl replace -f replicaset-definition.yaml`

Edit replicaset

`kubectl edit rs <replicaset-name>`

Scale the replicas (wihtout modifying the file)

- `kubectl scale --replicas=6 -f replicaset-definition.yaml`
- `kubectl scale --replicas=6 replicaset myapp-replicaset`
- `kubectl scale rs myapp-replicaset --replicas=6`

Delete

`kubectl delete replicaset my-appreplicaset  # Also deletes the underlaying pods`

## Deployments

Provides with capability to upgrade the underlying instance smoothly.

Create deployment definition file -> kind: Deployment

`kubectl create -f <file-name>.yaml`

It creates a replica set, and these create the underlying pods.

Get deployments

`kubectl get deployments`

Get all

`kubectl get all`

## Formatting Output

`kubectl [command] [TYPE] [NAME] -o <output_format>`  

Here are some of the commonly used formats:

- `-o json` Output a JSON formatted API object.
- `-o name` Print only the resource name and nothing else.
- `-o wide` Output in the plain-text format with any additional information.
- `-o yaml` Output a YAML formatted API object.

## Namespaces

Provide isolation, for example for dev / prod
Each can have their policies, who can do what. And limits.

`db-service.dev.svc.cluster.local`

- `db-service`: service name
- `dev`: namespace
- `svc`: specifies it's a service
- `cluster.local`: domain

Commands:
``kubectl get namespaces`
``kubectl get ns`
``kubectl get pods --namespace=kube-system`
``kubectl get pods -n=kube-system`
``kubectl create -f pod-definition.yml --namespace=dev`

Can be added into the pod definition file -> in the property `metadata`

Namespace definition file

`kubectl create -f namespace.yml`

Without definition file

`kubectl create namespace dev`

Set namespace in session

`kubectl config set-context $(kubectl config current-context) --namespace=dev`

Then kubectl get pod will be from that environment

View pod in all namaspaces

- `kubectl get pod --all-namespaces`
- `kubectl get pod -A`


If a service is within the same namespace, an application can access it just using its name `service_name`

If it's not in same namespace, then it has to use the full address with namespace
`service_name.namespace.srv.cluster.local`

## Resource Quota
To provide limits (amount of pods, request cpu, memory, etc)

## Conventions
- [k8s docs](https://kubernetes.io/docs/reference/kubectl/conventions/)

## Imperative Commands

While most of the time we work using definition files, imperative commands can help in getting one-time tasks done quickly, as well as generate a definition template easily.

Two options that can come in handy while working with the below commands:

`--dry-run`: By default, as soon as the command is run, the resource will be created. If you simply want to test your command, use the `--dry-run=client` option. This will not create the resource. Instead, tell you whether the resource can be created and if your command is right.

`-o yaml`: This will output the resource definition in YAML format on the screen.


Use the above two in combination along with Linux output redirection to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.


`kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml`


Create an NGINX Pod

`kubectl run nginx --image=nginx`

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

`kubectl run nginx --image=nginx --dry-run=client -o yaml`

Create a deployment

`kubectl create deployment --image=nginx nginx`

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

`kubectl create deployment --image=nginx nginx --dry-run -o yaml`

Generate Deployment with 4 Replicas

`kubectl create deployment nginx --image=nginx --replicas=4`

You can also scale deployment using the kubectl scale command.

`kubectl scale deployment nginx --replicas=4`

Another way to do this is to save the YAML definition to a file and modify

`kubectl create deployment nginx --image=nginx--dry-run=client -o yaml > nginx-deployment.yaml`

You can then update the YAML file with the replicas or any other field before creating the deployment.

Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

`kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml`

(This will automatically use the pod's labels as selectors)

Or

`kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml`
(This will not use the pods' labels as selectors; instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work well if your pod has a different label set. So generate the file and modify the selectors before creating the service)


Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

`kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml`

(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`

(This will not use the pods' labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

# Section 3: Configuration

## Docker

- `docker ps`
- `docker ps -a`
- `docker images`

docker run container command -> container created is not permanent, need to create an image for that

`docker-build -t name .`
Then
`docker run name`

You can specify a param with ENTRYPOINT

```bash
FROM Ubuntu
ENTRYPOINT ["sleep"]
```

`docker-build ubuntu-sleeper`
`docker run ubuntu-sleeper 10`

You can define both for default
```bash
FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

`docker run ubuntu-sleeper  # will be 5`

`cd` into Where the Dockerfile is and name it webapp-color

`docker build -t webapp-color .`

Build image with tag

`docker build -t webapp-color:lite .`

Change Entrypoint 

`docker run --name ubuntu ubuntu-sleeper --entrypoint sleep2.0 ubuntu-sleeper 10`


## Commands and Arguments

Equivalent from above docker example

```bash
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"]
      arg: ["10"]
```

`kubectl create -f pod-definition.yml`

command -> overrides ENTRYPOINT
args -> overrides CMD


You can also pass commands and args in the terminal

`kubectl run name --image=image-name --command=command-value -- arg1=arg1-value`


## Editing Pods and Deployments

### Editing Pods
Ww CANNOT edit specifications of an existing POD other than the below.

*   spec.containers\[\*\].image
*   spec.initContainers\[\*\].image
*   spec.activeDeadlineSeconds
*   spec.tolerations

For example we cannot edit the environment variables, service accounts, resource limits (all of which we will discuss later) of a running pod. There are 2 options to do so:

1\. `kubectl edit pod <pod name>`. This will open the pod specification in an editor (vi editor). Then edit the required properties. When you try to save it, you will be denied. This is because you are attempting to edit a field on the pod that is not editable.

A copy of the file with your changes is saved in a temporary location as shown above.

You can then delete the existing pod by running the command:

`kubectl delete pod webapp`

Then create a new pod with your changes using the temporary file

`kubectl create -f /tmp/kubectl-edit-ccvrq.yaml`

Or you can also replace the pod from the tmp file created

`kubectl replace --force -f /tmp/temp-file-name.yaml`

2\. The second option is to extract the pod definition in YAML format to a file using the command

`kubectl get pod webapp -o yaml > my-new-pod.yaml`

Then make the changes to the exported file using an editor (vi editor). Save the changes

`vi my-new-pod.yaml`

Then delete the existing pod

`kubectl delete pod webapp`

Then create a new pod with the edited file

`kubectl create -f my-new-pod.yaml`


### Editing Deployments

With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification,  with every change the deployment will automatically delete and create a new pod with the new changes. So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command

`kubectl edit deployment my-deployment`


## Environment Variables

Example in docker

`docker run -e APP_COLOR=pink app-color`

How it looks in the pod definition:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: app-color
soec:
  containers:
  - name: app-color
    image: app-color
    env:
      - name: APP_COLOR
        value: pink
```

But there are other ways as well like ConfigMaps and Secrets

## ConfigMaps

To take the env variables out of the definition file and managed them separately.
Are used to pass configuration data in the form of key-value pairs. 

1. We need to create the configMaps.
  - Imperative way: without definition file:

  `kubectl create configmap <config-name> --from-literal=<key>=<value>`

  Examples:

  `kubectl create configmap app-config --from-literal=APP_COLOR=blue`

  `kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod`

  - Imperative way: with a definition file:

  `kubectl create configmap <config-name> --from-file=<path-to-file>`

  Example:

  `kubectl create configmap app-config --from-file=app_config.properties`

  - Declarative way: with a definition file.

  `kubectl create -f config-map.yaml`

  ```bash
  apiVersion: v1
  kind: ConfigMap
  metadata: 
    name: app-config
  data:
    APP_COLOR: blue
    APP_MODE: prod
  ```

  Name appropiately to not create confusion when they're used.

View config maps

`kubectl get confirmaps`

``kubectl describe cm <name>`


2. Inject the ConfigMaps into the POD.

Inside the POD definition:

```bash
...
spec:
  containers:
  ...
  envFrom:
    - configMapRef:
        name: app-config
```
 
## Secrets

Are used to store sentitive information (passwords, keys, etc.)
Similar to configMaps, except that they're stored in an encoded format.

Also they're two steps involved: creation and injection.

1. Creation:

- Imperative way example:

`kubectl create secret generic app-secret --from-literal=DB_HOST=mysql`

- Declarative way example: 

`kubectl create -f secret-data.yml`

```bash
...
data:
  DB_HOST: mysql # encoded
  DB_USER: root # encoded
  DB_PASSWORD: postgres # encoded
```

To encode:

`echo -m 'mysql' | base64`

Get secrets:

`kubectl get secrets`
`kubectl describe secrets`

View values:

`kubect get secret app-secret -o yaml`

To decode:

`echo -n 'bXSl=qws-' | base64 --decode`


2. Inject the ConfigMaps into the POD.

Inside the POD definition:

```bash
...
spec:
  containers:
  ...
  envFrom:
    - secretRef:
        name: app-secret
```
 
*NOTE: secrets are not encrypted, only encoded!*


## Encrypting Secret Data at Rest

> WIP
