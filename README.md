# Replicacion base de datos mysql

Este archivo creara dos replicas. Las bases de datos solo seran acedidas desde el cluster. No podran ser acedidas por fuera del mismo.

Este ejemplo es valido para clustr creados de tipo Custom. Es decir sin un provedor especifico.

## PASO 1:
importar los archivos siguientes al cluster en el siguiente orden:
 ### storage.yml
 Crea los volumenes persitentes y el storage class, (Solo creara por el momento dos volumenes
 ### mysql-configmap.ymal
 Crea el config map de el servicio
 ### mysql-service
 Crea los servicios a utilizar
 ### mysql-statefulset.ymal
 Crea el workload con todos los contenedores necesarios para hacer la replicación

## PASO 2:
### Lanzar la consola de kubernets desde el cluster.
### Correr el siguiente comando para crear la base de datos

(Este paso solo se debe hacer una vez gracias a los volumenes)
 
> kubectl run mysql-client --image=mysql:5.7 -i --rm --restart=Never --\
  mysql -h mysql-0.mysql <<EOF
CREATE DATABASE authentication;
EOF 
 ### Verificar que la base de datos fue creada correctamente
 > kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
  mysql -h mysql-read -e "show databases"

## PASO 3:
Correr el microservicio a conestar con las siguiente variables de entorno

  > DBUSER	root 
  
  > DBURL	mysql 
  
  > DBPORT	3306 
  
  > DBPASSWORD	 
  
  ### Nota1: 
  la base de datos no tiene contraseña, esto no es accidental 
  ### Nota2: 
  la DBURL no es por el nombre del workload, si este se cambia sin modificar nada más, la variable debe seguir llamandose igual.


## Otros
### Más informacioń
 Ejemplo adaptado de:
https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/

### replicas
Para las crear más replicas se debe correr el siguiente comando (En este caso generaria 5 replicas)
kubectl scale statefulset mysql  --replicas=5

#### Nota 1
Note que esto no creará automaticamente los volumen persistentes, estos deben ser creados importando y adaptando el archivo pvc.ymal

#### Nota 2
Si reduce el numero de replicas debe eliminar los volumenes manualmente

Para listar los volumenes:
> kubectl get pvc -l app=mysql

para eliminar un volumen:

> kubectl delete pvc data-mysql-3

 donde data-mysql-3 es el nombre del volumen. Además debera eliminar manualmente el volumen persistente creado con el archivo pvc.ymal
