# platform.infrastructure.istioctl.auth
A wrapper workaround for istioctl authentication with heptio.

## Description

This bash script is a wrapper that creates a temporary config file at ~/.kube/config-istio and uses it to authenticate with heptio authenticator and allow istioctl access to EKS

## Pre-reqs
kubectl version 1.10+

The latest version of the AWS CLI, which has EKS in it.

Heptio authenticator with the location added to your $PATH. Here is how I installed it with ZSH. Modify where you copy the location to your path apporiately (~/.bash_profile on MacOS) if you are using bash.
```
curl -o heptio-authenticator-aws https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-06-05/bin/darwin/amd64/heptio-authenticator-aws
chmod +x ./heptio-authenticator-aws
mkdir ~/bin
cp ./heptio-authenticator-aws ~/bin/
echo 'export PATH=$HOME/bin:$PATH' >> ~/.zshrc
```

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

**SET YOUR $AWS_DEFAULT_REGION ENV VAR TO THE REGION EKS IS IN**
Authenticate your aws cli as you normally do with aws-adfs, but do not specify a profile. Your AWS token must be in your default profile. Run the istioctl command just as you would without this wrapper
```
./istioctl get routerules
```
This will determine your cluster name by looking at your kubeconfig current context and grabbing the cluster name from the args in the user section of the current context you are in.

You can add the path to where ever you clone this repo to run istioctl without the "./" in front.

## Troubleshooting

- If this fails to query AWS properly, you will get a bunk configuration file at ~/.kube/config-istio. One common example would be the wrong AWS region being set. You need to manually delete the file, as it will not be overwritten on your second attempt. Check your AWS `echo $AWS_DEFAULT_REGION` Set the AWS region with `export AWS_DEFAULT_REGION=<region>` and `rm ~/.kube/config-istio`.
- You can get path issues with istioctl where your shell will be trying to run the original istioctl binary instead of the wrapper. Run a `which istioctl` to be sure.

## Credits

Credit goes to @jaredeis for sharing this wrapper with the community.
