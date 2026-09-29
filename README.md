# Hardened and Monitored Linux Server

A security-focused Linux server project designed to demonstrate practical knowledge of **Linux system administration, security hardening, network security, service monitoring, logging, and basic automation**.

The main goal of this project is to transform a standard Linux server into a more secure and observable environment by reducing its attack surface, hardening remote access, protecting exposed services, monitoring system health, and validating each security control through practical tests.

> **Project Status:** In Progress

---

## Project Objectives

This project focuses on the following areas:

- Secure SSH remote access
- Disable unnecessary privileged access
- Protect SSH against brute-force attacks
- Deploy a web service securely with HTTPS
- Restrict network access using host-based firewall rules
- Monitor critical services and system resources
- Automate health checks with Bash and Cron
- Review system and security logs
- Validate security controls through controlled tests
- Document the purpose and security impact of each configuration

---

## Architecture Overview

```text
                         Client
                           |
                           |
                    Internet / Network
                           |
                           v
                  +------------------+
                  |   UFW / iptables |
                  |  Host Firewall   |
                  +------------------+
                       |         |
                       |         |
                       v         v
                    OpenSSH     Nginx
                       |         |
                       |         +---- HTTPS / TLS
                       |
                    Fail2ban
                       |
                       v
               Authentication Logs
                       |
                       +-------------------+
                                           |
                                           v
                                  Monitoring Scripts
                                           |
                                      Bash + Cron
                                           |
                                           v
                                  Security / Health Log
```

---

## Security Components

### 1. SSH Hardening

SSH is one of the primary administration interfaces of a Linux server and is therefore an important attack surface.

The SSH configuration is hardened by applying the following controls:

- Disable password-based authentication
- Use SSH key-based authentication
- Disable direct root login
- Restrict unnecessary remote access
- Optionally move SSH from the default port to a custom port for lab practice

Example configuration:

```text
PasswordAuthentication no
PermitRootLogin no
```

### Security Rationale

Password authentication can be targeted by automated brute-force and credential-guessing attacks.

SSH key-based authentication provides stronger authentication compared to reusable passwords.

Disabling direct root login also prevents attackers from directly targeting the most privileged account.

---

## 2. Fail2ban Brute-Force Protection

**Fail2ban** is configured to monitor SSH authentication failures and automatically block IP addresses that repeatedly generate failed login attempts.

The implementation includes:

- Installing Fail2ban
- Enabling the SSH jail
- Monitoring failed SSH authentication attempts
- Automatically banning suspicious IP addresses
- Verifying the configuration through controlled login failures

Status can be checked with:

```bash
sudo fail2ban-client status sshd
```

### Security Rationale

Internet-facing SSH services are frequently scanned and targeted by automated brute-force tools.

Fail2ban adds an additional defensive layer by detecting repeated authentication failures and temporarily blocking the source address.

---

## 3. Nginx Web Server

A simple static web page is deployed using **Nginx**.

The web server is used to demonstrate:

- Basic Linux web service administration
- Service management
- Network exposure
- TLS configuration
- HTTPS redirection

Nginx service status can be verified with:

```bash
systemctl status nginx
```

---

## 4. HTTPS and TLS Configuration

The Nginx web service is configured to support encrypted HTTPS communication.

For the lab environment, a self-signed certificate can be used to demonstrate the TLS configuration process.

The expected traffic flow is:

```text
HTTP Request
     |
     v
Port 80
     |
     v
HTTP -> HTTPS Redirect
     |
     v
Port 443
     |
     v
Encrypted HTTPS Connection
```

### Security Rationale

HTTP traffic is transmitted without encryption.

TLS provides:

- Confidentiality
- Integrity
- Server authentication

Redirecting HTTP traffic to HTTPS ensures that web communication uses encryption by default.

---

## 5. Host Firewall Configuration

The Linux server uses **UFW and iptables** to control inbound network traffic.

The firewall follows a restrictive approach:

> Allow only the services that are required.

Expected exposed services:

```text
SSH        -> Allowed
HTTP       -> Allowed
HTTPS      -> Allowed
Other      -> Denied
```

Firewall status can be reviewed using:

```bash
sudo ufw status verbose
```

Detailed packet filtering rules can also be inspected with:

```bash
sudo iptables -L -n -v
```

### Security Rationale

Every unnecessary open port increases the server's attack surface.

Restricting network access to required services reduces unnecessary exposure and limits potential attack vectors.

---

## 6. Automated Health-Check Script

A custom Bash script is used to monitor the operational state of the Linux server.

The script checks:

