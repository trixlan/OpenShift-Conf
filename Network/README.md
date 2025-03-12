# Redes en OpenShift

## Network Policies

> **By default, all Pods in a project are accessible from all other Pods and network endpoints**
>> To isolate one or more Pods in a project, you can create NetworkPolicy objects in that project to indicate the allowed incoming connections.

### Ejercicio mostrar como se pueden aplicar Network policies

- Creamos 3 proyectos
```shell
[localhost ~]$ oc new-project proj-a --display-name="Project A"
[localhost ~]$ oc new-project proj-b --display-name="Project B"
[localhost ~]$ oc new-project proj-c --display-name="Project C"
```
- Etiquetamos los namespace para posteriormente aplicarle Network policies
```shell
oc label namespace proj-a name=proj-a
oc label namespace proj-b name=proj-b
oc label namespace proj-c name=proj-c
```
- Revisamos las etiquetas
```shell
oc get projects proj-a proj-b proj-c --show-labels
```
- Creamos una aplicacion
```shell
[localhost ~]$ oc project proj-c
[localhost ~]$ oc new-app quay.io/bkozdemb/hello --name="hello"
```
> Se podria crear otra app
- Podemos crear un pod con Fedora o una VM
```shell
[localhost ~]$ oc run client --image=fedora --command -- tail -f /dev/null
pod/client created
```
- Creamos el cliente de fedora en las otras los otros proyectos
```shell
[localhost ~]$ oc project proj-b
[localhost ~]$ oc run client  --image=fedora --command -- tail -f /dev/null
[localhost ~]$ oc project proj-a
[localhost ~]$ oc run client  --image=fedora --command -- tail -f /dev/null
```
- Verificamos los labels, para aplicar las politicas
```shell
oc get pods --show-labels
```
- Desde un proyecto diferente al project-c podemos validar la comunicacion
```shell
oc rsh ${POD}
#Inside the pod you have to execute:
curl -v hello.proj-c:8080
```
- Desde el project-c agregamos la Network Policie
```yaml
# Deny other namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-other-namespaces
  namespace: project-c
spec
  podSelector: null
  ingress:
    - from:
      - podSelector: {}

# Allow trafic from all Pods in a particular namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-allow-production
  namespace: project-c
spec
  podSelector:
    matchLabels:
      deployment: hello
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            name: project-a
```
- Desde un proyecto diferente al project-c podemos validar la comunicacion
```shell
oc rsh ${POD}
#Inside the pod you have to execute:
curl -v hello.proj-c:8080
```