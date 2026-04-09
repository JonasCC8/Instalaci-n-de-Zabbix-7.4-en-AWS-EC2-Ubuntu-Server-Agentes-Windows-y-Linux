📊 Zabbix 7.4 — AWS EC2 Installation Guide (Ubuntu Server)

Complete guide to deploy Zabbix 7.4 monitoring server on AWS EC2 with MySQL, Apache, and agent configuration for Linux and Windows hosts.

<br>
Mostrar imagen
Mostrar imagen
Mostrar imagen
Mostrar imagen
<br>

📋 Tabla de Contenidos
<br>
#Section1Arquitectura2Crear Instancia EC23Security Group4Conexión SSH5Hardening6Instalación Zabbix7Componentes8Base de Datos9Zabbix Server Config10Zona Horaria11Inicio de Servicios12Acceso Web13Agente Linux14Agente Windows15Verificación16Agregar Hosts17Buenas Prácticas
<br>

🏗️ Arquitectura
<br>
```
Internet
    │
    ▼
AWS EC2 (t3.medium)
├── OS:       Ubuntu Server 22.04 LTS
├── Zabbix:   7.4
├── Database: MySQL 8.0
├── Web:      Apache2 + PHP
│
├── Port 80/443   ← Web Interface
├── Port 10051    ← Zabbix Server
└── Port 10050    ← Zabbix Agent
```
<br>

☁️ 1. Crear Instancia en AWS EC2
<br>
FieldValueInstance typet3.mediumAMIUbuntu Server 22.04 LTSStorageMinimum 20 GBKey pairCreate or select existing
<br>

🔐 2. Configurar Security Group
<br>
TypeProtocolPortSourceSSHTCP22My IPHTTPTCP800.0.0.0/0HTTPSTCP4430.0.0.0/0Zabbix ServerTCP100510.0.0.0/0Zabbix AgentTCP100500.0.0.0/0
<br>

⚠️ WARNING: In production environments, restrict source IPs to your specific network ranges instead of 0.0.0.0/0.

<br>

🖥️ 3. Conexión al Servidor
<br>
```bash
ssh -i tu-key.pem ubuntu@YOUR_PUBLIC_IP
```
<br>

🛡️ 4. Hardening Básico
<br>
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y
Configure firewall
sudo ufw allow OpenSSH
sudo ufw allow 80,443,10050,10051/tcp
sudo ufw enable
Verify firewall status
sudo ufw status

<br>

---

## 📦 5. Instalación de Zabbix 7.4

<br>
```bash
# Download Zabbix repository package
wget https://repo.zabbix.com/zabbix/7.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.4-1+ubuntu22.04_all.deb

# Install repository
sudo dpkg -i zabbix-release_7.4-1+ubuntu22.04_all.deb

# Update package list
sudo apt update
<br>

⚙️ 6. Instalación de Componentes
<br>
```bash
sudo apt install \
    zabbix-server-mysql \
    zabbix-frontend-php \
    zabbix-apache-conf \
    zabbix-sql-scripts \
    zabbix-agent \
    mysql-server -y
```
<br>

🗄️ 7. Configuración de Base de Datos
<br>
Step 1 — Access MySQL:
bashsudo mysql -uroot
<br>
Step 2 — Create database and user:
sqlCREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'StrongPassword';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;
<br>
Step 3 — Import Zabbix schema:
bashzcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix
<br>

💡 Tip: Replace StrongPassword with a secure password of your choice.

<br>

🔧 8. Configuración de Zabbix Server
<br>
```bash
sudo nano /etc/zabbix/zabbix_server.conf
```
<br>
Add or update the following line:
iniDBPassword=StrongPassword
<br>

🌐 9. Configuración de Zona Horaria
<br>
```bash
sudo nano /etc/zabbix/apache.conf
```
<br>
Add the following line:
iniphp_value date.timezone America/Mexico_City
<br>

💡 Change America/Mexico_City to your timezone. See PHP timezones list.

<br>

▶️ 10. Inicio de Servicios
<br>
```bash
# Restart all services
sudo systemctl restart zabbix-server zabbix-agent apache2
Enable services on boot
sudo systemctl enable zabbix-server zabbix-agent apache2
Verify services are running
sudo systemctl status zabbix-server
sudo systemctl status zabbix-agent
sudo systemctl status apache2

<br>

---

## 🌍 11. Acceso Web

<br>

Open in browser:
http://YOUR_PUBLIC_IP/zabbix

<br>

**Default credentials:**

| Field | Value |
|:------|:------|
| Username | `Admin` |
| Password | `zabbix` |

<br>

> ⚠️ **IMPORTANT:** Change the default password immediately after first login.

<br>

---

## 🖥️ 12. Instalación de Agente Linux

<br>
```bash
# Install agent
sudo apt install zabbix-agent -y

# Edit agent configuration
sudo nano /etc/zabbix/zabbix_agentd.conf
<br>
Update the following fields:
iniServer=ZABBIX_SERVER_IP
ServerActive=ZABBIX_SERVER_IP
Hostname=Linux-Host
<br>
```bash
# Restart and enable agent
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
Verify agent is running
sudo systemctl status zabbix-agent

<br>

---

## 🪟 13. Instalación de Agente Windows

<br>

**Step 1 — Download agent:**

Visit: [https://www.zabbix.com/download_agents](https://www.zabbix.com/download_agents)

<br>

**Step 2 — Edit configuration file** `zabbix_agentd.conf`:
```ini
Server=ZABBIX_SERVER_IP
ServerActive=ZABBIX_SERVER_IP
Hostname=Windows-Host
<br>
Step 3 — Install and start service:
cmdzabbix_agentd.exe --install
zabbix_agentd.exe --start
<br>

🔍 14. Verificación
<br>
```bash
# Test agent connectivity from Zabbix server
zabbix_get -s AGENT_IP -k system.hostname
Check Zabbix server logs
sudo tail -f /var/log/zabbix/zabbix_server.log
Check agent logs
sudo tail -f /var/log/zabbix/zabbix_agentd.log

<br>

---

## ➕ 15. Agregar Hosts

<br>

Go to: Configuration → Hosts
Click: Create Host
Fill in:
Host name:    your-server-name
IP address:   AGENT_IP
Port:         10050
Assign Templates:
→ Linux by Zabbix agent   (for Linux hosts)
→ Windows by Zabbix agent (for Windows hosts)
Click: Add ✅


<br>

---

## 🔐 Buenas Prácticas

<br>

| Practice | Description |
|:---------|:------------|
| 🔒 Use HTTPS | Configure SSL certificate with Let's Encrypt or AWS ACM |
| 🛡️ Restrict Security Groups | Limit access to known IPs only |
| 🗄️ Use AWS RDS | Replace local MySQL with managed RDS for production |
| 💾 Configure Backups | Schedule automated database backups |
| 🔑 Change defaults | Update Admin password and API keys immediately |
| 📊 Monitor the monitor | Set up alerts for Zabbix server itself |

<br>

---

## 📚 Referencias

<br>

- 📖 [Zabbix 7.4 Documentation](https://www.zabbix.com/documentation/7.4/)
- 📖 [Zabbix Download](https://www.zabbix.com/download)
- 📖 [AWS EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- 📖 [Ubuntu Server Guide](https://ubuntu.com/server/docs)

<br>

---

## 📄 Licencia

<br>

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

<br>

---

<div align="center">

<br>

**⭐ If this guide was helpful, give it a star! ⭐**

<br>

</div>
