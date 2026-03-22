# Advanced-EDR-SOAR-Implementation-Proactive-Threat-Detection-with-Wazuh-Sysmon

1. Introducción y Arquitectura del Proyecto
Este proyecto documenta la implementación exitosa de un sistema de Detección y Respuesta de Endpoints (EDR) y Orquestación, Automatización y Respuesta de Seguridad (SOAR) en un entorno Windows 11.

El objetivo principal fue pasar de un monitoreo pasivo a una postura de defensa proactiva, integrando telemetría avanzada de sistema operativo con inteligencia de amenazas externa para automatizar la identificación y análisis de malware.

Stack Tecnológico:

Wazuh (Manager & Agent): Núcleo del SIEM/EDR para recolección y correlación de eventos.

Sysmon (System Monitor): Provee telemetría profunda a nivel de kernel de Windows.

VirusTotal API: Fuente de inteligencia de amenazas para análisis reputacional automatizado.

2. Implementación de Telemetría Profunda con Sysmon
El Desafío
Los logs estándar de seguridad de Windows no ofrecen suficiente visibilidad sobre la cadena de ejecución de un ataque. Se necesitaba una "lupa" forense para monitorear la creación de procesos, carga de drivers y conexiones de red en tiempo real.

La Solución
Se desplegó Sysmon con una configuración optimizada para capturar eventos críticos (Event ID 1 y 6). Esto nos permitió visibilizar incluso drivers legítimos de protección, fundamentales para el análisis de falsos positivos.


<img width="941" height="378" alt="screen1" src="https://github.com/user-attachments/assets/b5f58a18-d9bc-42b0-9b46-701f753373f5" />


"Evidencia forense de Sysmon: Captura exitosa de la carga del driver de kernel MbamChameleon.sys, demostrando visibilidad total sobre módulos críticos del sistema."

3. Configuración del Agente EDR (agent.conf)
Para orquestar la telemetría, se modificó el archivo agent.conf del Agente Wazuh para realizar tres acciones clave:

Ingesta de Sysmon: Se configuró el canal de logs Microsoft-Windows-Sysmon/Operational como fuente de eventos.

Monitoreo de Integridad de Archivos (FIM): Se activó el monitoreo en tiempo real del Escritorio del usuario (C:\Users\joaqu\Desktop) para detectar la descarga o creación de artefactos sospechosos.

Habilitación de Active Response: Se prepararon los disparadores (triggers) para ejecutar scripts de remediación ante alertas de nivel crítico (12+).

(Nota: Enlazar aquí al archivo /config/agent.conf subido al repo)

4. Integración SOAR & Inteligencia de Amenazas (VirusTotal)
Esta es la pieza central de la automatización. Ante la creación de un archivo nuevo en el Escritorio (detectado por FIM), el sistema extrae automáticamente el Hash (MD5/SHA256) y lo consulta contra la API de VirusTotal.

Demostración de Funcionamiento (Ejemplo Real #1)
Durante el laboratorio, se simuló la descarga de un binario malicioso conocido (malware_final.exe). El sistema ejecutó el flujo SOAR completo.

📸 SCREENSHOT #2 A INSERTAR AQUÍ: Usá la imagen image_5c4b02.jpg (la del Discover).

Qué recuadrar en rojo:

data.virustotal.malicious: 1

data.virustotal.positives: 67

data.virustotal.source.file: c:\users\joaqu\desktop\malware_final.exe

Leyenda en GitHub: "Automatización SOAR en acción: Alerta crítica generada tras la consulta automatizada a VirusTotal. 67 motores de antivirus confirmaron la naturaleza maliciosa del archivo detectado en tiempo real."

5. Detección por Comportamiento y Reglas Personalizadas
No todas las amenazas tienen una firma conocida. Se desarrollaron reglas personalizadas para detectar comportamientos anómalos, como el uso indebido de PowerShell para lanzar ejecutables.

Demostración de Funcionamiento (Ejemplo Real #2)
Se detectó la ejecución de un script de PowerShell sospechoso intentando realizar cambios en el sistema.

📸 SCREENSHOT #3 A INSERTAR AQUÍ: Usá la torta de Tipos de Malware Detectados de la image_51713e.png.

Qué recuadrar en rojo: La rebanada que dice: Powershell process or execution from standard....

Leyenda en GitHub: "Detección heurística/comportamental: Identificación de actividad sospechosa de PowerShell, permitiendo la visibilidad de ataques sin archivos (fileless) o scripts maliciosos."

6. Dashboard de Operaciones de Seguridad (SOC)
Finalmente, se diseñó un Dashboard personalizado para transformar miles de eventos crudos en métricas accionables para un analista de seguridad.

📸 SCREENSHOT #4 A INSERTAR AQUÍ: Usá la captura final image_5d2fc9.png.

Qué recuadrar en rojo:

El contador gigante con el número 5 (Alertas Críticas).

La rebanada de VirusTotal en la torta.

Leyenda en GitHub: "Vista Consolidada del SOC Joaquin: Visualización en tiempo real que prioriza alertas críticas (Nivel 12) de VirusTotal y Sysmon, reduciendo la 'fatiga de alertas' y acelerando el tiempo de respuesta."

7. Conclusiones y Próximos Pasos (Respuesta Activa)
Estado Actual
El sistema cumple con el ciclo completo de detección e investigación (EDR). La fase de respuesta automatizada (SOAR) se configuró en modo 'Audit-Only'.

Lección Aprendida sobre Remediación
Aunque el script de Active Response (borrado del archivo) fue disparado por el Manager, la ejecución final en el endpoint fue bloqueada por las protecciones nativas de Windows (Estructura de Permisos). En un entorno de producción, esto se resuelve elevando los privilegios del script o aislando el host de la red como medida de contingencia.

Conclusión Final
Este laboratorio demuestra que es posible implementar una defensa de nivel empresarial utilizando herramientas Open Source, logrando visibilidad total, orquestación inteligente y reducción significativa del riesgo de seguridad.
