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

During deployment, advanced system administration, DevOps, and Blue Team practices were applied:

    Security & Operational Hardening (Misconfiguration Fix): Identified and resolved a critical naming collision (OWASP Top 10: Security Misconfiguration) where the db and app services shared the same container_name variable. I implemented Service Isolation by assigning unique, explicit identifiers, ensuring correct internal DNS resolution and logging.

    SRE & Monitoring: Integrated Uptime Kuma to monitor inter-container health. Configured custom HTTP Status Code validation (200-499) to manage Nextcloud's security redirects.

    Docker Networking: Resolved inter-container communication blocks by implementing a dedicated Docker Bridge Network, allowing Uptime Kuma to reach the Nextcloud service via internal IP.

    Security Hardening: Implemented Bcrypt Hashing for VPN authentication. Fixed environment variable parsing errors in Docker by using escaped characters ($$) for complex cryptographic hashes.

    Nextcloud Trusted Domains: Authorized VPN and internal Docker subnets using occ commands, allowing secure access from both the WireGuard tunnel and the monitoring system.

    Mobile Optimization: Fixed packet fragmentation on cellular networks by adjusting MTU to 1280 within the WireGuard configuration.

    Automated Disaster Recovery: Implemented a Bash automation script via Cron for daily MariaDB backups and critical Nextcloud configs, including a 7-day retention policy.
    

4. Key Administration Commands
Bash

# Generate Bcrypt hash for secure VPN access
docker exec dmxs-app-1 php -r 'echo password_hash("tu_password", PASSWORD_BCRYPT)."\n";'

# Inspect internal container networking
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' dmxs-app-1

# Authorize VPN IP in Nextcloud
docker exec --user www-data dmxs-app-1 php occ config:system:set trusted_domains 4 --value=10.8.0.1

5. Project Showcases
Real-Time Monitoring Dashboard (Uptime Kuma)

<img width="1869" height="899" alt="image" src="https://github.com/user-attachments/assets/8a23740e-584d-4015-b79b-b3631c426904" />

<img width="1869" height="899" alt="image" src="https://github.com/user-attachments/assets/e360ca72-45f4-45b3-9003-5b1411ac9e19" />

![wire](https://github.com/user-attachments/assets/f2805f7b-45d0-4845-aae8-29346a2611d1)


