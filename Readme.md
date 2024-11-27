# Comprehensive Monitoring and Alerting System on AWS

## **Overview**
This project demonstrates a fully functional monitoring and alerting setup using **Prometheus**, **Node Exporter**, **Blackbox Exporter**, and **Alertmanager** on AWS EC2 instances. The goal is to monitor system health, web application performance, and provide real-time alerts for critical issues.

---


### **Components:**
1. **Instance 1: Application and Node Exporter**
   - Hosts a Java-based web application on Nginx.
   - Node Exporter monitors system-level metrics (CPU, memory, disk).

2. **Instance 2: Prometheus, Blackbox Exporter, and Alertmanager**
   - Prometheus collects metrics.
   - Blackbox Exporter probes endpoints for uptime and response times.
   - Alertmanager handles alerts and integrates with Gmail for notifications.

### **Key Metrics Monitored:**
1. **Hardware Metrics:** CPU, memory, and disk usage via Node Exporter.
2. **Application Uptime:** Probed by Blackbox Exporter.
3. **Alert Rules:**
   - Instance down.
   - Website downtime.
   - High CPU/memory usage.
   - Low disk space.

---

## **Setup Instructions**

### **Phase 1: Provision EC2 Instances**
1. **Launch Two EC2 Instances:**
   - Instance Type: `t2.medium` (2 vCPUs, 8GB RAM).
   - AMI: Ubuntu Server 24.04 LTS.

2. **Security Group Configuration:**
   - Allow inbound traffic on the following ports:
     - `22` (SSH), `80` (HTTP for Nginx),
     - `9090` (Prometheus), `9093` (Alertmanager),
     - `9100` (Node Exporter), `9115` (Blackbox Exporter),
     - `587` (SMTP for email).

---

### **Phase 2: Configure Instance 1**

 1. **Install Node Exporter:**
   ```bash
   wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
   tar xvfz node_exporter-1.8.1.linux-amd64.tar.gz
   cd node_exporter-1.8.1.linux-amd64
   ./node_exporter &
   ```
 2. **Install Nginx and Deploy the Web Application:**
   ```bash
   sudo apt update
   sudo apt install nginx -y
   sudo apt install openjdk-17-jre-headless -y
   sudo apt-get install maven -y
   ```
 3. **Clone the application repository**
   ```bash
   git clone https://github.com/Sameer-Anwaar-Hussain/Prometheus-Monitoring.git
   cd BoardGame
   mvn package
   cd target
   java -jar database_service_project-0.0.2.jar &
   Access the application at <Instance_1_IP>:8080.
   ```

### **Phase 3: Configure Instance 2**

   1. **Install Prometheus:**
   ```bash
   wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
   tar xvfz prometheus-2.52.0.linux-amd64.tar.gz
   cd prometheus-2.52.0.linux-amd64
   ./prometheus --config.file=prometheus.yml &
   ```
   2. **Install Blackbox Exporter:**
   ```bash
   wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
   tar xvfz blackbox_exporter-0.25.0.linux-amd64.tar.gz
   cd blackbox_exporter-0.25.0.linux-amd64
   ./blackbox_exporter &
   ```
   3. **Install Alertmanager:**
   ```bash
   wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
   tar xvfz alertmanager-0.27.0.linux-amd64.tar.gz
   cd alertmanager-0.27.0.linux-amd64
   ./alertmanager --config.file=alertmanager.yml &
   ```

### **Configuration Placement and Service Access**
   1. **Prometheus Configuration (prometheus.yml):**
      - Location: Place the file in the directory where Prometheus binary    resides (e.g., /home/ubuntu/prometheus-2.52.0.linux-amd64/).

      **Restart Prometheus:**
      ```bash
          pkill prometheus
         ./prometheus --config.file=prometheus.yml &
      ```
   **Access Prometheus:**
     - Open <Instance_2_IP>:9090 in your browser.

   2. **Alert Rules (alert_rules.yml)**
       - Location: Place the file in the same directory as prometheus.yml.
       - Link to Prometheus: Ensure prometheus.yml includes the following line to **load the rules:**
        ```bash
            rule_files:
               - "alert_rules.yml"
        ```
        - Restart Prometheus: Follow the steps mentioned above.

   3. **Alertmanager Configuration (alertmanager.yml)**
        - Location: Place the file in the Alertmanager directory (e.g., /home/  ubuntu/alertmanager-0.27.0.linux-amd64/).
     
       **Restart Alertmanager:**

          ```bash
            pkill alertmanager
           ./alertmanager --config.file=alertmanager.yml &
          ```
      **Access Alertmanager:**
      - Open <Instance_2_IP>:9093 in your browser.


    4. **Blackbox Exporter Configuration:**
       - Ensure the prometheus.yml scrape job is updated for Blackbox Exporter.
       - Restart Prometheus: Follow the steps mentioned above.
  
    5. **Node Exporter:**
    - Node Exporter runs as a standalone binary; no additional configuration is needed.
    - Ensure Prometheus scrape job points to <Instance_1_IP>:9100.



### **Gmail Setup for Alertmanager**
   
**1. Enable Two-Factor Authentication (2FA):**
- Log in to your Google account.
      -  Go to Google Account Security.
      - Under "Signing in to Google," click 2-Step Verification and follow the prompts.

    **2. Generate an App Password:**
    - Go to App Passwords.
    -Choose Other (Custom name) and name it "Prometheus Alertmanager."
    - Click Generate to receive a 16-character app password.

    **3. Configure the App Password:**
    - Replace <your-app-password> in alertmanager.yml with the app password:
    ```yaml
       global:
       smtp_smarthost: 'smtp.gmail.com:587'
       smtp_from: 'your_email@gmail.com'
       smtp_auth_username: 'your_email@gmail.com'
       smtp_auth_password: '<your-app-password>'
    ```
    4. **Restart Alertmanager:**
    - After updating alertmanager.yml, restart Alertmanager:
     ```bash
        pkill alertmanager
        ./alertmanager --config.file=alertmanager.yml &
     ```
    5. **Test Alerts:**
    - Trigger an alert (e.g., by stopping Node Exporter) and check your Gmail for notifications.


## Future Improvements
- Automate setup using Terraform or Ansible.
- Integrate with ELK Stack for enhanced observability.

## File Structure
- **prometheus.yml**: Prometheus configuration.
- **alert_rules.yml**: Custom alert rules.
- **alertmanager.yml**: Alertmanager configuration.
- **README.md**: Documentation for the project.
   
