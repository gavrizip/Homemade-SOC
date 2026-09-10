

Wazuh:

En la sección de Endpoint security hay una sección que se llama "Configuration Asssessment", en la que aparece un dashboard en el que se muestra información relacionada con la configuración del Endpoint seleccionado, en este caso, estamos viendo el de un Rocky Linux básico, solamente se le ha instalado el agente y activado el `auditd` para hacer pipelines con el Active Response. Por ejemplo, que una determinada secuencia de acciones en un dispositivo final, active una alarma, que, a su vez, detone un script alojado en ese mismo endpoint para ejecutar una acción defensiva instantánea. En este caso, la velocidad es determinante, por ello, los scripts y el lenguaje de programación asociado debe ser el indicado para esta misma tarea.

En lo que se refiere a esta sección, de "configuration Assessment", puede resultar bien productiva en cuanto a la posibilidad de crear vídeos en torno a ello, sobre todo por el tema de la escalada de privilegios en un equipo local.


En el panel derecho, encima de la lista, nos muestran varios datos interesantes, por ejemplo, las configuraciones aprobadas por la herramienta, las que han fallado ese estándar de calidad, las que no son aplicables para esta ocasión y el score final que, como vemos, es de un 33%. Suspendido.

Por ello, resulta de nuestro interés, sacar vulnerabilidades relacionadas con estas recomendaciones de configuración para demostrar que son importantes a la hora de mantener tu empresa segura.

## La idea

Por cada configuración puede explicarse una o varias vulnerabilidades y ejecutarlas en un mismo vídeo, para dar a conocer de lo que podría aprovecharse un atacante si se encuentra con una mala configuración en el propio endpoint. Por otro lado, dar a conocer las medidas de mitigación podrían aplicarse para que ese ataque no ocurra y, acto seguido, intentar ejecutar la misma cadena de explotación previamente exitosa para demostrar que ahora no es explotable gracias a la configuración que hemos realizado. 

Esta última fase podría automatizarse para no hacer la explicación excesivamente larga. Podría hacerse con Python o el lenguaje de programación más adecuado para cada tarea.

