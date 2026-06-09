# RHEL Admin Ticket Labs

Selected Red Hat-based Linux administration labs covering infrastructure provisioning, identity integration, storage, automation, hardening, logging, monitoring, troubleshooting, backup, and vulnerability remediation.

This repository contains sanitized portfolio documentation based on practical Linux administration scenarios completed in a virtualized infrastructure environment.

## Purpose

The purpose of this repository is to document selected Linux administration work in a professional, portfolio-ready format. Each lab focuses on operational tasks commonly performed by Linux system administrators and Linux system engineers, including server provisioning, service configuration, storage management, access control, automation, monitoring, troubleshooting, and security remediation.

This repository complements deeper automation-focused projects by documenting the underlying Linux administration work behind infrastructure provisioning, identity, storage, hardening, monitoring, and remediation tasks.

## Lab Environment

The original work was performed in a virtualized Red Hat-based Linux environment using technologies such as:

- RHEL/CentOS Stream 9 systems
- VMware vSphere-based virtualization
- FreeIPA, SSSD, and Kerberos identity integration
- NFS shared storage
- Apache HTTP Server
- firewalld and SELinux
- Ansible automation
- Graylog centralized logging
- Nagios and CheckMK monitoring
- Veeam Linux Agent
- Lynis and OpenVAS security tooling

All environment-specific values have been sanitized.

## Featured Labs

| Lab | Focus |
|---|---|
| [VM Provisioning Lifecycle](labs/01-vm-provisioning-lifecycle/) | VM provisioning, templates, Kickstart, resource pools, and post-build validation |
| [Networking and Host Configuration](labs/02-networking-host-configuration/) | Static networking, local name resolution, and connectivity validation |
| [FreeIPA Identity Integration](labs/03-freeipa-identity-integration/) | IPA client enrollment, SSSD, Kerberos validation, and identity lookup |
| [NFS Storage and Shared Directories](labs/04-nfs-storage-shared-directories/) | Persistent NFS mounts, shared directories, ownership, and permissions |
| [Apache SELinux Firewall](labs/05-apache-selinux-firewall/) | Apache deployment, SELinux port labeling, firewalld, and HTTP validation |
| [SSH Hardening and Bastion Access](labs/06-ssh-hardening-bastion-access/) | Root SSH restriction, bastion-only access, and firewalld rich rules |
| [Ansible Server Administration](labs/07-ansible-server-administration/) | Inventory configuration, connectivity testing, patching, and playbook validation |
| [Logging, Logrotate, and Graylog](labs/08-logging-logrotate-graylog/) | rsyslog forwarding, Graylog validation, and HTTPD log rotation |
| [Monitoring with Nagios and CheckMK](labs/09-monitoring-nagios-checkmk/) | Monitoring agent setup, host onboarding, and service alert validation |
| [Backup and Recovery Operations](labs/10-backup-recovery-operations/) | Backup storage preparation, Linux agent installation, and recovery readiness |
| [Storage and Performance Troubleshooting](labs/11-storage-performance-troubleshooting/) | Storage expansion, service health checks, and performance troubleshooting |
| [Vulnerability Scanning and Remediation](labs/12-vulnerability-scanning-remediation/) | Lynis, OpenVAS, vulnerability review, remediation, and validation |

## Documentation Format

Each lab uses the following structure:

- Objective
- Environment
- Work Performed
- Commands or Configuration
- Validation
- Outcome
- Lessons Learned

## Sanitization Notice

All environment-specific values have been sanitized, including hostnames, IP addresses, usernames, domains, internal URLs, credentials, and organization-specific references.

This repository contains selected, rewritten portfolio labs only and does not include raw internal notes or private lab artifacts.
