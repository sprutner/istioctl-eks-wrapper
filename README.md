# platform.infrastructure.istioctl.auth
A wrapper workaround for istioctl authentication with heptio.

## Pre-reqs
kubectl version 1.10+

AWS CLI with the EKS module installed. To install the EKS module to your AWS CLI (until it's added to the CLI bu Amazon):
```
curl -O https://amazon-eks.s3-us-west-2.amazonaws.com/2018-04-04/eks-2017-11-01.normal.json
aws configure add-model --service-model file://eks-2017-11-01.normal.json --service-name eks
```
Heptio authenticator with the location added to your $PATH. You can install it with go. If you need to install go, see https://golang.org/doc/install. You can install it with homebrew on Mac.

### Install heptio
```
go get -u -v github.com/heptio/authenticator/cmd/heptio-authenticator-aws
```
Kubeconfig file in ~/.kube/config with a similar config as this:
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <your cluster certificate>
    server: <your cluster URL>
  name: mycluster
contexts:
- context:
    cluster: mycluster
    user: mycluster-admin
  name: mycluster-context
current-context: mycluster-context
kind: Config
preferences: {}
users:
- name: mycluster-admin
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      args:
      - token
      - -i
      - <EKS cluster name>
      command: heptio-authenticator-aws
      env: null
```

Be sure to replace ```<your cluster certificate>```, ```<your cluster URL>```, and ```<EKS cluster name>``` with the proper values.

## Usage
Authenticate your aws cli as you normally do with aws-adfs, but do not specify a profile. Your AWS token must be in your default profile. Run the istioctl command just as you would without this wrapper
```
./istioctl get routerules
```
This will determine your cluster name by looking at your kubeconfig current context and grabbing the cluster name from the args in the user section of the current context you are in.

You can add the path to where ever you clone this repo to run istioctl without the "./" in front.