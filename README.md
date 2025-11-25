# Ansible Collection - puppeteers.keycloak

This Ansible collection contains code such as roles for managing Keycloak.

# Roles

## puppeteers.keycloak.k3s

This role uses Kubernetes [Operator Lifecycle Manager](https://olm.operatorframework.io/) to set up [Keycloak Operator](https://www.keycloak.org/operator/installation/) and thus install [Keycloak](https://www.keycloak.org/) on [k3s](https://k3s.io/). You need to pass the following variables to it:

* **puppeteers_keycloak_k3s_namespace**: namespace to use for the Keycloak Kubernetes resources
* **puppeteers_keycloak_k3s_host**: Kubernetes API server URL. Example: "https://keycloak.example.org:6443".
* **puppeteers_keycloak_k3s_http_host**: hostname for the IngressRoute. Example: "keycloak.example.org".
* **puppeteers_keycloak_k3s_db_username**: PostgreSQL username for Keycloak.
* **puppeteers_keycloak_k3s_db_password**: PostgreSQL password for Keycloak.

The role works in two phases:

1. Setup phase, where Ansible copies yaml files to the to-be Keycloak host and
   applies them with "kubectl apply". This step create the namespace,
   serviceaccount, role and role binding. This phase also generates a
   short-lived service account token which is used for authentication in the
   Keycloak configuration phase.
1. Keycloak configuration phase, where the role sets up all Kubernetes resources
   required by Keycloak. This is done using *kubernetes.core.k8s* and API call
   with the service account token created in the first phase.

# License

This project is licensed under the BSD-2-Clause license. See
[LICENSE](LICENSE).
