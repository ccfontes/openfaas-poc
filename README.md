# Introduction
Quote from [docs.openfaas.com](https://docs.openfaas.com/):
> OpenFaaS® makes it easy for developers to deploy event-driven functions and microservices to Kubernetes without repetitive, boiler-plate coding. Package your code or an existing binary in a Docker image to get a highly scalable endpoint with auto-scaling and metrics.

In this tutorial we're going to explore deploying OpenFaaS to k8s, and creating an HTTP endpoint serving HTML.

# Requirements
- Around **15 minutes** of your time
- Le Mac + Homebrew
- Docker Desktop with k8s cluster option turned on, or other k8s cluster
- Logged into a Docker Hub account

# Setup

## Install utils
```
brew install kubectl faas-cli

curl -SLsf https://dl.get-arkade.dev/ | sh
```

## Deploy OpenFaaS to k8s cluster
```sh
ark install openfaas --load-balancer false
```

## Expose gateway
Forward the gateway port:
```sh
kubectl port-forward -n openfaas svc/gateway 8080:8080 &
```

Expose gateway to `faas-cli up` command:
```sh
export OPENFAAS_URL=http://127.0.0.1:8080
```

## Login to OpenFaaS
```sh
PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
echo -n $PASSWORD | faas-cli login --username admin --password-stdin
```

# Development

## Pull down a template function
Replace `<dockerhub_id>` below, with your Docker user name, then run:
```sh
faas-cli new --lang python3 --prefix <dockerhub_id> showhtml
```

## Rename created function to default stack name
Create `stack.yml` file used to gather all function's config.
`stack.yml` is used by `faas-cli up` by default.
```sh
mv showhtml.yml stack.yml
```

## Hello world from template function
In `showhtml/handler.py` file, replace `handle` function with:
```python
def handle(req):
    return '<html><h2>Hello world</h2></html>'
```

## Deploy all functions
```sh
faas-cli up
```

## Run the function
Point your browser to:
```
http://127.0.0.1:8080/function/showhtml
```

# Wrap up

## If you liked OpenFaaS
```sh
echo 'export OPENFAAS_URL=http://127.0.0.1:8080\n' >> .zshrc
```

## If you didn't like OpenFaaS
Undeploy OpenFaaS from k8s cluster:
```sh
kubectl delete namespace openfaas
```

Uninstall utils:
```sh
brew rm faas-cli
```

# Further reading
- [Comparison between K8s Microservices and OpenFaaS](https://github.com/ccfontes/openfaas-poc/blob/main/diff_microservices_openfaas.csv)
- [Explore other POCs](https://github.com/openfaas/workshop)

