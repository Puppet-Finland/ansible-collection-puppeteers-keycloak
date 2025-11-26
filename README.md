# Ansible Collection - puppeteers.keycloak

This Ansible collection contains code such as roles for managing Keycloak.

# Roles

## puppeteers.keycloak.k3s

This role uses Kubernetes [Operator Lifecycle Manager](https://olm.operatorframework.io/) to set up [Keycloak Operator](https://www.keycloak.org/operator/installation/) and thus install [Keycloak](https://www.keycloak.org/) on [k3s](https://k3s.io/). The role works in two phases:

1. Setup phase, where Ansible copies yaml files to the to-be Keycloak host and
   applies them with "kubectl apply". This step create the namespace,
   serviceaccount, role and role binding. This phase also fetches server-ca.crt
   and generates a short-lived service account token. Both of those are used
   for authentication in the next phase.
1. Keycloak configuration phase, where the role sets up all Kubernetes resources
   required by Keycloak. This is done using the *kubernetes.core.k8s* module.

You need to pass the following variables to it:

* **puppeteers_keycloak_k3s_namespace**: namespace to use for the Keycloak Kubernetes resources
* **puppeteers_keycloak_k3s_host**: Kubernetes API server URL. Example: "https://keycloak.example.org:6443".
* **puppeteers_keycloak_k3s_http_host**: hostname for the IngressRoute. Example: "keycloak.example.org".
* **puppeteers_keycloak_k3s_db_username**: PostgreSQL username for Keycloak.
* **puppeteers_keycloak_k3s_db_password**: PostgreSQL password for Keycloak.

If you wish to use an external PostgreSQL database with Keycloak you can do it
with these two variables:

* **puppeteers_keycloak_k3s_manage_postgres**: set to *false*
* **puppeteers_keycloak_k3s_db_host**: set to the hostname of the PostgreSQL server
* **puppeteers_keycloak_k3s_db_database**: set to the name of the Keycloak database

Note that Operator InstallPlans are approval is set to *Manual*. This means
that you *need* to approve the initial InstallPlan to get Keycloak running. On
the Keycloak host do something like this:

```
$ kubectl get installplans -n keycloak
$ kubectl patch -n keycloak installplan install-xyz12 -p '{"spec":{"approved":true}}' --type merge
```

After a while Keycloak should be running.

# License

This project is licensed under the BSD-2-Clause license. See
[LICENSE](LICENSE).
