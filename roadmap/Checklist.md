# Checklist priorizada por riesgo

**Leyenda de prioridad:** 
- 🔴 Crítico (empezar ya)
- 🟠 Alto
- 🟡 Medio
- ⚪ Bajo (una vez lo anterior esté cubierto)


---
## Top 10 transversal — lo primero que tocamos, sin importar el ámbito

|  #  | Elemento                                                                         |      Ámbito      | Por qué es lo más urgente                                                                                                        |
| :-: | -------------------------------------------------------------------------------- | :--------------: | -------------------------------------------------------------------------------------------------------------------------------- |
|  1  | 🔴 Inventario de activos (hardware + software)                                   |    Inventario    | Si no sabemos qué tenemos, no podemos proteger ni auditar nada. Es la base de todo lo demás.                                     |
|  2  | 🔴 MFA en accesos críticos (VPN, admin, correo, repositorios de código)          |       IAM        | Es el control individual con mayor ratio impacto/esfuerzo. Sin esto, una credencial filtrada es acceso directo para un atacante. |
|  3  | 🔴 Revisión y eliminación de cuentas/privilegios innecesarios                    |       IAM        | En empresas de desarrollo es habitual que acumulemos accesos "por si acaso" que nadie revoca.                                    |
|  4  | 🔴 Gestión de secretos (claves API, credenciales en código/repos)                | SDLC / Endpoints | Los secretos hardcodeados o en repositorios son nuestra vía de entrada más común en desarrollo de software.                      |
|  5  | 🔴 Segmentación de red (separar red de desarrollo, oficina, producción/clientes) |      Redes       | Desconozco la segmentación de red existente.                                                                                     |
|  6  | 🔴 Gestión de parches de sistemas operativos y software base                     | Vulnerabilidades | Sin esto, dejamos vulnerabilidades conocidas (CVE) expuestas de forma indefinida en nuestros sistemas.                           |
|  7  | 🟠 Visibilidad de endpoints (más allá del tráfico de red)                        |    Endpoints     | Hoy no tenemos logs de sistema ni EDR - un endpoint comprometido es invisible para nosotros hasta que actúa en la red.           |
|  8  | 🟠 Copia de seguridad y prueba de restauración                                   |   Continuidad    | Sin backups probados, un incidente de ransomware es catastrófico para nosotros, no solo grave.                                   |
|  9  | 🟠 Análisis de vulnerabilidades recurrente (no solo puntual)                     | Vulnerabilidades | Auditar una vez y no repetir nos da una foto falsa de nuestra seguridad actual.                                                  |
| 10  | 🟠 Política de acceso remoto y dispositivos personales (BYOD)                    | Endpoints / IAM  | Con un equipo técnico y desarrolladores, es habitual que accedamos desde equipos personales sin control centralizado.            |


---
## 1. Inventario de hardware y software

| Prioridad | Elemento                                                                                                 |
| :-------: | -------------------------------------------------------------------------------------------------------- |
|    🔴     | Inventario centralizado de todos nuestros equipos (servidores, portátiles, dispositivos de red)          |
|    🔴     | Inventario del software instalado por equipo, incluyendo las versiones exactas                           |
|    🟠     | Inventario de nuestros activos en la nube (instancias, buckets, bases de datos, contenedores)            |
|    🟠     | Identificación de "shadow IT" (herramientas SaaS que contratamos o usamos sin pasar por control central) |
|    🟡     | Clasificación de nuestros activos por criticidad y sensibilidad de la información que manejan            |
|    🟡     | Proceso de alta/baja de activos (qué hacemos cuando compramos o retiramos un equipo)                     |
|     ⚪     | Etiquetado físico y ubicación del hardware en nuestras instalaciones                                     |


---
## 2. Gestión de identidades y accesos (IAM)

| Prioridad | Elemento                                                                                                      |
| :-------: | ------------------------------------------------------------------------------------------------------------- |
|    🔴     | MFA obligatorio en todos nuestros accesos críticos (VPN, correo, repos de código, paneles de administración)  |
|    🔴     | Revisión de las cuentas que tienen privilegios de administrador (locales, de dominio, cloud)                  |
|    🔴     | Eliminación de cuentas huérfanas (ex-empleados, proveedores antiguos)                                         |
|    🟠     | Definición de nuestra política de contraseñas y despliegue de un gestor de contraseñas corporativo            |
|    🟠     | Aplicación del principio de mínimo privilegio en los accesos a nuestros repositorios y entornos de producción |
|    🟠     | Segregación de funciones (establecer quién despliega vs quién aprueba vs quién audita)                        |
|    🟡     | Centralización de accesos mediante Single Sign-On (SSO)                                                       |
|    🟡     | Revisión periódica (trimestral) de nuestros accesos y permisos vigentes                                       |
|     ⚪     | Gestión de identidades no humanas (cuentas de servicio, tokens de API, bots de CI/CD que utilizamos)          |


