# Gateway Operator

Simple k8s operator to deploy a pod.

## Development

### Pre-requisites

* minishift installed
* [dep]() installed
* golang 1.9+ installed

### Setup
* define a new minishift profile
```
minishift profile set operator
minishift config set memory 8GB
minishift config set cpus 3
minishift config set vm-driver xhyve
minishift addon enable admin-user
```

* install operator SDK
```
go install github.com/operator-framework/operator-sdk/commands/operator-sdk
```

* install OLM
```
wget https://github.com/operator-framework/operator-lifecycle-manager/archive/master.zip
unzip master.zip
alias k=kubectl
k create namespace tectonic-system
k apply -f operator-lifecycle-manager-master/deploy/tectonic-alm-operator/manifests/0.4.0/
```

* create new namespace/project
```
oc new-project gtw
```

### Build new operator

* initial scaffold of operator (already done)
```
$GOPATH/bin/operator-sdk new gtw-operator --api-version=gtw.fabric8.org/v1alpha1 —kind=GTW
```
This initial scaffold have generated best-practices project with metadata understand by operator sdk.
rgis step is not to be redone. It's a one shot generation.

* generate code on types.go changes
Do changes in types.go => code-generation for custom resources
```
$GOPATH/bin/operator-sdk generate k8s
```

### Package
* doker image in minishift

```
eval $(minishift docker-env)
docker login -u developer -p $(oc whoami -t) $(minishift openshift registry)
$GOPATH/bin/operator-sdk build 172.30.1.1:5000/gtw/operator:v0.0.1
docker push 172.30.1.1:5000/gtw/operator:v0.0.1
```

> Note: `operator-sdk build` generate the docker images. Thefore give a name with proper registry-url.

* deploy
```
oc apply -f deploy/rbac.yaml
oc apply -f deploy/operator.yaml
oc apply -f deploy/crd.yaml
```
> Note: if crd not deployed operator fails

* test
```
oc apply -f deploy/cr.yaml
oc delete gtw gateway-example
```
==> observe the new pod created

### Dev environment
In dev mode, no need to package in container and deploy, simply run your operator locally:

```
$GOPATH/bin/operator-sdk up local --kubeconfig=$HOME/.kube/config --namespace=gtw
```

> Note: if namespace is not specified, il apples to `default`

## Links
* [Developing Kubernetes Operator is now easy with Operator Framework](https://devops.college/developing-kubernetes-operator-is-now-easy-with-operator-framework-d3194a7428ff)