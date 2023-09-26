# WORKSHOP INSTRUCUTIONS

## Table of contents

- [WORKSHOP INSTRUCUTIONS](#workshop-instrucutions)
    - [Before you start](#before-you-start)
    - [Raw deployment FHIR + DB](#raw-deployment-fhir--db)
    - [Mesh deployment (Istio)](#mesh-deployment-istio)
    - [Accessing the Mesh](#accessing-the-mesh)

## Before you start

You must have access to a `kubernetes` cluster and `kubectl`

## Raw deployment FHIR + DB

In orther to deploy the raw enviroment without using Istio you must follow the next steps:

- Set up the database with the secrets, changing them with a proper password 

```shell 
# The user/password must be converted to base64 

echo -n "SuperSecurePass1234!" | base64 
```
- Then apply the following yaml's
```shell
kubectl apply -f kubernetes/mesh-deployment/001_postgress-secret.yaml

kubectl apply -f kubernetes/mesh-deployment/002_postgres-db.yaml

kubectl apply -f kubernetes/mesh-deployment/003_postgres-svc.yaml

```
- Finally set up the FHIR server that depends on the postgres
```shell
kubectl apply -f kubernetes/mesh-deployment/004_fhir-deployment.yaml
```

- With the config applied the expected output should be:

```shell
kubectl get all

# output


```


## Mesh deployment (Istio)

Now in order to set up the service mesh you must follow the installation steps in [istio.io](https://istio.io/latest/docs/setup/getting-started/#download) and get Istio.


Once installed istioctl, apply the following Operator that, automatically, injects the sidecar to the pods in the _default_ namespace.

```shell
istioctl apply -f kubernetes/raw-deployment/001_istio-operator.yaml
```

Then, in order to set up Istio's most usefull benefit, apply the prometheus and kiali observability tools.

```shell
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.19/samples/addons/prometheus.yaml

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.19/samples/addons/kiali.yaml
```




## Accesing the Mesh 

