# aws-iam-roles-ec2-s3
En este tutorial mostraré los pasos para crear un rol EC2-S3.
El objetivo es que desde una instancia de EC2 se pueda obtener un archivo zip almacenado en un bucket S3. Los permisos del bucket no son públicos y solo la instancia EC2 podrá obtener (get)  el archivo .zip

## Qué son los roles IAM 
- Los roles nos permiten asignar políticas de permisos a una identidad de AWS. 
- Un rol de IAM es similar a un usuario de IAM, sin embargo, en lugar de asociarse exclusivamente a una persona, la intención es que cualquier usuario o servicio pueda asumir un rol que necesite. 
- Un rol no tiene asociadas credenciales a largo plazo, cuando se asume un rol, este proporciona credenciales de seguridad temporales para la sesión de rol.

## Prerequisitos
1. Contar con un usuario en AWS con permisos para acceder a los servicios IAM, EC2 y S3 
2. Crear un bucket de S3 con los permisos de default y subir un archivo .zip 
3. Crear una instancia EC2 y obtener el par de llaves (key pair) que se utilizarán para entrar a través de ssh a la instancia.

Para este ejemplo suponer 
1. El nombre del bucket es **repositorio-maestro-12345**
2. El nombre del archivo es **misitiodeweb.zip**

## Pruebas

Prueba 1. 
Utilizando el par de llaves que se generaron al crear la instancia EC2  accede a ella a través de ssh e intenta traer el archivo guardado en el *bucket*. 

```
wget https://repositorio-maestro-12345.s3.amazonaws.com/misitiodeweb.zip
```
:no_entry: marcará un error ya que el bucket no tiene permisos públicos para permitir la lectura del archivo 

Prueba 2. 
Desde la instancia, intentar traer el archivo via aws cli 
```
aws s3 ls 
Unable to locate credentials
```
:no_entry: marcará un error: *Unable to locate credentials*.
debido a que no contamos con las llaves para configurar aws configure y por seguridad no utilizaremos las llaves de acceso 

# Pasos para crear un rol

### 1 Crear el rol para que EC2 puede acceder a la información de S3 

 - [X] En la consola selecciona el servicio **IAM** 
 - [X] En el menú de lado izquierdo selecciona la opción **\<roles>**
 - [X] Selecciona el botón **\<Create role>**
 - [X] Selecciona la identidad *Trusted Identity*=  **EC2**
 - [X] Selecciona el permiso **AmazonS3FullAccess**
 - [X] Escribe en el área de texto el nombre **ec2-s3-accesototal**  Este será el nombre del rol

### 2 Adjuntar el rol a la instancia 

 - [X] En la consola selecciona el servicio EC2
 - [X] Selecciona la instancia
 - [X] En el menú seleccionar Attach/Replace IAM Role adicionar el rol creado ec2-s3-accesototal

## Prueba 

Entrar a la instancia vía ssh
```
aws s3 ls
aws s3 mv s3://repositorio-maestro-12345.s3.amazonaws.com/misitiodeweb.zip .
ls
```
:white_check_mark: Debido al rol asignado si fue posible traer el archivo misitiodeweb.zip   

# Conclusión

Es muy común utilizar el servicio de almacenamiento S3 para guardar archivos que después se utilizarán en las instancias. Cuando se comienza a utilizar AWS, vemos que los principiantes dejan todos los permisos públicos en el *bucket* para su fácil acceso, pero si esto lo dejamos así en producción, corremos riesgos de seguridad. Es posible también dejar las **claves de acceso** dentro de la instancia, pero éstas pueden cambiar en el tiempo y  si entra algún hacker podrá no solo entrar a S3, obtendrá más accesos y tenderemos graves problemas de seguridad.  Por lo tanto, la mejor opción es asignar un rol, este puede quedar tan restringido que únicamente una instancia pueda acceder a un *bucket* en específico.


:raised_hands: @professormartha
