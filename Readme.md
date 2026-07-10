# 🛡️ Building My First SOC: The ShadowCipher Wazuh Lab

Hey, I'm Kritan Aryal. I'm a cybersecurity student and an aspiring SOC Analyst. 

I built this lab because I was tired of just reading about SIEMs, threat hunting, and incident response in textbooks. I wanted to get my hands dirty, break things, and actually see what an attack looks like from the defender's side. 

This repository documents how I built the **ShadowCipher SOC Lab** using Wazuh. It includes all the commands I used, the mistakes I made, and how I tuned the system so it wouldn't spam me with false positives. If you're studying for an L1 role or just want to practice detection engineering, feel free to use this as a blueprint.

---

## 🧠 What I Actually Built

The goal wasn't just to install software; it was to simulate a real enterprise environment. 
* **The Brain:** An Ubuntu server running the Wazuh Manager, Indexer, and Dashboard.
* **The Victims (Agents):** A couple of VMs (Windows and Linux) that I intentionally attack to generate telemetry.
* **The Intelligence:** VirusTotal API integration to automatically check suspicious files.

## ⚠️ The Reality Check: What You Actually Need

When I first tried building this, I severely underestimated how resource-hungry a SIEM is. The Wazuh Indexer (which is basically Elasticsearch) will silently crash if it runs out of memory. **Learn from my pain and don't skimp on RAM.**

### The Hardware (My VM Setup)
* **The SIEM Server (Ubuntu 22.04 LTS):** 
  * You need at least **4 CPU Cores** and **8GB of RAM** (honestly, give it 12GB if you have it). 
  * Give it at least **50GB of storage**. Logs eat up disk space ridiculously fast.
* **The Endpoints (Windows 10 / Ubuntu):** 
  * 1 Core and 2GB RAM is fine for these. They just sit there and send logs.

### The Network
I assigned a static IP to my Wazuh Server so the agents wouldn't lose connection if my router rebooted. You also need to open a few ports on the server:
* `443` (So we can see the web dashboard)
* `1514` (The pipeline where agents send their logs)
* `1515` (The door agents knock on to register for the first time)

---

## 🛠️ Phase 1: Prepping the Server

Before touching Wazuh, the Ubuntu server needs to be ready. This is exactly what I ran on my fresh install.

```bash
# 1. Update everything. Never start a project on a stale OS.
sudo apt update && sudo apt upgrade -y

# 2. Install the basics needed to pull down the Wazuh packages.
sudo apt install curl apt-transport-https unzip wget chrony ufw -y