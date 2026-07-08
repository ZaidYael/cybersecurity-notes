# 🥒 [Write-up] Pickle Rick - TryHackMe

![TryHackMe](https://img.shields.io/badge/TryHackMe-212c42?style=for-the-badge&logo=tryhackme&logoColor=cc0000)
![Difficulty: Easy](https://img.shields.io/badge/Dificultad-F%C3%A1cil-green?style=for-the-badge)

## 1. Resumen Ejecutivo y Metadatos
* **Fecha:** 30/06/026
* **Plataforma:** TryHackMe
* **Categoría:** Pentesting / Linux Web Exploitation
* **Objetivo:** Conseguir los tres ingredientes secretos ocultos en el servidor para ayudar a Rick a volver a su forma humana.

---

## 2. Entorno de Trabajo y Herramientas
* **Sistema Atacante:** Kali Linux
* **Sistema Objetivo:** Ubuntu (Máquina virtual de la sala)
* **Herramientas utilizadas:** `nmap`, `gobuster` (o `dirb`), navegador web, comandos de Linux (`cat`, `ls`, `sudo`).

---

## 3. Fase de Reconocimiento y Enumeración

### Escaneo de Puertos (Nmap)
Se inició con un escaneo básico para identificar los puertos abiertos en la IP objetivo:
```bash
nmap -sV -sC -T4 <IP_OBJETIVO>
