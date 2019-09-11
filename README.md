# mysql-operator

Manage your Mysql databases and users in the whole new way and deploy them along with your app!

## Why I created this operator?

I manage multiple Wordpress websites and it was always an additional work to manage Mysql databases for them.
Of course I automated management using Ansible playbooks, but nevertheless instead of deploying just Kubernetes resources from the template I had to edit inventory file, run playbook to provision databases and users and then get the password and put it in Kubernetes secret. A lot of boring work.

## How it works?

This operator provides Custom Resource Definitions that allow you to manage databases and users for Mysql clusters.

First, we need to define a cluster (with secret for string password) - the cluster tells mysql-operator how to connect to Mysql host for management of databases and users:

```
apiVersion: mysql.operator.luktom.net/v1
kind: MysqlCluster
metadata:
  name: example-mysqlcluster
spec:
  host: mysql.test.luktom.net
  username: administrator
  secretRef:
    name: example-mysqlcluster
    namespace: default
---
apiVersion: v1
stringData:
  password: TopSecretPasswordToDatabase
kind: Secret
metadata:
  name: example-mysqlcluster
type: Opaque
```

Keep in mind that the user specified above has to have permissions to create/drop database and to manage users.

Next, we can provision a database:

```
apiVersion: mysql.operator.luktom.net/v1
kind: MysqlDatabase
metadata:
  name: example-mysqldatabase
spec:
  dropOnDelete: false
  clusterRef:
    name: example-mysqlcluster
```

The `name` of the resource maps to the name of the database.
`clusterRef` refers to the above defined MysqlCluster object with connection details.
`dropOnDelete` tells operator if it should drop the database when CRD is deleted from Kubernetes.

After creating above object in Kubernetes you should see the database provsioned.

The same principles apply to the user:

```
apiVersion: mysql.operator.luktom.net/v1
kind: MysqlUser
metadata:
  name: example-mysqluser
spec:
  clusterRef:
    name: hosting
  privileges: ""
```

`name` is mapped to username to create, `clusterRef` refers to the cluster to use, and `privileges` allows to specifiy all privileges user should have after creation.
The user is always dropped when CRD is deleted from Kubernetes.

As you can see, there's no way to specify a password for the user. This is because the password is randomly generated and saved in a new Kubernetes secret called `mysql-user-example-mysqluser` (`mysql-user-` prefix + `name` from metadata).

This way, you can use a secret directly in your deployment, like that:

```
...
- name: MYSQL_PASSWORD
  valueFrom:
      secretKeyRef:
        key: password
        name: "mysql-user-example-mysqluser"
...
```

Keep in mind that `MysqlUser` and `MysqlDatabase` are namespace-scoped CRD, `MysqlCluster` is cluster-scoped, so you can refer to it from any namespace.

## Installation

```
git clone https://github.com/tomaszkiewicz/mysql-operator
cd mysql-operator/deploy
kustomize build | kubectl apply -f -
```