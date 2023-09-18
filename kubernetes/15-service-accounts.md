# Service accounts and RBAC

Kubernetes service accounts are used to provide an identity for processes that run in a Pod. Service accounts exist as `ServiceAccount` objects in the API server. They are namespaced objects, so they are only visible in the same namespace where they are created.

There are four objects that play a role in the authorization process:
- `Role` or `ClusterRole` - Defines the access permissions.
- `RoleBinding` or `ClusterRoleBinding` - Binds the role to a service account.
- `ServiceAccount` - The service account that will be granted the permissions.
- `Pod` - The pod that will use the service account.

Roles and RoleBindings can be used with other objects, such as users and groups. This document focuses on the usage with service accounts.

- [Service accounts and RBAC](#service-accounts-and-rbac)
  - [Authorization process](#authorization-process)
  - [Service accounts](#service-accounts)
    - [Service account tokens](#service-account-tokens)
    - [Service account lifecycle](#service-account-lifecycle)
    - [Create service account](#create-service-account)
    - [Get service account information](#get-service-account-information)
    - [Attach service account to a pod](#attach-service-account-to-a-pod)
  - [Role-based access control (RBAC)](#role-based-access-control-rbac)
    - [Usage and scope](#usage-and-scope)
    - [Defining permissions](#defining-permissions)
    - [Role and ClusterRole](#role-and-clusterrole)
    - [RoleBinding and ClusterRoleBinding](#rolebinding-and-clusterrolebinding)
  - [References](#references)

## Authorization process

The authorization process is the following:
- 1. Create a `ServiceAccount` object. See [Service accounts](#service-accounts).
- 2. Create a `Role` or `ClusterRole` object with the desired permissions. See [Role-based access control (RBAC)](#role-based-access-control-rbac).
- 3. Create a `RoleBinding` or `ClusterRoleBinding` object to bind the role (with the permissions) to the service account. See [Role-based access control (RBAC)](#role-based-access-control-rbac).
- 4. Attach the service account to a pod. See [Attach service account to a pod](#attach-service-account-to-a-pod).

## Service accounts

Service accounts are different to user accounts, these are used to identify users. Service accounts are used to identify processes inside a Pod instead. 

Aliases: `serviceaccount` and `sa`

### Service account tokens

The token is the credential used to access the API server. Service accounts use JWT to authenticate the requests to the API server. 

Prior to version 1.24, service accounts were stored in long-lived secrets (no expiration). This was a security risk, because the tokens were valid until they were manually revoked. The recommendation is to use short-lived tokens instead.

### Service account lifecycle

- 1. Creation:
  - Each namespace has a `default` service account, which is created automatically when the namespace is created.
  - When additional service accounts are created, the API server generates new tokens for each one.
- 2. Granting permissions:
  - Different permissions are granted to the service account using [role-based access control (RBAC)](#role-based-access-control-rbac).
- 3. Usage:
  - To assign the token to a pod, the service account must be specified in the pod definition using the `.spec.serviceAccountName` property. If not specified, the `default` service account is used.
  - The token is mounted in the pod as a file at `/var/run/secrets/kubernetes.io/serviceaccount/token`. Applications can read the token from this file. 
  - To opt out of automounting the API credentials, use the `automountServiceAccountToken: false` property in the pod definition or in the service account definition.
- 4. Expiration:
  - The short-lived tokens have default expiration time of 1 hour.
  - These tokens are automatically renewed by the API server when they expire.
  - The expiration time can be changed using the `--service-account-token-ttl` flag when starting the API server.

### Create service account

Creating a service account is similar to creating other Kubernetes objects. The following example creates a service account called `dashboard-sa` in the `default` namespace:

```bash
$ kubectl create sa dashboard-sa
serviceaccount/dashboard-sa created
```

The service account can be described with a YAML file:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-sa
```

### Get service account information

Listing service accounts works like listing other objects:

```bash
$ kubectl get serviceaccounts 
NAME           SECRETS   AGE
default        0         9m4s
dashboard-sa   0         3m11s
```

### Attach service account to a pod

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-dashboard
spec:
  (...)
  template:
    spec:
      serviceAccountName: dashboard-sa
      containers:
  (...)
```

To test the service account, you could access the pod and run the `curl` command to access the API server:

```bash
$ curl https://kubernetes/api/v1/namespaces/default/pods --header "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

Without granting permissions to the service account, the request will fail with a `403 Forbidden` error. This is done with the RBAC API, as explained in the next section. 

## Role-based access control (RBAC)

If no permissions are granted to the service account, it will not be able to do much in the cluster. The permissions are granted using role-based access control (RBAC).
 
The RBAC API declares four kinds of Kubernetes object: `Role`, `ClusterRole`, `RoleBinding` and `ClusterRoleBinding`. The first two define a set of permissions, and the last two are used to bind these roles to `ServiceAccounts`.

### Usage and scope

As the name suggest, the `ClusterRole` and `ClusterRoleBinding` objects are cluster-scoped, while the `Role` and `RoleBinding` objects are namespaced:

- The `Role` grant permissions in a specific namespace.
- The `ClusterRole` object is not namespaced, so it can be used to grant permissions in the entire cluster using cluster-scoped resources (e.g. nodes), or in multiple namespaces using namespaced resources (e.g. pods).
- Consequently, the `RoleBinding` are used with `Role` objects, and the `ClusterRoleBinding` are used with `ClusterRole` objects.

### Defining permissions

Permissions are defined using the `rules` property. Each rule defines a set of resources and operations that can be performed on them.

- Resources: All kubernetes resources can be used in a role. Examples: `pods`, `deployments`, `services`, `configmaps`, etc.
- Operations: The operations are defined using the HTTP verbs. Examples: `get`, `list`, `watch`, `create`, `update`, `patch`, `delete`, `deletecollection`, etc.
- The `apiGroups` property defines the API group of the resources. The default value is `""`, which means the core API group. New cluster resource definitions (CRDs) can define their own API groups, and these can be used in the `apiGroups` property.

By default the RBAC policy denies all requests. The permissions must be individually granted using the `rules`.

### Role and ClusterRole

The `Role` and `ClusterRole` objects are similar, these are the definition files for a `Role` and a `ClusterRole`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-manager
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

The previous cluster role example defines extensive permissions for pods. 

On the other hand, the following role definition grants permissions to only list pods in the `default` namespace:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: list-pods-role
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list"]
```

### RoleBinding and ClusterRoleBinding

Following the previous examples, the following `ClusterRoleBinding` object grants the `manager-sa` service account the `pod-manager` cluster role:


```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: manager-sa-binding
roleRef:
  kind: ClusterRole
  apiGroup: rbac.authorization.k8s.io
  name: pod-manager # must match the cluster role name
subjects:
- kind: ServiceAccount
  name: manager-sa # must match the service account name
```

The following example grants the `manager-sa` service account the `list-pods-role` role in the `default` namespace:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dashboard-sa-view-binding
  namespace: default
roleRef:
  kind: Role
  apiGroup: rbac.authorization.k8s.io
  name: list-pods-role # must match the role name above
subjects:
- kind: ServiceAccount
  name: dashboard-sa
  namespace: default # must match the namespace where the service account is created
```

Alternatively, the `RoleBinding` can be created using the `kubectl create rolebinding` command:


```bash
$ kubectl create rolebinding dashboard-sa-view-binding --role=list-pods-role --serviceaccount=default:dashboard-sa
rolebinding.rbac.authorization.k8s.io/dashboard-sa-view-binding created
```

The previous command binds the `dashboard-sa` service account with the permissions to list pods in the `default` namespace.

****
## References

- [Service accounts](https://kubernetes.io/docs/concepts/security/service-accounts/)
- [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
- [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
