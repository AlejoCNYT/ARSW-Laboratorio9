
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)

![imagen](https://github.com/user-attachments/assets/05d70220-44eb-401f-b324-88845fbe15cc)

![imagen](https://github.com/user-attachments/assets/4ff0d76e-1ab4-48d3-b359-2a4d6f0638d1)

![imagen](https://github.com/user-attachments/assets/7a388a8a-ed44-47c2-b75d-dc5fae7593b3)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

   ![imagen](https://github.com/user-attachments/assets/92f67487-cf81-4419-b7da-033ea2850be4)

   ![imagen](https://github.com/user-attachments/assets/bff3cc07-2f30-4057-8999-fbcced094c2a)
   
4. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/). ![imagen](https://github.com/user-attachments/assets/2f2c3caa-a8a9-4d96-8656-4bb9bf5ed850) 

5. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

   ![imagen](https://github.com/user-attachments/assets/5408a4d0-0dfc-4d35-813f-b2cb2f971d19)

   ![imagen](https://github.com/user-attachments/assets/dcdb166e-6e9b-4005-b1f1-a106da16aaf7)
   
7. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

   ![imagen](https://github.com/user-attachments/assets/f8876f9b-68ec-4e46-a4cf-11e168da1a1e)

9. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

![imagen](https://github.com/user-attachments/assets/996195e0-53b5-416d-8d23-5049985c7688)

![imagen](https://github.com/user-attachments/assets/ef2321b1-869f-4d6f-aa4e-143ebfc2a882)

![imagen](https://github.com/user-attachments/assets/b6741d60-4a3f-4f5b-9211-c102ca9e369c)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    * 1010000
    * 1020000
    * 1030000
    * 1040000
    * 1050000
    * 1060000
    * 1070000
    * 1080000
    * 1090000    

   ![imagen](https://github.com/user-attachments/assets/dce2e00a-c8e4-41e1-8e8b-c7dd28a7e980)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

   ![Imágen 2](images/part1/part1-vm-cpu.png)

   ![imagen](https://github.com/user-attachments/assets/365cb8ac-d6f4-4932-bd53-2f71baacbd84)

10. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
   ![imagen](https://github.com/user-attachments/assets/070a1b0b-af51-4213-9b0e-d5e392fb416c)
      
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
      ![imagen](https://github.com/user-attachments/assets/abb8949c-f636-46cc-b350-54e7ec2b3846)

    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
      ![imagen](https://github.com/user-attachments/assets/bfa6667b-06c7-4c1b-a82a-f3585c84a238)

      El cual contiene

      ![imagen](https://github.com/user-attachments/assets/a16efb28-5674-444a-9b76-a8f7eddec6b2)

      
      
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```
    ![imagen](https://github.com/user-attachments/assets/5eb2bc70-84e3-42dd-aa3f-1c0a142ea8b4)

11. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

![imagen](https://github.com/user-attachments/assets/3b3c512c-9676-42ef-a373-057466ece67c)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9. ![imagen](https://github.com/user-attachments/assets/a510361d-7e8b-4ed8-9058-a7f07f4039a4)  ![imagen](https://github.com/user-attachments/assets/e20bd317-8fff-4971-a97f-a1f696729d33) ![imagen](https://github.com/user-attachments/assets/25aff970-4798-477b-9e48-f36ccf31251f) ![imagen] (https://github.com/user-attachments/assets/37668bec-23c9-4ba4-8932-982dad3fc573) [imagen](https://github.com/user-attachments/assets/c3b846ca-5238-4ed2-a8fd-a090047aada6) ! [imagen](https://github.com/user-attachments/assets/b9076c6a-1301-4482-ab1f-0101fd2e62c8) ![imagen](https://github.com/user-attachments/assets/5b7e65d9-601e-4dd1-ba45-95b0267fd27e)
 ![imagen](https://github.com/user-attachments/assets/4ba56ca9-6df5-4e25- a9f5-8c0070494bbe) ![imagen](https://github.com/user-attachments/assets/506f8ea9-0bed-49b6-8d98-d2544fdd4ca9)
![imagen](https://github.com/user-attachments/assets/e6ae35d4-be35-4332-bfca-3842263ad779)

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM? ![imagen](https://github.com/user-attachments/assets/bf03a6cd-53e5-4c39-b329-b3e2655460be) 
Azure genera los siguientes recursos, en principio:
   - Vertical-Scalability
   - Scalabilit-y_lab
   - Public key
   - Ubuntu Server
3. ¿Brevemente describa para qué sirve cada recurso?
   - Vertical-Scalability: Permite aumentar o diminuir recursos sin cambiar su número. Los recursos modificados pueden ser RAM, CPU o almacenamiento.
   - Scalabilit-y_lab: Entornos de prueba para experimentar configuraciones de escalabilidad.
   - Public key: Enclave para permitir acceso público.
   - Imagen: Imagen de despliegue de contenedores.
5. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?
   - La aplicacion se cae por pérdida de la conexión. El puerto de entrada se crea para permitir el tráfico de datos por el puerto 3000.
7. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

   - La aplicación tarda por el recorido iterativo que realiza sobre todos los recursos. ![imagen](https://github.com/user-attachments/assets/dce2e00a-c8e4-41e1-8e8b-c7dd28a7e980)
   ![imagen](https://github.com/user-attachments/assets/506f8ea9-0bed-49b6-8d98-d2544fdd4ca9)
   ![imagen](https://github.com/user-attachments/assets/e6ae35d4-be35-4332-bfca-3842263ad779)     
   
9. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.
![imagen](https://github.com/user-attachments/assets/0c78db02-3032-48a8-93bb-3fbcc4bdd9ce)
- La función consume gran cantidad de recursos debido al recorrido iterativo sobre todos los números que debe recorrer.
    
11. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
      ![imagen](https://github.com/user-attachments/assets/7d9a9b03-32e1-40f2-980a-cc204c9a872e) Antes del escalamiento se mostraban 4 vulnerabilidades. Posteriormente, un mejor uso de los recursos y, configuración elimina estos fallos.

    * Si hubo fallos documentelos y explique.
   Errores asociados al cierre de servidores (ECONNRESET), proxys y pérdidas de conexión principalmente. Éstos se pueden deber a diferentes causas como intermediarios, climáticos, de escalamiento, etc.
      
12. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?
   - B1ls: Tiene un tamaño _burstable_ de 1 VCPU y 0.5 GiB RAM, el cual es ideal para cargas ligeras y esporádicas. El límite de créditos es bajo,, lo que genera _throttling_ de carga sostenida.      
   - B2ms: Tiene 2 vCPU y 8 GiB RAM, esto lo hace ideal para cargas moderadas y constantes. Su capacidad base e de 30% mayor a la anterior (40%) y tiene créditos de CPU que mejoran el rendimiento bajo carga.
    
14. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?
   - No necesariamente, esto se debe a baja eficiencia en el software. Incluso, la caída de VM durante el escalamiento genera la caída del servicio.
      ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
     - No escala automáticamente ante reinicios de carga, lo cual significa reinicio manual tras el escalamniento (downtime).
    
16. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
   - La aplicación debe reiniciarse y el costo por hora aumenta cerca de 35%.
    
18. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
   - Sí, pero esto fue limitado: CPU -**30**% y tiempo -**40**%. Esto se debe al uso de concurrencia en el procesamiento.
    
20. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

    - El rendimiento mejoró porcentualmente (más peticiones exitosas), pero la CPU alcanzó cerca de 90%, mostrando que el escalamiento vertical tiene un techo.

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```
![imagen](https://github.com/user-attachments/assets/36412b39-8acc-47c2-a7ba-24a3aa4d3c2d)

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```
![imagen](https://github.com/user-attachments/assets/9488c7f8-efa4-4af5-b4b8-f639e45c2db5)
![imagen](https://github.com/user-attachments/assets/21b76654-7463-4ed5-a01e-71e71efb7204)

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

Las imagenes con balanceo de carga procesan mejor las peticiones, generando mejores resultados.
**VM1**![imagen](https://github.com/user-attachments/assets/2bf2aacf-fcbb-4611-9c3e-27f10758a3dc) ![imagen](https://github.com/user-attachments/assets/d5e6cc31-3932-45be-9d76-6304bb9b3cea) **VM2** ![imagen](https://github.com/user-attachments/assets/aa6716a4-5565-498f-92cb-731bc64748ea)
![imagen](https://github.com/user-attachments/assets/ce064adf-28dc-4c6d-a646-321726757ab9)

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
 **Azure Load Balancer** (Capa 4) Permite balanceo básico de tráfico de red, ideal para apps no HTTP y, soporta alta disponibilidad con IP pública. 
  **Aplicatión Gateway** (Capa 7) Ofrece enrutamiento de URL, Cookies y SSL Offiline. Es ideal para aplicaciones Web.
  **Azure Front Door** (Capa 7) Da Balanceo Global, Protección DDOS y acelera contenido caché.
  **Traffic Manager** (Capa DNS) Balanceo DNS y no maneja tráfico directo, sólo redirije.
  
* ¿Cuál es el propósito del *Backend Pool*?
   - Es un grupo de recursos que reciben trafico balanceado. Tiene como propósito distribuir la carga entre múltiples unestancias y, asegurar alta disponibilidad.
  
* ¿Cuál es el propósito del *Health Probe*?
   - Es el mecanismo de verificación de estado de las instancias del _Backend Pool_. Cumple el própósito de evitar tráfico no saludable y detectar fallos. También soporta protocolos TCP, HTTP y HTTPS.
  
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
   Éste define cómo distribuír el tráfico. Los tipos de sesión persistente son **ninguna** (petición diferente para el backend con mejor balanceo pero sin estado), **Ip de cliente** (Mismo backend para cliente, es útil para aplicaciones con estado pero puede variar) y **Cookie de aplicación**. (persistencia de cookies). 
  
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
   - **Virtual Network** (VNet): Red privada para conectar recursos, en Azure (VMs, LB, etc.).

    **Subnet**: Segmento lógico de una Virtual Network (VNet) (ej: frontend-subnet, backend-subnet).

    **Address Space**: Rango IP global de la Virtual Network (ej: 10.0.0.0/16).

    **Address Range**: Subconjunto del Address Space asignado a una red (ej: 10.0.1.0/24).
  
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
  **Availability Zone** (AZs): Son centros de datos separados en una región. Se requieren 3 para sostener la tolerancia de fallos.
  **IP Zone-redundant**: Se aplica en múltiples AZs y garantiza disponibilidad bajo fallas, inclusive.
  
* ¿Cuál es el propósito del *Network Security Group*?
- Es un _firewall_ de control de tráfico saliente y entrante.Comúnmente permite SSH (puerto 22), HTTP (puerto 80), o personalizados (ej: 3000 para FibonacciApp).
  
* Informe de newman 1 (Punto 2)
![imagen](https://github.com/user-attachments/assets/25d2ceec-66ec-4641-936f-ad8970c7caca)

* Presente el Diagrama de Despliegue de la solución.




