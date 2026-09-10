
# **🛡️ Home SOC Lab — Building a Security Operations Center from Scratch**

![status|86](https://img.shields.io/badge/status-active-brightgreen)

> Un laboratorio documentado, paso a paso, para construir un Security Operations Center (SOC) completo en casa: SIEM, IDS/IPS, gestión de logs, detección de amenazas y automatización.

---

## Tabla de contenidos

- [Sobre el proyecto](#-sobre-el-proyecto)
- [¿Para quién es esto?](#-para-quién-es-esto)
- [Arquitectura del laboratorio](#-arquitectura-del-laboratorio)
- [Requisitos previos](#-requisitos-previos)
- [Inicio rápido](#-inicio-rápido)
- [Estructura del proyecto](#-estructura-del-proyecto)
- [Cómo usar este proyecto](#-cómo-usar-este-proyecto)
- [Contribuir](#-contribuir)
- [Recursos y créditos](#-recursos-y-créditos)
- [Licencia](#-licencia)
- [Contacto](#-contacto)

---
## Sobre el proyecto

Este proyecto documenta, día a día, el proceso completo de construir un **SOC (Security Operations Center) funcional desde cero**, usando herramientas open source en un entorno de laboratorio casero. Nace como una bitácora de aprendizaje y experimentación, pensada para que cualquiera pueda replicar el proceso, entender las decisiones detrás de cada configuración y evitar los errores que ya se cometieron aquí.

El proyecto cubre los componentes esenciales de un SOC real:

- **SIEM** — recolección, correlación y visualización de eventos de seguridad
- **IDS/IPS** — detección y prevención de intrusiones en red
- **Gestión de logs** — centralización, normalización y retención
- **Automatización y detección de amenazas** — scripts propios para análisis y respuesta
- **Documentación y buenas prácticas** — todo lo aprendido en el camino, incluidos los fallos

---
## ¿Para quién es esto?

- Estudiantes y entusiastas de ciberseguridad que quieren aprender haciendo
- Profesionales que quieran montar un entorno de práctica (home lab) para certificarse o mantenerse al día
- Cualquiera que quiera entender cómo funciona un SOC "por dentro", más allá de la teoría

No se requiere experiencia previa avanzada, pero sí conocimientos básicos de redes, Linux y línea de comandos.

---
## Arquitectura del laboratorio

*(Añade aquí un diagrama de la arquitectura del lab: hipervisor, VMs, red virtual, flujo de logs hacia el SIEM, etc. Un diagrama vale más que mil palabras y ayuda muchísimo a quien llega nuevo.)*

Ejemplo de lo que podrías incluir:
- Diagrama de red (segmentos, VLANs)
- Máquinas virtuales y su rol (SIEM, IDS, víctima, atacante, etc.)
- Flujo de datos: fuente de logs → agente → SIEM → alertas

---
## Requisitos previos

Antes de empezar, asegúrate de tener:

- [ ] Un equipo con al menos **32 GB de RAM** y **500 GB de disco libres** aproximadamente
- [ ] Software de virtualización (VirtualBox / VMware / Proxmox)
- [ ] Conocimientos básicos de Linux y redes
- [ ] Herramientas: Wazuh 4.14.5, Shuffle v2.2.0, TheHive 5.7.3, pfsense 2.8.1


---
## Inicio rápido

```bash
# Clona el repositorio
git clone https://github.com/tu-usuario/repo.git
cd repo

# Revisa la guía de instalación
cat setup/01-getting-started.md
```

Para una guía completa de instalación, ve a [`/setup`](Setup%20-%20detection.md).


---
## 📂 Estructura del proyecto

```
.
├── docs/
│   ├── journal/          # Bitácoras diarias: qué se hizo, falló y aprendió
│   └── reference/        # Documentación: arquitectura, decisiones de diseño
├── setup/                # Guías de instalación paso a paso
├── configs/              # Archivos de configuración reales, listos para usar
├── scripts/
│   ├── detection/        # Scripts de análisis y detección de amenazas
│   └── automation/       # Scripts de automatización y utilidades
├── detections/           # Reglas de detección (Sigma, Suricata, Wazuh)
├── diagrams/             # Diagramas de arquitectura y de red
├── data/                 # Logs y PCAPs de ejemplo para practicar
├── resources/            # Enlaces, artículos y material de aprendizaje
├── .github/              # Plantillas de issues y pull requests
└── README.md
```


| Carpeta               | Contenido                                                                   |
| --------------------- | --------------------------------------------------------------------------- |
| `/docs/journal`       | Registro día a día del progreso, problemas y aprendizajes                   |
| `/docs/reference`     | Documentación de referencia no cronológica (arquitectura, decisiones)       |
| `/diagrams`           | Diagramas de arquitectura y topología del laboratorio                       |
| `/setup`              | Instalación y configuración de cada herramienta del SOC                     |
| `/scripts/detection`  | Scripts para análisis y detección de amenazas                               |
| `/scripts/automation` | Scripts de automatización y tareas repetitivas                              |
| `/configs`            | Configuraciones reales (`.conf`, `.yml`, `.json`) listas para copiar y usar |
| `/resources`          | Enlaces externos, papers, cursos y referencias                              |
| `/detections`         | Reglas de detección propias (Sigma, Suricata, Wazuh, etc.)                  |
| `/data`               | Muestras de logs, PCAPs y eventos para practicar sin generar tráfico propio |
| `/.github`            | Plantillas para issues y pull requests                                      |

> 💡 Si alguna de estas carpetas aún no tiene contenido (por ejemplo, `/detections` o `/data`), puedes omitirla del repo hasta que la necesites — no hace falta crear la estructura completa desde el día uno.


---
## Cómo usar este proyecto

1. **Explora la documentación**
   Sigue las bitácoras día a día en [`/docs`](docs.md) para una guía paso a paso. Cada entrada incluye configuraciones, herramientas usadas y lecciones aprendidas.

2. **Configura tu entorno**
   Clona este repositorio y sigue las guías en [`/setup`](Setup%20-%20detection.md) para replicar el laboratorio.

3. **Experimenta y aprende**
   Usa los scripts, configuraciones y ejemplos provistos para probar y mejorar tu propio SOC. Siéntete libre de adaptarlos a tus necesidades.

4. **Contribuye**
   ¿Encontraste una mejor forma de configurar algo? ¿Tienes sugerencias o mejoras? ¡Las contribuciones son bienvenidas! Ver [Contribuir](#-contribuir).


---
## Contribuir

¡Las contribuciones son bienvenidas! Si quieres colaborar:

1. Haz un fork del repositorio
2. Crea una rama para tu cambio (`git checkout -b mejora/nombre-mejora`)
3. Haz commit de tus cambios con mensajes claros
4. Abre un Pull Request describiendo qué cambia y por qué

Consulta [`CONTRIBUTING.md`](./CONTRIBUTING.md) para más detalles sobre el estilo de commits y el proceso de revisión.


---
## Recursos y créditos

Enlaces, artículos y material usado como referencia están disponibles en [`/resources`](./resources).


---
## Licencia

Este proyecto está bajo la licencia [MIT](./LICENSE). Puedes usarlo, modificarlo y compartirlo libremente, dando el crédito correspondiente.


---
## Contacto

¿Tienes preguntas o ideas? Abre un [issue](../../issues) o inicia una [discusión](../../discussions) en este repositorio. ¡Colaboremos y aprendamos juntos!
