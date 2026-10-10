# Windows Server 2025 Standard Domain Controller Decommissioning Plan

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Environment Assessment](#2-environment-assessment)
3. [Pre-Decommission Checklist](#3-pre-decommission-checklist)
4. [Application Removal](#4-application-removal)
5. [Data Removal](#5-data-removal)
6. [Hyper-V Decommissioning](#6-hyper-v-decommissioning)
7. [Active Directory Cleanup](#7-active-directory-cleanup)
8. [Group Policy Cleanup](#8-group-policy-cleanup)
9. [DNS Cleanup](#9-dns-cleanup)
10. [Domain Controller Demotion](#10-domain-controller-demotion)
11. [Remove Server Roles](#11-remove-server-roles)
12. [System Cleanup](#12-system-cleanup)
13. [Verification Phase](#13-verification-phase)
14. [Final Shutdown and Retirement](#14-final-shutdown-and-retirement)
15. [Troubleshooting Guide](#15-troubleshooting-guide)

---

## 1. Executive Summary

- Purpose: Decommission the Windows Server 2025 Standard DC lab environment.
- Scope: Remove applications, data, VMs, Hyper-V resources, AD objects, GPOs, DNS, roles.
- Risk: Accidental data loss; verify backups before proceeding.
- Validation: Confirm zero remnants remain.
- Final Goal: Safe shutdown and retirement.

---

## 2. Environment Assessment

| Component | Inventory Method |
|-----------|-----------------|
| Applications | `Get-WmiObject -Class Win32_Product` |
| Windows Roles | `Get-WindowsFeature` |
| AD Objects | `Get-ADObject` |
| Hyper-V VMs | `Get-VM` |
| DNS Zones | `Get-DnsServerZone` |
| Shares | `Get-SmbShare` |

---

## 3. Pre-Decommission Checklist

- [ ] Inventory all server roles
- [ ] Inventory all installed applications
- [ ] Inventory all virtual machines
- [ ] Inventory all user-created files
- [ ] Inventory all shares
- [ ] Inventory all Active Directory objects
- [ ] Inventory all Group Policy Objects
- [ ] Confirm backups complete
- [ ] Confirm stakeholder approval
- [ ] Confirm no retention requirements remain

---

## 4. Application Removal

```powershell
Get-WmiObject -Class Win32_Product
```
Uninstall each application via Control Panel or `Uninstall-Package`. Verify removal with `Get-WmiObject`. Clean registry/app remnants manually if needed.

---

## 5. Data Removal

- [ ] Delete all user-created files
- [ ] Delete all class/project files
- [ ] Delete all application data
- [ ] Delete all custom configurations
- [ ] Delete all installed applications
- [ ] Delete all custom backups
- [ ] Delete all shared folders

Action: `Remove-Item -Recurse -Force` on identified directories. Verify with `Get-ChildItem`.

---

## 6. Hyper-V Decommissioning

- [ ] Identify VMs: `Get-VM`
- [ ] Delete VMs: `Remove-VM -Name <VM> -Force`
- [ ] Delete checkpoints: `Get-VMSnapshot` then `Remove-VMSnapshot`
- [ ] Delete VHDX: `Remove-Item` on `.vhdx` files
- [ ] Delete virtual switches: `Get-VMSwitch` then `Remove-VMSwitch`
- [ ] Remove Hyper-V configurations as needed

---

## 7. Active Directory Cleanup

- [ ] Identify AD objects: `Get-ADObject`
- [ ] Remove users: `Remove-ADUser`
- [ ] Remove groups: `Remove-ADGroup`
- [ ] Remove service accounts: `Remove-ADUser`
- [ ] Remove OUs: `Remove-ADOrganizationalUnit`
- [ ] Remove computer accounts: `Remove-ADComputer`
- [ ] Verify removal: `Get-ADObject`

---

## 8. Group Policy Cleanup

- [ ] Inventory GPOs: `Get-Gpo -All`
- [ ] Document GPOs (optional)
- [ ] Remove GPO links: `Get-GPLink` then unlink
- [ ] Remove GPOs: `Remove-Gpo`
- [ ] Remove WMI filters: `Get-WmiFilter` and remove
- [ ] Verify: `Get-Gpo -All` should return nothing

---

## 9. DNS Cleanup

```powershell
Get-DnsServerZone
```
- [ ] Remove zones and records
- [ ] Remove stale records
- [ ] Verify with `Get-DnsServerZone`

---

## 10. Domain Controller Demotion

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Uninstall-ADDSDomainController -DemoteOperationMasterRole -Force
```
- Verify domain health before demotion
- Confirm successful demotion
- Clean metadata if needed

---

## 11. Remove Server Roles

```powershell
Uninstall-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
Uninstall-WindowsFeature -Name DNS -IncludeManagementTools
Uninstall-WindowsFeature -Name Hyper-V -IncludeManagementTools
```
- Verify with `Get-WindowsFeature`

---

## 12. System Cleanup

- [ ] Remove temporary files
- [ ] Remove logs (if appropriate)
- [ ] Remove custom scripts
- [ ] Remove scheduled tasks
- [ ] Clean administrator content
- [ ] Remove custom shares

---

## 13. Verification Phase

- [ ] No virtual machines remain (`Get-VM`)
- [ ] No VHDX files remain (`Get-ChildItem *.vhdx`)
- [ ] No Active Directory remains (`Get-ADDomain` fails)
- [ ] No GPOs remain (`Get-Gpo` empty)
- [ ] No DNS zones remain (`Get-DnsServerZone` empty)
- [ ] No applications remain
- [ ] No lab files remain
- [ ] No lab shares remain
- [ ] No project artifacts remain

---

## 14. Final Shutdown and Retirement

- Final validation complete
- Screenshots required
- Shutdown: `Stop-Computer`
- Storage retirement as required
- Document retirement date

---

## 15. Troubleshooting Guide

| Issue | Symptoms | Resolution |
|-------|----------|------------|
| Demotion fails | AD role still present | Verify no replication partners needed |
| VM removal fails | File locked | Stop VM services first |
| GPO removal fails | Linked GPO | Unlink before removal |
| DNS cleanup fails | Active records | Stop DNS service temporarily |

---

## Decommissioning Completion Checklist

- [ ] All applications removed
- [ ] All user files removed
- [ ] All project files removed
- [ ] All virtual machines removed
- [ ] All checkpoints removed
- [ ] All VHDX files removed
- [ ] All Hyper-V resources removed
- [ ] All Group Policy Objects removed
- [ ] All DNS entries removed
- [ ] All Active Directory objects removed
- [ ] Domain Controller successfully demoted
- [ ] Active Directory Domain Services removed
- [ ] DNS role removed
- [ ] Hyper-V role removed
- [ ] No remnants of lab activities remain
- [ ] Server ready for retirement
