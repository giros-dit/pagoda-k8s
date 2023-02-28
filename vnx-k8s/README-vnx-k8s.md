## Escenario de pruebas del cluster de Kubernetes

### Requisitos
Linux con VNX instalado (vnx.dit.upm.es). Receta probada sobre Ubuntu 20.04 y 22.04.

El escenario utiliza dos imágenes de VNX:
- vnx_rootfs_kvm_ubuntu64-22.04-v025.qcow2, usada para las máquinas virtuales KVM que implementan los tres nodos del cluster k8s.
- vnx_rootfs_lxc_ubuntu64-20.04-v025, usada para los contenedores auxiliares del escenario. 

Para descargarlas, ejecutar:
```bash
cd /usr/share/vnx/filesystems
vnx_download_rootfs -r vnx_rootfs_kvm_ubuntu64-22.04-v025.qcow2 -y -l
vnx_download_rootfs -r vnx_rootfs_lxc_ubuntu64-20.04-v025 -y -l
cd -
```

### Manual de usuario

- Arranque del escenario:
```bash
sudo vnx -f k8s-lab-kubeadm.xml --create
```
- Instalación del cluster:
```bash
./install-k8s    # Utilizar la opción -p para realizar una pausa entre los distintos pasos de la instalación
```
- Copiar las credenciales del cluster al host para poder acceder al cluster utilizando *kubectl*:
```bash
scp k8s-master:.kube/config ~/.kube
```

### Comprobación del funcionamiento del cluster
- Disponibilidad de nodos del cluster:
```bash
$ kubectl get nodes
NAME          STATUS   ROLES           AGE   VERSION
k8s-master    Ready    control-plane   15m   v1.26.1
k8s-worker1   Ready    <none>          13m   v1.26.1
k8s-worker2   Ready    <none>          13m   v1.26.1
```
- Estado de los pod del sistema:
```bash
$ kubectl get pods -n kube-system
NAME                                      READY   STATUS    RESTARTS   AGE
calico-kube-controllers-57b57c56f-dsqfk   1/1     Running   0          35m
calico-node-ddjjh                         1/1     Running   0          35m
calico-node-fzzq9                         1/1     Running   0          35m
calico-node-rvgcd                         1/1     Running   0          35m
coredns-787d4945fb-fvk7c                  1/1     Running   0          37m
coredns-787d4945fb-mssvq                  1/1     Running   0          37m
etcd-k8s-master                           1/1     Running   0          37m
kube-apiserver-k8s-master                 1/1     Running   0          37m
kube-controller-manager-k8s-master        1/1     Running   0          37m
kube-multus-ds-5gd4p                      1/1     Running   0          34m
kube-multus-ds-ch2ns                      1/1     Running   0          34m
kube-multus-ds-whqb5                      1/1     Running   0          34m
kube-proxy-lw54x                          1/1     Running   0          37m
kube-proxy-pvnvd                          1/1     Running   0          35m
kube-proxy-xsskr                          1/1     Running   0          36m
kube-scheduler-k8s-master                 1/1     Running   0          37m
```
### Ejemplos de despliegue de servicios
Se proporcionan junto con el escenario virtual de pruebas algunos ejemplos sencillo de despliegue de servicios. Los ejemplos se basan en el despliegue de tres servidores web nginx mediante el siguiente *Deployment*:
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-web-server-pool
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-web-server
  template:
    metadata:
      labels:
        app: nginx-web-server
    spec:
      containers:
        - image: nginx
          name: nginx-web-server
          ports:
            - containerPort: 80
              protocol: TCP
          lifecycle:
            postStart:
              exec:
                command: ["/bin/sh", "-c", "echo $( hostname ) > /usr/share/nginx/html/index.html"]
```
- Para desplegar los tres servidores web: 
```bash
kubectl apply -f examples/nginx-web-server.yaml 
```
- Para comprobar que se han desplegado correctamente 
```bash
$ kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
nginx-web-server-pool-6867b54ccc-5r6qv   1/1     Running   0          80m
nginx-web-server-pool-6867b54ccc-fx79w   1/1     Running   0          80m
nginx-web-server-pool-6867b54ccc-k78kp   1/1     Running   0          80m
```

#### Despliegue de un servicio de tipo NodePort

- Despliegue del servicio NodePort para acceder a los servidores:
```bash
kubectl apply -f examples/nginx-service-nodeport.yaml
```
- En este caso, el servicio estará accesible en el puerto 3000 Para comprobar 

### Referencias

Referencias:
- How to Install Kubernetes Cluster on Ubuntu 22.04. https://www.linuxtechi.com/install-kubernetes-on-ubuntu-22-04/
- https://sangvhh.net/exposing-kubernetes-services-with-metallb-and-nginx-ingress-controller/
