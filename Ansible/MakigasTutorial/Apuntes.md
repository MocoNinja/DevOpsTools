# Apuntes de Ansible

## Inventory

Es un archivo localizado en ```/etc/ansible/hosts/```, que contiene el *inventario global*. Este archivo contiene líneas, equivaliendo cada una a un host que se desea aprovisionar.

Las ips de las másquinas  "sencillas" se pueden poner línea a línea. Si se desea introudcir un patrón más complejo, se pueden usar rangos. Por ejemplo, si tengo 10 dominios del estilo *"javi1.es"*, en vez de introducir 10 líneas distintas, puede usarse una expresión del tipo: ```javi[1:10].es```

También pueden usarse los **grupos** de servidores. Esto se especifica poniendo las ips deseadas bajo una marca de tipo *```[nombredelgrupo]```*. Esto permite hacer un *fine tunning* de las tareas.

Si se desea, por cualquier razón, usar puntualmente otros servidores, se puede crear un archivo de hosts local (con cualquier nombre). Este fichero se especifica mediante las opciones ```-i hosts```

Como se explicará más abajo, se tiene que especificar en ocasiones la clave privada para conectarse or ssh. En vez de ejecutarla para cada comando, si se tiene un **grupo de servidores**, se puede especificar dicha clave en el *inventario*. Por ejemplo:

```
[mahHosts]
...
SERVIDORES
...
[mahHosts:vars]
ansible_ssh_private_key_file=PATH TO CLAVE
```

También es importante hablar del **usuario**. Por defecto, el usuario que se coge es el mismo que el que resultaría de ejecutar un ```whoami``` en la máquina que está corriendo las definiciones de Ansible. En el caso de este ejemplo no es necesario cambiarlo, ya que los tres nodos de mi clúster de OpenStack corren imágenes de Ubuntu, con el usuario **ubuntu**.

Para especificar un usuario distinto, se tiene que:

* Si se usa la orden inline, especificar el flag ```-u```
* Si se escribe en el archivo de configuración como en el ejemplo anterior, ```ansible_user=usuario```

## Tasks

Son las funciones que ejecuta *Ansible*, pudiéndose entender como la tarea mínima que interactúa con el servidor.
Son **declarativas**, es decir, no se especifican *comandos específicos*, sino los *estados deseados* en los que queremos dejar el servidor.

Se pueden ejecutar de distintas maneras

### Comandos *ad-hoc*

Permite lanzar comandos de Ansible sin crear ningun tipo de archivo. Por ejemplo:

```shell
ansible host -m ping
```

Donde *host* es el host en el que se desea llevar a cabo la tarea. La opción *```-m```* es la que indica que se está usando un ***módulo*** de ansible, en este caso, *ping*

Al acceder por ssh, se intenta usar la configuración de ssh del sistema, pudiéndose consultar [la documentación](https://docs.ansible.com/ansible/2.5/user_guide/intro_getting_started.html) para una explicación mas rigurosa.
En resumen, si se desea acceder con un fichero *.pem*, puede usarse el siguiente comando:

```ansible host -m ping --private-key pem```

Una lista comprensiva de módulos se encuentra en la [documentación](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html).

Es **importante** notar que si no se especifica ningún módulo con la opción ```-m```, se asume por defecto el módulo ***shell***.

Por ejemplo, el siguiente comando ejecuta  el comando ```echo``` con la opción ```"hola"```:

```ansible host -a "echo hola" --private-key pem```

El siguiente comando de ejemplo se aseguraría de que vim está instalado. Hace uso del módulo *apt* de Ansible, documentado en el enlace anterior.

```ansible host --private-key pem -m apt -a "name=vim state=present" -b ```

Notar que, debido a que la instalación de paquetes requiere permisos de *root*, hay que especficar la opción ```-b```, equivalente a la opción de Ansible *```become```, que es la encargada de que se cambie el rol de usuario. Si este paso requiere contraseña, habrá que especificar la opción ```-K````, que *prompterá* dicha credencial.

### Playbooks

Son el grupo de tareas que se desea que ejecuten sobre un inventario (o sobre un subconjunto de un inventario).

Son archivos ```.yml```

Ver los .yml de ejemplo

#### Handlers

Cuando tenemos un *playbook* como en el apartado anterior, los pasos descritos en él se ejecutan secuencialmente. A veces puede ser interesante que el paso se ejecute sólo bajo ciertas condiciones; por ejemplo, en el playbook de ejemplo, podría ser interesante que el estado del servidor fuera *reiniciado* sólo si se ha copiado un fichero que requiera reiniciar el servidor.

Esto son los *handlers*. Se describen en el *playbook* bajo la sección *handlers* en vez de *tasks*. Para que sean llamados, debe especificarse su **nombre** en la *tarea* adecuada, usando la opción **notify**, con uso ejemplo:

```yaml
notify:
  - nombre de 1 handler
  (- nombre de otro...)
```
