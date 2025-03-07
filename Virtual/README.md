# Maquinas virtuales en OpenShift

## Exponer una app dentro de una VM por medio de un servicio dentro de OpenShift

- Se debe crear una maquina virtual de preferencia Centos 9
- Instalar httpd
```shell
# sudo yum install httpd
```
- Iniciar el servicio de httpd
```shell
# sudo systemctl enable --now httpd
# sudo systemctl start httpd
Verificar
# sudo systemctl status httpd
```
- Agregar la etiqueta en la maquina virtual
```yaml
spec:
  template:
    metadata:
      annotations:
        vm.kubevirt.io/flavor: small
        vm.kubevirt.io/os: centos-stream9
        vm.kubevirt.io/workload: server
      creationTimestamp: null
      labels:
        env: webapp # Esta etiqueta debe estar igual en el servicio
```
- Crear el servicio
```yaml
apiVersion: v1
kind: Service
metadata:
  name: centos
  namespace: proyecto
spec:
  selector:
    env: webapp # Esta etiqueta debe estar igual la VM
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
- Crear el router