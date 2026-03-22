### Advanced-EDR-SOAR-Implementation-Proactive-Threat-Detection-with-Wazuh-Sysmon

Este proyecto documenta la implementación de un sistema de **Detección y Respuesta de Endpoints (EDR)** y **Orquestación, Automatización y Respuesta de Seguridad (SOAR)** en un entorno controlado de Windows 11.

El objetivo fue transformar un monitoreo pasivo en una defensa proactiva, integrando telemetría de kernel con inteligencia de amenazas externa.

---

##  Arquitectura del Proyecto

El sistema se basa en tres pilares fundamentales:
* **Wazuh (Manager & Agent):** Recolección y correlación de eventos distribuidos.
* **Sysmon (System Monitor):** Visibilidad profunda a nivel de Kernel (Event ID 1, 6, etc.).
* **VirusTotal API:** Enriquecimiento de alertas y análisis reputacional automatizado.

---

<img width="1210" height="1890" alt="mermaid-diagram (1)" src="https://github.com/user-attachments/assets/c9d29977-d6f8-468b-8fb6-1c7225188f08" />

*Esta arquitectura representa un pipeline de detección proactiva de amenazas utilizando Sysmon para la recolección de telemetría a nivel endpoint y Wazuh como SIEM central.

*Se implementan reglas personalizadas para identificar comportamientos maliciosos alineados con MITRE ATT&CK, mientras que una capa tipo SOAR permite la automatización de respuestas o su simulación según las restricciones del entorno.
---


##  1. Telemetría de Kernel con Sysmon

Para superar las limitaciones de los logs estándar de Windows, se implementó **Sysmon**. Esto permite rastrear la carga de drivers y la creación de procesos sospechosos en tiempo real.

> **Evidencia Forense:** Captura de carga de drivers críticos y servicios del sistema.
<img width="941" height="378" alt="screen1" src="https://github.com/user-attachments/assets/6af024d4-ea0f-43ba-837d-bb2099fa057d" />

*En la imagen se observa la detección del driver `MbamChameleon.sys` y la actividad del propio binario de Sysmon, confirmando visibilidad total sobre módulos de kernel.*

---

##  2. Configuración del EDR (`agent.conf`)

La orquestación se logró modificando el archivo de configuración del agente para centralizar la ingesta de datos y activar el monitoreo de integridad (FIM).

* **Ingesta:** Canal `Microsoft-Windows-Sysmon/Operational`.
* **FIM:** Monitoreo en tiempo real de la ruta `C:\Users\joaqu\Desktop`.
* **Triggers:** Preparación de disparadores para niveles críticos (12+).

📂 [Ver archivo de configuración completo](./config/agent.conf)

---

##  3. Integración SOAR: Análisis con VirusTotal

Ante la creación de cualquier archivo nuevo en el escritorio, el sistema extrae el Hash y realiza una consulta automática a la API de VirusTotal sin intervención humana.

### Caso de Uso: Detección de Malware Real
Se introdujo el artefacto `test_malware.exe` para testear el flujo de trabajo.

> **Resultado del Análisis:**

<img width="944" height="379" alt="image" src="https://github.com/user-attachments/assets/0af337b0-4523-4df1-ad53-cb21a705255a" />

*El sistema reportó **67 detecciones positivas**, disparando una alerta de **Nivel 12** en el Manager de Wazuh de forma inmediata.*

---

##  4. Detección por Comportamiento 

No dependemos solo de firmas. Se crearon reglas para detectar técnicas de ataque comunes, como el uso de PowerShell para ejecutar binarios desde rutas no convencionales.

> **Alerta de Comportamiento:**

<img width="934" height="380" alt="image" src="https://github.com/user-attachments/assets/a63a9cca-aec8-4036-9a7a-433f8f3f7755" />
 
*Identificación de procesos de PowerShell ejecutando comandos fuera del estándar, permitiendo detectar ataques 'fileless'.*

---

## 5. Dashboard del SOC (Security Operations Center)

Se diseñó una interfaz personalizada para el analista, priorizando los eventos críticos y reduciendo la "fatiga de alertas".

<img width="1912" height="779" alt="screen4" src="https://github.com/user-attachments/assets/cee6d341-47ba-4c45-adb8-0002eee6c5db" />


**Métricas Clave:**
1.  **Contador de Alertas Críticas:** Enfoque directo en incidentes nivel 12-14.
2.  **Distribución por Nivel:** Separación clara entre ruido de sistema y amenazas reales.
3.  **Top Malware:** Identificación visual de los archivos más peligrosos detectados por VT.

---

##  Conclusiones Técnicas

* **Remediación (Active Response):** El sistema disparó las acciones de respuesta, aunque la eliminación final fue auditada debido a las restricciones de integridad de Windows (un comportamiento esperado en entornos de alta seguridad).
* **Escalabilidad:** Esta arquitectura es capaz de escalar a cientos de agentes, manteniendo una latencia de detección menor a 5 segundos.

---
**Desarrollado por:** Joaquín Domenech  
