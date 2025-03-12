# OpenShift Role Based Acces Control

## Authentication and Authorization in OpenShift
Within the context of security, two of the fundamental terms are Authentication and Authorization. Let’s break down the differences and how they apply to OpenShift.

- **Authentication** - Verifying the identity of a user

- **Authorization** - Verifying access to a resource

## OpenShift Authentication
A User is an entity that can make requests against the OpenShift API. They are broken out in the following types:

- **Regular users** - Typical individuals that access the cluster

- **System users** - For use by infrastructure related assets, such as nodes

- **Service Accounts** - Special system users that are associated with Projects/Namespaces. These users are typically used to run pods or access the cluster within external systems.

Multiple users can then be organized into Groups to better manage policies within the cluster.

Authentication against the OpenShift API is accomplished using one of the following methods:

- **OAuth Access Tokens** - Asset for communicating with the OpenShift API once authenticated.

- **X.509 Client Certificates** - Use of certificates to authenticate against the API

## OpenShift OAuth Server and Identity Providers
OpenShift contains an included OAuth server for allowing users to obtain access tokens for communicating with the API. The OAuth server can integrate with a number of identity providers that stores information about users. Common examples are LDAP, HTPasswd and OpenID Connect (OIDC).

## OpenShift Role Based Access Control (RBAC)
RBAC is used to determine whether a user is allowed to perform a given action. Authorization is managed using the following:

- **Rules** - Sets of permitted verbs on a set of objects. For example, whether a user or service account can create pods

- **Roles** - Collections of rules.

- **Bindings** - Associations between users and/or groups with a role.

Roles and bindings can be created at one of the following scopes:

- **Local** - Scoped to a given project

- **Cluster** - Applicable across all projects

To simplify access within the platform, several default cluster roles are automatically configured:

- **admin** - rights to view any resource in the project and modify any resource in the project except for quota.

- **basic-user** - A user that can get basic information about projects and users.

- **cluster-admin** - A super-user that can perform any action in any project. When bound to a user with a local binding, they have full control over quota and every action on every resource in the project.

- **cluster-status** - A user that can get basic cluster status information.

- **edit** - A user that can modify most objects in a project but does not have the power to view or modify roles or bindings.

- **self-provisioner** - A user that can create their own projects.

- **view** - A user who cannot make any modifications, but can see most objects in a project. They cannot view or modify roles or bindings.

### Ejercicio para mostrar el uso del RBAC en un namespace especifico

- Creamos un proyecto y lo seleccionamos
```shell
$ oc create-project rbac-labs
$ oc project rbac-labs
```
- Desplegamos una app dentro del proyecto
```shell
$ oc get pods
```
- Podemos crear roles tomando en cuenta dos consideraciones
  - Resources
  - Verb
```shell
$ oc create role pod-lister --verb=list --resource=pods
```
- Una vez creado podemos revisarlo
```shell
$ oc get role pod-lister -o yaml 
```
- Ahora podemos asociar el role a un serviceaccount
```shell
oc create rolebinding pod-listers --role=pod-lister --serviceaccount=rbac-lab:default

# --serviceacount = <namespace>:<serviceaccount>
```
- Podemos revisar el contenido del rolebinding
```shell
$ oc get rolebinding pod-listers -o yaml
```
- Una vez creado el rolebinding podemos validar que el serviceaccount default puede **listar** los **pods**

### Ejercicio para mostrar el uso del RBAC en todos los namespaces

- Creamos un clusterrole
```shell
$ oc create clusterrole namespace-lister --verb=list --resource=namespace
```
- Ahora podemos crear el ClusterRoleBing para asociar namespace-lister con el ServiceAccount de default
```shell
$ oc create clusterrolebinding namespace-listers --clusterrole=namespace-lister --serviceaccount=rbac-lab:default
```
- Con este ClusterRoleBinding podriamos listar todos los namespaces con el ServiceAccount

### Api Groups

- Para obtener todos los ApiGroups
```shell
$ oc api-resources
```
- Podemos validar si el usuario tiene acceso a los recursos de OpenShift API
```shell
$ oc auth can-i list users
```
- Podemos usar la Impersonation para determinar si otra cuenta puede hacer alguna accion
```shell
$ oc auth can-i list users --as=system:serviceaccount:rbac-lab:default
```

### Crearemos una Politica para recursos fuera del Core API

- Crearemos una politica que active la consulta a la cantidad de usuarios
- Lo primero es determinar el API group donde los usuario son parte
```shell
$ oc api-resources | grep users
```
- Ya que conocemos el API group crearemos el ClusterRole
```shell
$ oc create clusterrole user-lister --verb=list --resource=users.user.openshift.io
# -resource=<resource name>.<API group>
```
- Por ultimo creamos el ClusterRoleBinding para garantizar el acceso a la cuenta
```shell
$ oc create clusterrolebinding user-listers --clusterrole=user-lister --serviceaccount=rbac-lab:default
```
- Si verificamos ahora si puede listar usuarios lo podria hacer