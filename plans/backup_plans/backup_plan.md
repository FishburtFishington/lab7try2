# Windows Server 2025 Standard Domain Controller Backup Plan

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Environment Overview](#2-environment-overview)
3. [Pre-Backup Requirements](#3-pre-backup-requirements)
4. [Install and Configure Windows Server Backup](#4-install-and-configure-windows-server-backup)
5. [File Backup Plan](#5-file-backup-plan)
6. [Active Directory Backup Plan](#6-active-directory-backup-plan)
7. [Group Policy Backup Plan](#7-group-policy-backup-plan)
8. [Full Server Backup Plan](#8-full-server-backup-plan)
9. [Backup Schedule](#9-backup-schedule)
10. [Recovery Procedures](#10-recovery-procedures)
11. [Backup Verification Procedures](#11-backup-verification-procedures)
12. [Security Recommendations](#12-security-recommendations)
13. [Monitoring and Maintenance](#13-monitoring-and-maintenance)
14. [Troubleshooting Guide](#14-troubleshooting-guide)

---

## 1. Executive Summary

- [ ] All backups are stored on drive E:
- Purpose: Protect server files, Active Directory, and GPOs.
- Scope: One Domain Controller, Windows Server 2025 Standard, AD DS, DNS.
- Recovery Objectives: File, AD System State, GPO, and bare-metal recovery.
- Storage: Secondary drive E: (`E:\Backups\Files`, `E:\Backups\ActiveDirectory`, `E:\Backups\GroupPolicy`, `E:\Backups\FullServer`).
- Importance: AD loss = domain failure; backups must be verified and recoverable.

---

## 2. Environment Overview

| Component | Details |
|-----------|---------|
| OS | Windows Server 2025 Standard |
| Role | Domain Controller |
| Services | AD DS, DNS |
| Backup Software | Windows Server Backup |
| Backup Drive | E: (secondary hard drive) |
| Number of DCs | 1 |
| Third-party Backup | None |

---

## 3. Pre-Backup Requirements

- [ ] Verify AD health: `Get-ADDomainController`
- [ ] Verify DNS: `Resolve-DnsName`
- [ ] Check SYSVOL: `Get-ChildItem C:\Windows\SYSVOL`
- [ ] Check disk: `Get-PhysicalDisk`
- [ ] Event Viewer review
- [ ] Confirm E: space: `Get-PSDrive E`
- [ ] Confirm folders: `E:\Backups\Files`, `E:\Backups\ActiveDirectory`, `E:\Backups\GroupPolicy`, `E:\Backups\FullServer`
- [ ] Confirm admin permissions

---

## 4. Install and Configure Windows Server Backup

```powershell
Install-WindowsFeature Windows-Server-Backup -IncludeManagementTools
Get-WindowsFeature Windows-Server-Backup
```

- Configure schedule with `wbadmin`
- Set destination: `E:`
- Retention: maintain at least 7 days
- Verify: `wbadmin get status`

---

## 5. File Backup Plan

- Critical folders: business data, shared folders, admin docs, config files
- Destination: `E:\Backups\Files`
- Schedule: daily
- Verify after each run

```powershell
wbadmin start backup -backupTarget:E: -include:C: -allCritical -quiet
```

---

## 6. Active Directory Backup Plan

- [ ] Active Directory is backed up using System State backups
- System State includes AD DB, registry, SYSVOL, boot files
- Destination: `E:\Backups\ActiveDirectory`
- Frequency: weekly minimum

```powershell
wbadmin start systemstatebackup -backupTarget:E: -quiet
```

---

## 7. Group Policy Backup Plan

- [ ] Group Policy Objects are backed up and recoverable
- Use GPMC: Backup All
- Export to `E:\Backups\GroupPolicy`
- Verify exports complete

```powershell
Backup-Gpo -All -Path E:\Backups\GroupPolicy
```

---

## 8. Full Server Backup Plan

- Destination: `E:\Backups\FullServer`
- Schedule: weekly full + incremental
- Verify with `wbadmin get versions`

---

## 9. Backup Schedule

| Frequency | Task |
|-----------|------|
| Daily | File backup, verification |
| Weekly | System State, GPO backup, validation |
| Monthly | Full server review |
| Annual | Disaster recovery test, doc review |

---

## 10. Recovery Procedures

### File Recovery
- Restore from `E:\Backups\Files`
- Verify file integrity

### Active Directory Recovery
- System State restore via `wbadmin`
- Non-authoritative restore for replication
- Authoritative restore using `ntdsutil` if needed

### Group Policy Recovery
- Import from `E:\Backups\GroupPolicy`
- Verify policies in GPMC

### Bare-Metal Recovery
- Boot from recovery media
- Select System Image from `E:\Backups\FullServer`
- Validate services after restore

---

## 11. Backup Verification Procedures

- [ ] Backup verification is performed after every backup
- Check `wbadmin get status`
- Review Event Viewer logs
- Confirm files exist on `E:`
- Perform periodic restore tests
- Document results

---

## 12. Security Recommendations

- [ ] 3-2-1 Backup Strategy implemented
  - 3 copies of data
  - 2 different media types
  - 1 copy stored separately
- Least-privilege access to `E:\Backups`
- Backup encryption where supported
- Ransomware protection: restrict write access
- Audit backup folder changes
- Secure recovery documentation offline

---

## 13. Monitoring and Maintenance

| Interval | Action |
|----------|--------|
| Daily | Check backup status, review logs |
| Weekly | Verify storage, test GPO restore |
| Monthly | Review full server backups |
| Quarterly | Health checks, corrective actions |
| Annual | Disaster recovery test |

---

## 14. Troubleshooting Guide

| Issue | Symptoms | Resolution |
|-------|----------|------------|
| Backup failure | `wbadmin` error | Check disk space, permissions, services |
| Disk space | Low space on E: | Clean old backups, expand storage |
| AD backup error | System State fails | Verify AD health, retry `wbadmin` |
| GPO failure | Export missing | Run `Backup-Gpo` manually, verify GPMC |
| Access denied | Permission error | Confirm admin rights on backup folders |
| Restore failure | Corrupt image | Use alternate backup version |

---

## Final Confirmation

- [ ] All backups are stored on drive E:
- [ ] Files are backed up
- [ ] Active Directory is backed up using System State backups
- [ ] Group Policy Objects are backed up and recoverable
- [ ] Backup verification is performed after every backup
- [ ] Recovery procedures are documented and tested
- Document rendered in clean Markdown for Obsidian and GitHub
