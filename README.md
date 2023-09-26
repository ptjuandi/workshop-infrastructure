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
- Then apply
```shell
kubectl apply -f kubernetes/001_postgress-secret.yaml

kubectl apply -f kubernetes/002_postgres-db.yaml

kubectl apply -f kubernetes/003_postgres-svc.yaml

```
- Finally set up the FHIR server that depends on the postgres
```shell
kubectl apply -f kubernetes/004_fhir-deployment.yaml
```


## Mesh deployment (Istio)

## Accesing the Mesh 

