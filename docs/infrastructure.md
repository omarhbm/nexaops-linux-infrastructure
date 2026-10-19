# NexaOps — Infrastructure Documentation

## 1. Project Overview

NexaOps is a three-server Linux infrastructure lab built with VMware Workstation to demonstrate enterprise-style Linux system administration, configuration management, container deployment, database administration, and recovery testing.

The environment uses Rocky Linux 9 and Red Hat Enterprise Linux 9.8, with Ansible for centralized administration.

## 2. Server Inventory

| Hostname | Operating System | Private IP | Function |
|---|---|---|---|
| CTRL01 | Rocky Linux 9 | 192.168.60.10 | Ansible Control Node |
| APP01 | RHEL 9.8 | 192.168.60.20 | Docker and Nginx |
| DB01 | RHEL 9.8 | 192.168.60.30 | PostgreSQL Database |

### Virtual Machine Resources

| Server | vCPU | RAM | Storage |
|---|---|---|---|
| CTRL01 | 2 | 2 GB | 25 GB |
| APP01 | 2 | 2 GB | 35 GB |
| DB01 | 2 | 2 GB | 25 GB OS + 15 GB data |

## 3. Network Architecture

The infrastructure uses two VMware virtual networks.

### VMnet8 — NAT

- Network: `192.168.67.0/24`
- DHCP addressing
- Internet access for system updates and package installation
- Gateway: `192.168.67.2`

### VMnet5 — Host-only

- Network: `192.168.60.0/24`
- Static IP addressing
- Dedicated private communication between servers
- No default gateway on private interfaces

Internal management and database traffic use the private network.

## 4. Centralized Administration with Ansible

CTRL01 acts as the Ansible Control Node.

The Ansible project is stored in:

`/home/nexaadmin/nexaops-ansible`

The project includes inventories, playbooks, and reusable roles for common configuration, Docker, Nginx, and PostgreSQL.

### Connectivity Validation

Ansible connectivity was tested against CTRL01, APP01, and DB01 using:

```bash
ansible all -m ansible.builtin.ping -K
```

All three managed hosts responded successfully.

## 5. Application Server — APP01

APP01 hosts a containerized Nginx demonstration service.

| Setting | Value |
|---|---|
| Operating System | RHEL 9.8 |
| Container Engine | Docker |
| Deployment | Docker Compose |
| Web Server | Nginx |
| Container Name | `nexaops-nginx` |
| Service Address | `192.168.60.20:8080` |

The deployment uses Ansible roles and Jinja2 templates to manage the Nginx configuration and container definition.

The service is bound to the private network rather than all host interfaces.

## 6. Database Server — DB01

DB01 runs PostgreSQL 13.

| Setting | Value |
|---|---|
| Database | `helpdesk_db` |
| Application Role | `helpdesk_app` |
| Database Address | `192.168.60.30` |
| Port | `5432` |
| Authentication | SCRAM-SHA-256 |

PostgreSQL listens on localhost and the private DB01 address.

The `pg_hba.conf` configuration restricts the application database connection to APP01 at `192.168.60.20/32`.

The Firewalld configuration also restricts database access to the approved application server.

Remote database authentication from APP01 was successfully tested.

## 7. Storage Administration — LVM

A dedicated virtual disk was provisioned for PostgreSQL data.

| Component | Configuration |
|---|---|
| Physical Volume | `/dev/nvme0n2` |
| Volume Group | `vg_postgres` |
| Logical Volume | `lv_data` |
| Logical Volume Size | 12 GB |
| Filesystem | XFS |
| Mount Point | `/var/lib/pgsql/data` |

The database filesystem is configured for persistent mounting through `/etc/fstab`.

The storage configuration demonstrates Linux disk provisioning, LVM administration, filesystem management, and database storage separation.

## 8. Security Controls

Security measures implemented in the lab include:

- SSH key-based authentication for remote administration
- SELinux Enforcing mode
- Firewalld network restrictions
- PostgreSQL SCRAM-SHA-256 authentication
- Restricted database access from APP01
- Private network service bindings
- Linux user and group administration

The project is a lab environment; these controls do not represent a complete production security assessment.

## 9. Time Synchronization and Scheduling

Chrony was configured for system time synchronization on CTRL01 and DB01.

A daily PostgreSQL service health-check script was configured on DB01 using Cron.

| Setting | Value |
|---|---|
| Script | `/usr/local/bin/check-postgresql.sh` |
| Schedule | Daily at 08:00 |
| Log File | `/var/log/postgresql-health.log` |

The script checks PostgreSQL service availability and records the result.

## 10. PostgreSQL Backup and Recovery

A database backup and restoration test was performed using PostgreSQL utilities.

### Backup

```bash
pg_dump -Fc -d helpdesk_db \
  -f /var/backups/nexaops/postgresql/helpdesk_db.dump
```

The backup was created in PostgreSQL custom format.

### Restore

A separate database named `helpdesk_restore_test` was created, and the backup was restored using `pg_restore`.

The `support_tickets` table was used to validate data recovery.

### Recovery Verification

| Validation | Result |
|---|---|
| Original database records | 3 |
| Restored database records | 3 |
| Backup restoration | Successful |
| Data consistency check | Passed |

The successful restoration demonstrates the ability to recover database records from a backup archive.

## 11. Testing Summary

| Component | Validation |
|---|---|
| Server networking | Private network connectivity configured |
| Ansible | Successful connectivity to three servers |
| Docker / Nginx | Containerized web service deployed |
| PostgreSQL | Application server connection verified |
| LVM | Dedicated database filesystem configured |
| Security | SELinux and firewall controls configured |
| Cron | Database service health check scheduled |
| Backup / Restore | Three test records successfully restored |

## 12. Project Limitations

NexaOps is a practical infrastructure lab, not a production environment.

- The web service is a demonstration Nginx deployment, not a complete Helpdesk application.
- Some database and storage operations were performed manually.
- Automated backup rotation and off-site backup storage are not included.
- Centralized metrics monitoring is outside the completed scope.
- High availability and multi-node database replication are not implemented.

## 13. Conclusion

NexaOps demonstrates practical Linux infrastructure administration across three servers, covering automation, containerization, database operations, storage management, network security, scheduled maintenance, and backup recovery.

The project is intended as a technical portfolio example for Linux System Administrator and junior infrastructure engineering positions.