---
## 3. Endpoints (puestos de trabajo)

| Prioridad | Elemento                                                                                             |
| :-------: | ---------------------------------------------------------------------------------------------------- |
|    🔴     | Cifrado de disco obligatorio en todos nuestros portátiles                                            |
|    🔴     | Antivirus/EDR desplegado y con telemetría centralizada hacia nuestro equipo (hoy no lo tenemos)      |
|    🟠     | Gestión centralizada de las actualizaciones del sistema operativo en nuestros puestos                |
|    🟠     | Control y restricción de dispositivos extraíbles (USB)                                               |
|    🟠     | Establecimiento de una política clara de BYOD para cuando accedemos desde dispositivos personales    |
|    🟡     | Bloqueo de pantalla automático y políticas estrictas de sesión                                       |
|    🟡     | Control de aplicaciones permitidas en nuestros equipos (allowlisting básico)                         |
|     ⚪     | Gestión de dispositivos móviles (MDM) si accedemos al correo o documentación corporativa desde ellos |


---
## 4. Redes

| Prioridad | Elemento                                                                                                         |
| :-------: | ---------------------------------------------------------------------------------------------------------------- |
|    🔴     | Segmentación real de red: mantener nuestra red de oficina, desarrollo y producción/clientes totalmente separadas |
|    🔴     | Firewall perimetral con reglas revisadas (dejar de aplicar el "todo permitido de salida")                        |
|    🟠     | Wi-Fi corporativo aislado y separado del Wi-Fi que ofrecemos a invitados                                         |
|    🟠     | VPN con autenticación fuerte (MFA) para todo nuestro acceso remoto                                               |
|    🟠     | Monitorización de nuestro tráfico este-oeste (interno), no solo del tráfico perimetral externo                   |
|    🟡     | Deshabilitación de protocolos y servicios de red innecesarios en nuestra infraestructura (ej. SMBv1, telnet)     |
|    🟡     | Gestión activa de nuestros certificados y su renovación (tanto TLS interno como externo)                         |
|     ⚪     | Mantener actualizada la documentación de nuestra topología de red                                                |


---
## 5. Servidores e infraestructura

| Prioridad | Elemento                                                                                                          |
| :-------: | ----------------------------------------------------------------------------------------------------------------- |
|    🔴     | Hardening de nuestros servidores siguiendo una línea base (CIS Benchmarks o similar)                              |
|    🔴     | Gestión de parches con un calendario cerrado y definido (evitar parcheos ad-hoc)                                  |
|    🟠     | Centralización de logs de sistema (actualmente no los recopilamos fuera de la red local)                          |
|    🟠     | Separación estricta de nuestros entornos: desarrollo, pruebas y producción                                        |
|    🟡     | Gestión de nuestra configuración como código (para evitar cambios manuales que no podamos auditar)                |
|    🟡     | Revisión de los servicios que exponemos innecesariamente (puertos abiertos en nuestros servidores sin uso real)   |
|     ⚪     | Inventario de nuestras licencias y control del ciclo de vida del soporte (evitar quedarnos con software obsoleto) |


---
## 6. Gestión de vulnerabilidades

| Prioridad | Elemento                                                                                                        |
| :-------: | --------------------------------------------------------------------------------------------------------------- |
|    🔴     | Escaneo de vulnerabilidades programado y recurrente sobre nuestra propia infraestructura                        |
|    🔴     | Establecer un proceso claro de priorización y remediación (tenemos que cerrar el ciclo, no solo detectar)       |
|    🟠     | Escaneo de dependencias de software (analizar las librerías de terceros que metemos en nuestros desarrollos)    |
|    🟠     | Definición de un SLA interno de parcheo según la criticidad (ej. crítico en 72h, alto en 7 días)                |
|    🟡     | Integración de análisis estático de código (SAST) en nuestros pipelines de despliegue                           |
|    🟡     | Lanzamiento de análisis dinámico (DAST) sobre las aplicaciones que desarrollamos                                |
|     ⚪     | Creación de un programa de bug bounty o habilitación de un canal de reporte responsable para analistas externos |


