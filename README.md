#🦠 Ransomware Detection Home Lab — Wazuh + Infection Monkey

Home Lab Personal · Abril 2026
Analista: Lisa M. Moreno | Rol: Blue Team / SOC Analyst
Entorno: Ubuntu Server (Wazuh) + Windows 10 + Infection Monkey v2.3.0


📌 Resumen
Simulación controlada de un ataque de ransomware usando Infection Monkey sobre un endpoint Windows 10 monitorizado por Wazuh SIEM. El objetivo: demostrar detección en tiempo real, correlación con MITRE ATT&CK, y alertas automáticas a Slack.
El lab tiene tres capas defensivas encadenadas:

Detección → Wazuh con alertas nivel 15 (crítico)
Enriquecimiento → VirusTotal API analiza hashes automáticamente
Respuesta → firewall-drop activo en alertas nivel 7+


⚠️ Entorno 100% aislado en VirtualBox. Ningún sistema real fue afectado.


🏗️ Arquitectura
┌─────────────────────────────────────────────┐
│               RED INTERNA (NAT)             │
│                                             │
│  ┌─────────────────┐   ┌─────────────────┐  │
│  │  Ubuntu Server  │◄─►│  Windows 10     │  │
│  │  Wazuh Manager  │   │  Wazuh Agent    │  │
│  │  100.84.115.29  │   │  Sysmon         │  │
│  └────────┬────────┘   └────────▲────────┘  │
│           │                     │            │
│           │         ┌───────────┴──────────┐ │
│           │         │  Infection Monkey    │ │
│           │         │  Island Server       │ │
│           │         │  100.111.171.76      │ │
│           │         └──────────────────────┘ │
└───────────┼─────────────────────────────────┘
            ▼
   ┌─────────────────┐    ┌──────────────────┐
   │  Slack          │    │  VirusTotal API  │
   │  #soc-incidents │    │  (hash lookup)   │
   └─────────────────┘    └──────────────────┘

⚔️ El ataque — Infection Monkey
Infection Monkey es una plataforma open-source de simulación de adversario de Akamai. Se configuró para ejecutar el módulo de ransomware sobre el agente WIND10.
8 archivos cifrados — evidencia del impacto
Mostrar imagen
Infection Monkey cifró 8 archivos en WIND10 con algoritmo Bit Flip:
HostArchivoWIND10C:\Users\Public\Monkey\List.txtWIND10C:\Users\Public\Monkey\MONKEY.txtWIND10C:\Users\Public\Monkey\MONKEY (1).txtWIND10C:\Users\Public\Monkey\MONKEY (2).txtWIND10C:\Users\Public\Monkey\Sysmon.zipWIND10C:\Users\Public\Monkey\KMSpico-CJKT.rarWIND10C:\Users\Public\Monkey\How to use KMSpico.txtWIND10C:\Users\Public\Monkey\TMServerAgent_Windows_auto_x86_64_BULTOC_SA_CLOUD.zip
La nota de rescate
Mostrar imagen
Infection Monkey desplegó un archivo README con la nota "Don't Panic" en el sistema comprometido. Evidencia visual del impacto dentro de la VM Windows 10.

🔍 Detección — Wazuh SIEM
59 alertas en 41 segundos
Mostrar imagen
Wazuh correlacionó el comportamiento del ransomware con MITRE ATT&CK y generó 59 alertas entre las 21:28:07 y las 21:28:48.
CampoValorAgenteWIND10TécnicaT1105 — Ingress Tool TransferTácticaCommand and ControlRule ID92213Rule Level15 — crítico (máximo en Wazuh)DescripciónExecutable file dropped in folder commonly used by malwareTécnica secundariaT1059.001 — PowerShell spawned

🔔 Alertas en tiempo real — Slack
Mostrar imagen
Las alertas llegaron automáticamente al canal #soc-incidents. El webhook de Slack actúa como canal SOC — cualquier analista recibe la notificación sin tener Wazuh abierto.

⚙️ Configuración — ossec.conf
Mostrar imagen
Las tres integraciones configuradas en /var/ossec/etc/ossec.conf:
xml<!-- Slack — alertas nivel 5+ -->
<integration>
  <name>slack</name>
  <hook_url>https://hooks.slack.com/services/[REDACTED]</hook_url>
  <level>5</level>
  <alert_format>json</alert_format>
</integration>

<!-- VirusTotal — análisis automático de hashes via syscheck -->
<integration>
  <name>virustotal</name>
  <api_key>[REDACTED]</api_key>
  <group>syscheck</group>
  <alert_format>json</alert_format>
</integration>

<!-- Active Response — bloqueo automático nivel 7+ durante 5 min -->
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <level>7</level>
  <timeout>300</timeout>
</active-response>
Mostrar imagen

🗺️ MITRE ATT&CK Mapping
TácticaTécnicaIDDetectado porCommand and ControlIngress Tool TransferT1105Wazuh Rule 92213 (Level 15)ExecutionPowerShellT1059.001Wazuh Rule 92027ImpactData Encrypted for ImpactT1486Infection Monkey Report

💡 Conclusión
Wazuh detectó el comportamiento en menos de un minuto y con nivel máximo de criticidad. Sin embargo, los 8 archivos ya estaban cifrados cuando llegó la primera alerta.
Esto ilustra la brecha real entre detección y respuesta: un SIEM da visibilidad, pero sin un EDR o SOAR que aísle el endpoint en milisegundos, el daño ya ocurrió. Este lab fue construido específicamente para demostrar ese gap y argumentar la necesidad de una arquitectura de respuesta automatizada.

🛠️ Stack
ComponenteDetalleWazuh ManagerUbuntu Server 24.04Wazuh AgentWindows 10 (WIND10) + SysmonInfection Monkeyv2.3.0 — Akamai (open-source)VirusTotalAPI — grupo syscheckSlackWebhook → #soc-incidentsActive Responsefirewall-drop, level 7+, 300sVirtualBoxEntorno de virtualización

Lisa M. Moreno · Cybersecurity Analyst · Blue Team & SOC
LinkedIn · Portfolio · GitHub
