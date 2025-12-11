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
* **puppeteers_keycloak_k3s_http_hosts**: a list DNS names Keycloak should be accessible as: used for the IngressRoute.
* **puppeteers_keycloak_k3s_db_username**: PostgreSQL username for Keycloak.
* **puppeteers_keycloak_k3s_db_password**: PostgreSQL password for Keycloak.
* **puppeteers_keycloak_k3s_starting_version**: Version of Keycloak to install *initially*, for example 26.4.5. This gets passed to the end of the `startingCSV` parameter ("keycloak-operator.v24.6.5"). This has no effect beyond the initial Keycloak install: upgrades are handled by accepting Operator InstallPlans (see below).
* **puppeteers_keycloak_k3s_tls_secret_name**: get TLS certificate and key from this secret, for example when using [cert-manager](https://cert-manager.io) for example. If this is left undefined default Traefik certificates will be used instead.

Ingress route configurations are:

* **puppeteers_keycloak_k3s_admin_allow_ips**: IP or IP ranges (IPv4 or IPv6) from which to allow connections to the /admin. Defaults to '- 10.0.0.0/8'.
* **puppeteers_keycloak_k3s_admin_http_hosts**: hostnames which don't have restrictions on access to /admin. This would typically be an internal DNS name in IP range defined above. Defaults to and empty list.

If you wish to use an external PostgreSQL database with Keycloak you can do it
with these two variables:

* **puppeteers_keycloak_k3s_manage_postgres**: set to *false*
* **puppeteers_keycloak_k3s_db_host**: set to the hostname of the PostgreSQL server
* **puppeteers_keycloak_k3s_db_database**: set to the name of the Keycloak database

To use a custom Keycloak image:

* **puppeteers_keycloak_k3s_keycloak_image**: set to the something along the lines of `repo.example.com/keycloak:26.4.6; the default value is the same as in `keycloak/operator/src/main/resources/application.properties`, i.e. `quay.io/keycloak/keycloak:nightly`.

Make sure to follow the instructions in
[Customizing Keycloak](https://www.keycloak.org/operator/customizing-keycloak) and
[Running Keycloak in a container](https://www.keycloak.org/server/containers)
articles when building your custom image.

**NOTE**: Operator InstallPlan approval is set to *Manual*. This means that you
*need* to approve the initial InstallPlan to get Keycloak running. On the
Keycloak host do something like this:

```
$ kubectl get installplans -n keycloak
$ kubectl patch -n keycloak installplan install-xyz12 -p '{"spec":{"approved":true}}' --type merge
```

After a while Keycloak should be running.

# License

This project is licensed under the BSD-2-Clause license. See
[LICENSE](LICENSE).
