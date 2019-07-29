# ocs-postgres
Deploying postgres on openshift, to run pgbench through ripsaw


## Deploying postgres

This deployment assumes that namespace `my-ripsaw` has already been created.

If it's not created please do so by `kubectl create namespace ripsaw`

First step is to clone the repository and it can be done as follows:

```
git clone https://github.com/aakarshg/ocs-postgres.git
cd ocs-postgres
```

Now that we've cloned the repo, we can start with creating objects required for
a postgres deployment on k8s.

Step 1: Creating a service:
```
kubectl apply -f service.yaml
```

Step 2: Creating a configmap with the vars
The configmap contains useful env variables such as database name, user and password.
You can change/add more vars as deemed necessary before creating the configmap.

```
kubectl apply -f configmap.yaml
```

Step 3: Creating the statefulset
Take a look at the `sfset.yaml` and you'll notice the commented out lines for the storage,
if you've a specific storage class, please update accordingly, as well as the storage size

```
kubectl apply -f sfset.yaml
```

Once the steps are followed, you should see the postgres pod up and running as follows:

```
[root@marquez ocs_pgbench]# kubectl get pods -n my-ripsaw -w
NAME         READY   STATUS    RESTARTS   AGE
postgres-0   1/1     Running   0          30m
```

## Interacting with postgres

We have placed the postgres statefulset behind the service name `postgres`, so we can
interact with it directly in the same namespace of `my-ripsaw` using short address of `postgres-0.postgres`

If interacting from a resource in another namespace then we'll need to use the FQDN, which would be:
`postgres-0.postgres.my-ripsaw.svc.cluster.local`

### Updating the cr to run pgbench using ripsaw

The cr for ripsaw to run pgbench against the above database would look like following:
```yaml
apiVersion: ripsaw.cloudbulldozer.io/v1alpha1
kind: Benchmark
metadata:
  name: pgbench-benchmark
  namespace: my-ripsaw
spec:
  workload:
    name: "pgbench"
    args:
      timeout: 5
      clients:
        - 1
        - 2
      threads: 1
      transactions: 10
      cmd_flags: ''
      init_cmd_flags: ''
      scaling_factor: 1
      samples: 2
      databases:
        - host: postgres-0.postgres # assuming that postgres was deployed in same namespace as ripsaw i.e. my-ripsaw
          user: test # Following the values from the configmap
          password: test
          db_name: testdb

```
