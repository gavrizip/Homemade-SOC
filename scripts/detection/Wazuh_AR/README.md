
La nomenclatura utilizada para los scripts de Active Response de Wazuh está vinculada directamente con las reglas de detección de la mencionada herramienta. Aquí tenemos los ficheros más relevantes para estas tareas de respuesta activa.

- Active-Response Scripts: `/var/ossec/active-response/bin/script.sh` (AGENTE)

- Reglas: `/var/ossec/etc/rules/local_rules.xml`

- Configuración: `/var/ossec/etc/ossec.conf`
	1. Define scripts ejecutables guardados en `/var/ossec/active-response/bin`
	2. Define bajo qué reglas se ejecutan esos scripts almacenadas en `/var/ossec/etc/rules/local_rules.xml`

Como deberíamos saber, los scripts deben alojarse en la carpeta correspondiente del propio agente. No en el servidor, de lo contrario, no será posible su ejecución. En segundo lugar, las reglas deben estar escritas en `xml` dentro de la carpeta especificada más arriba.  Por último, el fichero de configuración debe contener primero el `<command>` para definir qué scripts son ejecutables y, tras ello, debe encontrarse la sección de Active Response.


___
## Flujo de trabajo de Active Response


Se ejecuta un script malicioso
    ↓
auditd captura EXECVE con a1="./linpeas.sh" → key=cmd_exec
    ↓
Wazuh matchea regla 100101 (audit.execve.a1 contiene "linpeas")
    ↓
Active response ejecuta kill-linpeas.sh en el agente
    ↓
Script extrae PID del JSON y hace kill -9
    ↓
Terminado (killed)


Para comprender mejor la manera en la que funciona este módulo de Active Response, por favor siga la [referencia](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html#active-response) a la documentación oficial de Wazuh. 


Here are some of the Active Response Custom Scripts I used to do some specific action on users endpoints:

- Kill a malicious process
- Isolate devices
- Send an email/Slack alert when an specific IP is connected


