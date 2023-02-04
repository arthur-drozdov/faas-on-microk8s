# OpenFaaS on MicroK8s Workshop

## Prerequisites

Install Docker: https://docs.docker.com/get-docker/
Install MicroK8s: https://microk8s.io/#install-microk8s

## Deploy your first OpenFaaS function on Kubernetes!

### Create a microk8s cluster

Create the microk8s cluster with the required addons including openfaas
```
microk8s install
microk8s status --wait-ready
microk8s enable community registry
microk8s enable openfaas --no-auth
alias mk='microk8s kubectl'
```

### Set the environment variables

Get the microk8s VM ip:
```
multipass info microk8s-vm
```
Add the following to the "Docker Engine" section of the Docker desktop preferences:
```
{
  ...
  "insecure-registries" : ["<vm-ip>:32000"]
}
```
Then export the required environment variables:
```
export VM_IP=<vm-ip>  # Get the microk8s VM IP
export OPENFAAS_PORT=`mk get svc gateway-external -n openfaas -o jsonpath='{.spec.ports[].nodePort}'`  # Get the OpenFaaS port
export OPENFAAS_URL=$VM_IP:$OPENFAAS_PORT  # Set the OpenFaaS URL
export OPENFAAS_PREFIX=localhost:32000  # Set the OpenFaaS function image location
export FN=curly-fries  # Set the function name
```

### Create and deploy the function

Install the OpenFaaS CLI:
```
curl -sSL https://cli.openfaas.com | sudo sh
```
Fetch the function template:
```
faas-cli template store pull python3-flask
```
Create the function:
```
faas-cli new --lang python3-flask $FN
```
Build the function:
```
faas-cli build -f curly-fries.yml
```
Tag and push the function to the Docker registry:
```
docker tag localhost:32000/curly-fries:latest $VM_IP:32000/curly-fries:latest
docker push $VM_IP:32000/curly-fries:latest
```
Deploy the function!
```
faas-cli deploy -f curly-fries.yml
```

This will output a URL that you can use to curl the function, e.g:
```
curl http://192.168.64.2:31112/function/curly-fries -d 'Curly fries with cheese please'
```

### Access the OpenFaaS UI and function store

The OpenFaaS UI is visible on the URL given by the OPENFAAS_URL environment variable:
```
echo $OPENFAAS_URL  # Outputs the OpenFaaS URL that you can copy and paste into a browser
```
You can click "Deploy New Function" to explore the function store!