---
## 7. Seguridad en el ciclo de desarrollo (SDLC)

Este es nuestro ámbito propio y uno de los de mayor impacto, dado que casi toda nuestra plantilla está enfocada al desarrollo.

| Prioridad | Elemento                                                                                                          |
| :-------: | ----------------------------------------------------------------------------------------------------------------- |
|    🔴     | Gestión de secretos: tolerancia cero con las credenciales o API keys incrustadas en nuestro código o repositorios |
|    🔴     | Revisión estricta de permisos en nuestros repositorios (controlar quién puede hacer merge/push a producción)      |
|    🟠     | Firma y verificación de nuestros artefactos y builds                                                              |
|    🟠     | Revisión de las dependencias de terceros que introducimos (seguridad en nuestra supply chain)                     |
|    🟡     | Securización de nuestros entornos de CI/CD, manteniendo el acceso restringido y auditado                          |
|    🟡     | Política de revisión de código (hacer obligatorio el code review por pares antes de un merge)                     |
|     ⚪     | Formación continua en desarrollo seguro (secure coding) para nuestro equipo de desarrollo                         |


---
## 8. Backup y continuidad

| Prioridad | Elemento                                                                                          |
| :-------: | ------------------------------------------------------------------------------------------------- |
|    🔴     | Copias de seguridad totalmente automatizadas de nuestros sistemas y datos críticos                |
|    🔴     | Pruebas periódicas de restauración (tenemos claro que un backup que no probamos no existe)        |
|    🟠     | Almacenamiento de copias offline o inmutables para protegernos eficazmente contra el ransomware   |
|    🟡     | Redacción y documentación de nuestro plan de continuidad de negocio                               |
|    🟡     | Desarrollo de nuestro plan de recuperación ante desastres con RTO y RPO definidos formalmente     |
|     ⚪     | Ejecución de simulacros de recuperación completos (ya sean en formato tabletop o técnicos reales) |


---
## 9. Logging, monitorización y respuesta

| Prioridad | Elemento                                                                                                              |
| :-------: | --------------------------------------------------------------------------------------------------------------------- |
|    🔴     | Centralización de logs a gran escala, yendo más allá de la red (recopilar eventos de sistema, aplicación e identidad) |
|    🟠     | Configuración de una correlación básica de eventos (nuestro paso previo necesario antes de montar un SIEM real)       |
|    🟠     | Retención de logs con tiempo suficiente para que podamos realizar investigaciones forenses si nos comprometen         |
|    🟡     | Implementación de alertas automáticas ante eventos críticos (dejar de depender de una revisión manual semanal)        |
|    🟡     | Documentar nuestro propio plan de respuesta a incidentes (establecer un mínimo viable)                                |
|     ⚪     | Integración de nuestras alertas con feeds de inteligencia de amenazas (Cyber Threat Intelligence)                     |


---
## 10. Gobierno y cumplimiento (mínimo viable, avanzando en paralelo)

| Prioridad | Elemento                                                                                                  |
| :-------: | --------------------------------------------------------------------------------------------------------- |
|    🔴     | Nombrar formalmente nuestros roles de seguridad (aunque asumamos una misma persona varios roles)          |
|    🟠     | Redactar, documentar y comunicar internamente nuestra Política de Seguridad de la Información             |
|    🟠     | Realizar nuestro análisis de riesgos inicial (partiendo de una base sencilla)                             |
|    🟡     | Documentar los procedimientos técnicos de los controles que ya hemos conseguido implementar               |
|    🟡     | Recopilar y estructurar nuestro registro de evidencias para estar listos ante la futura auditoría del ENS |
|     ⚪     | Contratar una revisión externa o preauditoría para evaluar de forma objetiva nuestro nivel de madurez     |


---
## Cómo debemos usar esta checklist

1. Tenemos que marcar primero todo lo 🔴 en todos los bloques antes de pasar a lo 🟠. Sabemos que el riesgo no se reparte por igual, así que nos interesa mitigar los críticos de todos los ámbitos antes de profundizar en las prioridades altas de un solo bloque.

2. Cada control que marquemos como completado debe dejar una evidencia mínima (una captura, un documento configurado o una exportación). Esto nos ahorrará semanas de trabajo cuando empecemos la preparación oficial para la auditoría del ENS.

3. Revisaremos esta tabla completa de manera trimestral; somos conscientes de que el estado de "cubierto" es efímero (los parches caducan, aparecen cuentas nuevas y surgen activos en la sombra que debemos controlar).

