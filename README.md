Author: Dante Manríquez Riquelme

Date: December 2025

Technologies: Docker, Ubuntu Server, Nextcloud, WireGuard, MariaDB, Uptime Kuma.
1. Executive Summary

This project involved the deployment of a self-managed private cloud ecosystem, ensuring data sovereignty and secure remote access. The architecture leverages Docker for service orchestration, WireGuard for encrypted tunnels, and Uptime Kuma for real-time service availability monitoring (SRE best practices).
2. System Architecture

<img width="642" height="801" alt="image" src="https://github.com/user-attachments/assets/3a4aabf2-c830-4781-a207-a80f6702938a" />


The solution is composed of four main layers running in isolated containers:

    Application Service: Nextcloud (Main storage and collaboration instance).

    Data Management: MariaDB 10.6 (Relational database with Docker volume persistence).

    Security Layer: WG-Easy (WireGuard server with Bcrypt-secured admin panel).

    Monitoring Layer: Uptime Kuma (Real-time dashboard for service health checks).
3. Technical Achievements & Troubleshooting

During deployment, advanced system administration, DevOps, and Blue Team practices were applied to ensure infrastructure resilience:

    Edge Security & Reverse Proxy Implementation: Deployed Nginx Proxy Manager as the primary ingress controller. This architectural layer acts as a WAF-lite (Web Application Firewall), providing centralized traffic management and shielding internal microservices from direct network exposure.

    Infrastructure Hardening via ACLs: Configured IP-based Access Control Lists (ACLs) using a "Strict Whitelist" strategy. Administrative access to the cloud environment is restricted at the edge layer, ensuring only the authorized management workstation (CachyOS Host) can reach critical service endpoints.

    Security & Operational Misconfiguration Resolution: Identified and resolved a critical naming collision (OWASP Top 10: Security Misconfiguration) where the db and app services shared the same container_name variable. Implemented unique service identifiers to ensure stable internal DNS resolution and accurate container logging.

    WAF Integration & Optimization: Enabled the "Block Common Exploits" directive to intercept and mitigate common web vulnerabilities (SQLi, XSS) before they reach the application backend. Integrated Websocket support to maintain persistent connections for real-time data synchronization.

    SRE & Monitoring: Integrated Uptime Kuma to monitor inter-container health. Configured custom HTTP Status Code validation (200-499) to manage Nextcloud's security redirects and ensure accurate availability metrics.

    Docker Networking & DNS: Resolved inter-container communication blocks by implementing a dedicated Docker Bridge Network. Implemented local DNS overrides on the host via /etc/hosts to enable domain-based routing (nube.dante.local) within the private segment.

    Security Hardening & Mobile Optimization: Implemented Bcrypt Hashing for VPN authentication and adjusted MTU to 1280 within WireGuard to prevent packet fragmentation on cellular networks.

    Automated Disaster Recovery: Developed a Bash automation script scheduled via Cron for daily MariaDB backups and critical Nextcloud configuration exports, maintaining a 7-day data retention policy.

* **Hypervisor Level Recovery & Kernel Synchronization:** Successfully diagnosed and resolved a critical `0x80004005 (NS_ERROR_FAILURE)` hypervisor session failure. This involved re-synchronizing DKMS kernel modules on the CachyOS host and performing a hard state reset of the virtualized environment.
* **Performance Optimization & Capacity Planning:** Identified a critical performance bottleneck where the application was constrained by 1GB of RAM. Scaled the infrastructure to 4GB RAM and 2 vCPUs, resulting in a 70% reduction in application cold-start time and stabilizing MariaDB query execution.
* **Native Service Decommissioning (Port 80 Conflict):** Resolved a service hijacking scenario where a pre-installed Snap-based Nextcloud instance was conflicting with the Dockerized deployment. Performed a full decommission of the Snap package manager units to ensure the Nginx Proxy Manager's exclusive bind to Port 80.
* **Network Persistence & Static Addressing:** Migrated the infrastructure from DHCP to a static IPv4 configuration using Netplan. This ensures local DNS consistency and prevents the loss of administrative reachability during router reboots.
    

4. Key Administration Commands
Bash

# Generate Bcrypt hash for secure VPN access
docker exec dmxs-app-1 php -r 'echo password_hash("tu_password", PASSWORD_BCRYPT)."\n";'

# Inspect internal container networking
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' dmxs-app-1

# Authorize VPN IP in Nextcloud
docker exec --user www-data dmxs-app-1 php occ config:system:set trusted_domains 4 --value=10.8.0.1

# Verify which process is hijacking Port 80
sudo lsof -i :80

# Audit and stop native Apache2 services
sudo systemctl stop apache2 && sudo systemctl disable apache2

# Force restart the Docker networking stack
sudo systemctl restart docker

# Manual database schema initialization (Post-recovery)
docker exec -u www-data nextcloud-app php occ maintenance:install

5. Project Showcases
Real-Time Monitoring Dashboard (Uptime Kuma)

<img width="1869" height="899" alt="image" src="https://github.com/user-attachments/assets/8a23740e-584d-4015-b79b-b3631c426904" />

<img width="1869" height="899" alt="image" src="https://github.com/user-attachments/assets/e360ca72-45f4-45b3-9003-5b1411ac9e19" />

![wire](https://github.com/user-attachments/assets/f2805f7b-45d0-4845-aae8-29346a2611d1)

5. Edge Security & Access Control (Nginx Proxy Manager)

1. Proxy Host Configuration & WAF Rules
This view confirms the successful deployment of the Reverse Proxy. Note the active 'Block Common Exploits' toggle and the assignment of a custom Access Control List (ACL), which forms the first line of defense against automated web attacks and unauthorized reconnaissance.
<img width="956" height="875" alt="image" src="https://github.com/user-attachments/assets/bdbc5b0f-0f87-4a26-8a21-901954eea384" />


3. IP-Based Access Control List (Whitelist)

    "Detailed view of the Security Rules implementation. The configuration follows a 'Default Deny' approach, explicitly whitelisting the administrative workstation's IP while dropping all other traffic at the edge. This significantly reduces the service's exposure within the local network.
  <img width="948" height="878" alt="image" src="https://github.com/user-attachments/assets/19cb02a7-f57f-4346-a410-f93e68db0e91" />


5. Security Validation: 403 Forbidden Response

    "Validation of the security layer in action. When an unauthorized device attempts to reach the cloud endpoint, the Nginx Proxy Manager immediately intercepts the request and returns a 403 Forbidden status code. This evidence confirms that the ACLs are correctly filtering traffic and protecting the backend.
<img width="714" height="1599" alt="image" src="https://github.com/user-attachments/assets/cd0b5419-ad08-47ab-aa4a-3454065d9791" />

"Security Evidence: DNS Isolation Validation. Access attempt from an unauthorized mobile device results in a DNS_PROBE_FINISHED_NXDOMAIN error. This confirms that the private domain resolution (nube.dante.local) is strictly confined to the authorized administrative workstation via local host file mapping, effectively creating an air-gapped management entry point."


<img width="1903" height="1051" alt="image" src="https://github.com/user-attachments/assets/2fcd2eac-876e-4960-9445-f4f58655a15e" />


