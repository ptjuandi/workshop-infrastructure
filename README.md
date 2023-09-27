# WORKSHOP INSTRUCUTIONS

## Table of contents

- [WORKSHOP INSTRUCUTIONS](#workshop-instrucutions)
    - [Before you start](#before-you-start)
        - [Test](#test)
        - [Clean up](#clean-up)
    - [Raw deployment FHIR + DB](#raw-deployment-fhir--db)
    - [Mesh deployment (Istio)](#mesh-deployment-istio)
    - [Accessing the Mesh](#accessing-the-mesh)

## Before you start

You must have access to a `kubernetes` cluster and `kubectl`

## Raw deployment FHIR + DB

In orther to deploy the raw enviroment without using Istio you must follow the next steps:

- Set up the database with the secrets, changing them with a proper password in the [001_postgress-secret.yaml](./kubernetes/raw-deployment/001_postgress-secret.yaml)

```shell 
# The user/password must be converted to base64 

echo -n "SuperSecurePass1234!" | base64 
```
- Then apply the following yaml's
```shell
kubectl apply -f kubernetes/raw-deployment/001_postgress-secret.yaml

kubectl apply -f kubernetes/raw-deployment/002_postgres-db.yaml

kubectl apply -f kubernetes/raw-deployment/003_postgres-svc.yaml

```
- Finally set up the FHIR server that depends on the postgres
```shell
kubectl apply -f kubernetes/raw-deployment/004_fhir-deployment.yaml
```

- With the config applied the expected output should be:

```shell
kubectl get all

# output

NAME                               READY   STATUS    RESTARTS   AGE
pod/fhir-server-5f4c6b6b8-hmksw    1/1     Running   0          x
pod/postgres-db-7cf44b646d-4n8kw   1/1     Running   0          x

NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/kubernetes    ClusterIP   10.96.0.1      <none>        443/TCP    x
service/postgres-db   ClusterIP   10.97.173.90   <none>        5432/TCP   x

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/fhir-server   1/1     1            1           x
deployment.apps/postgres-db   1/1     1            1           x

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/fhir-server-5f4c6b6b8    1         1         1       x
replicaset.apps/fhir-server-7c4d84f949   0         0         0       x
replicaset.apps/postgres-db-7cf44b646d   1         1         1       x


```
### Test 

Some commands to test that the services behavior.

In order to expose locally the FHIR server we need to create a service that gives access to it on a free port in our machine. If you are testing in a external cluster, exposing it through a gateway may be a better choice.

```shell
kubectl expose deployment/fhir-server --type="NodePort" --port 8080
```
Once exposed it's usefull to set the port assigned to a variable doing
```shell 
export FHIR_PORT="$(kubectl get services/fhir-server -o go-template='{{(index .spec.ports 0).nodePort}}')"
```
Then we can check if doing a curl to the Patient endpoints returns 200 (OK)
```shell
curl "http://localhost:$FHIR_PORT/fhir/Patient" -X GET -sS -o /dev/null -w "%{http_code}\n"
```

### Clean up

Before continuing to the next stage a clean up must be made so the changes on top of the previous are aplied.

```shell
kubectl delete -f kubernetes/raw-deployment/004_fhir-deployment.yaml

kubectl delete -f kubernetes/raw-deployment/002_postgres-db.yaml

kubectl delete -f kubernetes/raw-deployment/003_postgres-svc.yaml

kubectl delete -f kubernetes/raw-deployment/001_postgress-secret.yaml

kubectl delete svc fhir-server

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

