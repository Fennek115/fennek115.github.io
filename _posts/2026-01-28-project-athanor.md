---
layout: post
title: "Project Athanor: 10 Minutos para el Colapso - Análisis de un Honeypot en Azure"
date: 2026-01-28 12:00:00 -0300
categories: [Ciberseguridad, Blue Team]
tags: [Honeypot, T-Pot, Azure, Malware Analysis, Forensics, Suricata, Cowrie, Threat Intelligence]
image:
  path: /assets/img/2026-01-28-project-athanor/elastic_dashboard.png
  alt: Dashboard de Kibana mostrando ataques globales
---

## 1. Introducción: La Hostilidad del Ruido de Fondo

En ciberseguridad, a menudo teorizamos sobre la "superficie de ataque". Con **Project Athanor**, decidí medirla empíricamente. El objetivo no era solo recolectar inteligencia de amenazas (CTI), sino probar la resistencia de una infraestructura estándar expuesta a la nube pública sin protección perimetral.

La hipótesis era simple: "Internet es hostil". La realidad fue brutal: **La instancia colapsó en 10 minutos.**

---

## 2. El Incidente (T+10 Minutos)

Desplegué una instancia **Standard_D2s_v3** (2 vCPUs, 8GB RAM) en Microsoft Azure corriendo la plataforma **T-Pot 24.04**. 

* **T+00:00:** Exposición a la WAN (IP pública abierta sin NSG restrictivo).
* **T+05:00:** Sensores Honeytrap registran picos de escaneo masivo.
* **T+08:00:** Saturación de RAM al 95%. Contenedores Docker comienzan a competir por recursos.
* **T+10:00:** **Kernel Panic / OOM (Out of Memory).** El volumen de escritura de logs y conexiones entrantes saturó el buffer de memoria. El servidor dejó de responder.

Fue necesario un *Hard Reboot* para recuperar el control. Esto demuestra que en 2026, la "seguridad por oscuridad" es un mito. Un servidor sin *hardening* o WAF no sobrevive al escaneo automatizado.

---

## 3. Telemetría de Red: Lo Viejo y Lo Nuevo

Tras el reinicio, logré mantener el honeypot operativo durante 4 horas. El análisis de **Suricata** reveló una dicotomía fascinante en los vectores de ataque.

![Histograma de Alertas Suricata](/assets/img/2026-01-28-project-athanor/suricata_alert_histogram.png)
*Volumen de alertas de red detectadas por firma.*

### El Retorno del Rey: DoublePulsar
La alerta principal por volumen fue `ET EXPLOIT [PTsecurity] DoublePulsar Backdoor`. Esto confirma que internet sigue saturado de botnets antiguas buscando máquinas Windows vulnerables al exploit de la NSA (usado en WannaCry). Es "ruido de fondo" radiactivo que nunca desaparece.

### Vulnerabilidades Críticas (CVEs)
Sin embargo, entre el ruido, detectamos intentos quirúrgicos y modernos:

| CVE | Objetivo | Descripción |
|:---|:---|:---|
| **CVE-2025-55182** | Web Apps | Intento de RCE en frameworks modernos (React/Next.js). |
| **CVE-2024-14007** | IoT | Ataque a firmware NVMS-9000 (Cámaras/DVRs) para reclutamiento de botnets. |
| **CVE-2023-46604** | Enterprise | Exploit contra Apache ActiveMQ (usado frecuentemente para desplegar Ransomware). |

---

## 4. Origen y Objetivos

La procedencia de los ataques confirma la naturaleza global de las botnets. No hay un "país enemigo"; hay una infraestructura global comprometida utilizada como proxy.

![Mapa de Amenazas](/assets/img/2026-01-28-project-athanor/mapa_de_amenazas.png)
*Mapa de calor de IPs atacantes. Nótese la dispersión global.*

### Distribución por Puertos

![Ataques por Puerto Destino](/assets/img/2026-01-28-project-athanor/attack_by_destination_port.png)
*Servicios más atacados durante la ventana operativa.*

El gráfico de puertos valida la teoría de los "Low Hanging Fruits":
1.  **TCP/445 (SMB):** El rey indiscutible (Gusanos Windows/Ransomware).
2.  **TCP/22 (SSH):** Fuerza bruta constante y bots de reconocimiento.
3.  **TCP/80 & 443:** Escáneres de vulnerabilidades web (dirbusters).

---

## 5. Identidad y Fuerza Bruta

![Nube de Credenciales](/assets/img/2026-01-28-project-athanor/username_and_password_tagcloud.png)
*Combinación de Usuarios y Contraseñas más probados.*

Como se observa en la nube de etiquetas, los ataques de fuerza bruta se basan en diccionarios por defecto (`root`, `admin`, `123456`, `support`), buscando dispositivos IoT que nunca cambiaron sus credenciales de fábrica. La persistencia de `ubnt` (Ubiquiti) y `pi` (Raspberry Pi) es notable.

---

## 6. Análisis Forense de Shell (Cowrie)

La parte más interesante técnicamente ocurrió dentro del honeypot SSH (Cowrie). Los atacantes que lograron "entrar" ejecutaron scripts automatizados inmediatamente.

![Dashboard de Cowrie](/assets/img/2026-01-28-project-athanor/cowrie_dashboard.png)
*Visión general de las sesiones interactivas capturadas en Cowrie.*

**El Comando Crítico Detectado:**
```bash
cd /dev/shm; cat .s || cp /bin/echo .s; /bin/busybox FVYZL