- Nginx service status
- Disk utilization
- Listening network ports
- Basic system health information

Example monitoring logic:

```text
Check Nginx
    |
    +---- Running ----> [OK]
    |
    +---- Stopped ----> [ALERT]

Check Disk Usage
    |
    +---- Normal -----> [OK]
    |
    +---- High --------> [ALERT]

Check Listening Ports
    |
    +---- Expected ----> [OK]
    |
    +---- Unexpected --> Review
```

When a problem is detected, the script records an alert in the monitoring log.

Example:

```text
[ALERT] Nginx service is not running
```

---

## 7. Automated Monitoring with Cron

The health-check script is executed periodically using **Cron**.

Example schedule:

```cron
*/15 * * * * /path/to/health-check.sh
```

This configuration executes the monitoring script every 15 minutes.

### Operational Rationale

Monitoring should not depend on manual execution.

Automating recurring health checks provides continuous visibility into the state of important services and system resources.

---

## 8. Monitoring Validation

The monitoring mechanism is tested by intentionally stopping the Nginx service.

Example:

```bash
sudo systemctl stop nginx
```

The health-check script should detect that Nginx is unavailable and generate an alert.

Expected result:

```text
[ALERT] Nginx service is not running
```

After the test:

```bash
sudo systemctl start nginx
```

This verifies that the monitoring script can detect an actual service failure instead of only producing static output.

---

## 9. Log Analysis

Linux generates valuable operational and security information across multiple log sources.

The project focuses on reviewing:

- systemd logs
- SSH authentication logs
- Nginx access logs
- Nginx error logs
- Fail2ban logs
- Firewall-related logs

Useful commands include:

```bash
journalctl
```

SSH-related events:

```bash
sudo journalctl -u ssh
```

Nginx access logs:

```bash
sudo tail -f /var/log/nginx/access.log
```

Nginx error logs:

```bash
sudo tail -f /var/log/nginx/error.log
```

Fail2ban logs:

```bash
sudo tail -f /var/log/fail2ban.log
```

### Purpose

The goal is not only to configure security tools, but also to understand where evidence is generated when something happens on the system.

Logs are essential for:

- Troubleshooting
- Security monitoring
- Incident investigation
- Authentication analysis
- Service failure analysis

---

## Security Monitoring Flow

```text
Linux Server
     |
     +-------------------------+
     |                         |
     v                         v
System Logs                Application Logs
     |                         |
 journalctl               Nginx / Fail2ban
     |                         |
     +------------+------------+
                  |
                  v
          Monitoring Script
                  |
                  v
          Condition Analysis
                  |
          +-------+-------+
          |               |
          v               v
        [OK]           [ALERT]
                          |
                          v
                  Security Report
```

---

## Validation Scenarios

Security controls are validated through practical tests rather than only being configured.

### SSH Authentication Test

Verify that:

- Password-based authentication is rejected
- SSH key authentication succeeds
- Direct root SSH login is disabled

---

### Fail2ban Test

Generate controlled failed SSH login attempts.

Then verify that Fail2ban detects and blocks the source IP.

```bash
sudo fail2ban-client status sshd
```

Expected result:

```text
Repeated failed authentication
            |
            v
        Fail2ban
            |
            v
        IP Banned
```

---

### HTTPS Test

Verify that HTTP traffic is redirected to HTTPS.

```text
http://server
      |
      v
Redirect
      |
      v
https://server
```

---

### Firewall Test

Verify that only intentionally exposed services are reachable.

Expected services:

```text
SSH
HTTP
HTTPS
```

All unnecessary inbound ports should remain blocked.

---

### Monitoring Test

Stop Nginx intentionally:

```bash
sudo systemctl stop nginx
```

Run the monitoring script or wait for the scheduled Cron execution.

Expected result:

```text
[ALERT] Nginx service is not running
```

Restart the service after validation:

```bash
sudo systemctl start nginx
```

---

## Security Principles Applied

### Defense in Depth

The project does not rely on a single security control.

Multiple layers are used together:

```text
SSH Key Authentication
        +
Root Login Restriction
        +
Host Firewall
        +
Fail2ban
        +
TLS
        +
Monitoring
        +
Logging
```

---

### Least Privilege

Only the services and network access required by the system are allowed.

Unnecessary access is restricted.

---

### Attack Surface Reduction

Unused or unnecessary exposure is minimized.

The server only exposes ports that are explicitly required.

---

### Secure-by-Default Configuration

The system is configured with restrictive defaults instead of allowing broad access and attempting to secure it afterward.

---

### Continuous Monitoring

The server is monitored periodically rather than relying entirely on manual checks.

Operational failures are recorded automatically.

---

## Technologies Used

| Technology | Purpose |
|---|---|
| Ubuntu / Linux | Server operating system |
| OpenSSH | Secure remote administration |
| SSH Keys | Strong authentication |
| UFW | Host firewall management |
| iptables | Packet filtering inspection |
| Fail2ban | Brute-force protection |
| Nginx | Web server |
| OpenSSL / TLS | HTTPS encryption |
| Bash | Monitoring and automation |
| Cron | Scheduled task execution |
| systemd | Service management |
| journalctl | System log analysis |
| Git | Version control |
| GitHub | Repository and documentation |

---

## Repository Structure

```text
hardened-linux-server/
|
├── README.md
|
├── configs/
|   ├── sshd_config
|   ├── fail2ban-jail.conf
|   └── nginx.conf
|
├── scripts/
|   ├── health-check.sh
|   └── security-report.sh
|
├── firewall/
|   └── iptables-rules.txt
|
├── logs/
|   └── example-security-report.log
|
└── screenshots/
    ├── ssh-hardening.png
    ├── fail2ban-test.png
    ├── nginx-https.png
    ├── firewall-status.png
    └── health-check-alert.png
```

---

## Skills Demonstrated

This project demonstrates practical knowledge of:

- Linux system administration
- Linux server hardening
- SSH security
- Public-key authentication
- Firewall configuration
- Network service management
- Brute-force protection
- Nginx administration
- TLS / HTTPS configuration
- Bash scripting
- Cron automation
- Service monitoring
- Linux logging
- Basic security monitoring
- Security testing
- Technical documentation
- Git and GitHub workflow

---

## Project Completion Checklist

### SSH Hardening

- [ ] SSH key-based authentication configured
- [ ] Password authentication disabled
- [ ] Direct root SSH login disabled
- [ ] SSH configuration documented

### Fail2ban

- [ ] Fail2ban installed
- [ ] SSH jail enabled
- [ ] Controlled failed login test performed
- [ ] IP banning successfully verified
- [ ] Test evidence documented

### Nginx and TLS

- [ ] Nginx installed
- [ ] Static web page deployed
- [ ] TLS certificate configured
- [ ] HTTPS working
- [ ] HTTP automatically redirected to HTTPS

### Firewall

- [ ] UFW rules reviewed
- [ ] Only required ports exposed
- [ ] iptables output reviewed
- [ ] Firewall configuration documented

### Monitoring

- [ ] Health-check Bash script created
- [ ] Nginx monitoring implemented
- [ ] Disk utilization monitoring implemented
- [ ] Listening port monitoring implemented
- [ ] Alert logging implemented
- [ ] Cron automation configured
- [ ] Real service failure detected successfully

### Logging

- [ ] journalctl reviewed
- [ ] SSH authentication logs reviewed
- [ ] Nginx logs reviewed
- [ ] Fail2ban logs reviewed
- [ ] Log locations documented

### Documentation

- [ ] Security rationale documented
- [ ] Configuration files added
- [ ] Terminal outputs added
- [ ] Screenshots added
- [ ] Tests documented
- [ ] Sensitive information removed before publishing

---

## Security Notice

Sensitive information must never be committed to the repository.

Examples include:

- Private SSH keys
- Passwords
- API keys
- Authentication tokens
- Real credentials
- Sensitive IP addresses
- Confidential log data
- Private server information

Before committing files, all configuration files, logs, and screenshots should be reviewed for sensitive information.

---

## What I Learned

Through this project, I aim to gain practical experience in securing and operating a Linux server rather than only studying Linux security concepts theoretically.

The project connects multiple areas of system security:

```text
Linux Administration
        |
        v
SSH Hardening
        |
        v
Network Filtering
        |
        v
Brute-Force Protection
        |
        v
TLS / HTTPS
        |
        v
Monitoring
        |
        v
Logging
        |
        v
Operational Security
```

The most important objective is to understand **why each security control exists, what threat it addresses, and how to verify that it actually works**.

---

## Future Integration

This project is the first part of a broader DevSecOps portfolio.

The hardened Linux server created here will later be used as a foundation for projects involving:

- Secure CI/CD pipelines
- Containerized applications
- Cloud infrastructure
- Infrastructure as Code
- Kubernetes security
- Security monitoring
- Incident response
- AI-assisted security analysis

This allows the projects to build on each other instead of functioning as isolated demonstrations.

---

## Disclaimer

This project is created for educational, laboratory, and portfolio purposes.

All security testing is performed only against systems that I own or environments where I have explicit authorization to perform testing.
