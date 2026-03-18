# User-Provisioning-Automation

## Objective
The User Provisioning Automation project established a PowerShell-based Identity and Access Management automation system built on top of the Active Directory infrastructure deployed in Lab 1. Running on a Windows Server 2022 Domain Controller hosted in Proxmox VE on a Dell PowerEdge T340, three automation scripts were developed to handle the full employee account lifecycle — onboarding, offboarding, and auditing. The goal was to demonstrate how IAM teams automate repetitive identity tasks to reduce human error, enforce consistent security standards, and maintain a clean and auditable Active Directory environment.



### Skills Learned

- Writing PowerShell automation scripts to manage Active Directory user accounts at scale
- Automating user provisioning from CSV input files to simulate HR system integration
- Implementing duplicate detection logic to ensure idempotent script execution
- Automating user offboarding including account disabling, group removal, and OU relocation
- Generating stale account audit reports to identify inactive accounts that pose security risks
- Understanding and applying the principle of least privilege through role-based account creation
- Configuring forced password change on first login as a security best practice
- Organizing PowerShell projects with proper folder structures and documentation
- Understanding the IAM account lifecycle from provisioning through deprovisioning


### Tools Used

- Windows Server 2022 as the Active Directory Domain Controller environment
- PowerShell 5.1 for all automation scripting
- Active Directory Users and Computers (ADUC) for visual verification of script results
- PowerShell ISE as the script development and testing environment
- Remote Desktop Protocol (RDP) for secure remote access to DC01 from the desktop
- Proxmox VE 9.1 running on Dell PowerEdge T340 as the hypervisor platform

## Steps

### Step 1 – User Onboarding Automation

The onboard.ps1 script was created and executed against a CSV file of new employees stored at C:\IAM-Scripts\new_users.csv. The script reads each employee record, generates a username from the first initial and last name, maps the department to the correct Organizational Unit, checks for duplicate accounts to prevent errors on repeat runs, and creates the user account with a temporary password of Welcome123! with forced password change on first login.
On the first run all three employees were successfully created — James Wilson (jwilson) in the Helpdesk OU, Emma Davis (edavis) in HR Department, and Robert Taylor (rtaylor) in Finance Department. On the second run the script demonstrated idempotent behaviour by skipping the three existing accounts and only creating the new employee Omeed Nazemi (onazemi) who had been added to the CSV. Active Directory Users and Computers was used to verify all users appeared in their correct OUs.
<img width="1000" height="800" alt="Audit Script Ran in Powershell" src="https://github.com/user-attachments/assets/af0c682f-94fa-4ce9-9d05-3b87a99d08b2" />


---

### Step 2 – User Offboarding Automation

The offboard.ps1 script was created and executed to simulate an employee departure. The username onazemi was entered as the employee to offboard. The script disabled the account preventing further logins, removed the user from all Active Directory groups, and relocated the account to the Disabled Accounts OU for archiving. Active Directory Users and Computers was opened to verify the account was successfully moved to the Disabled Accounts OU with a disabled status, confirming the offboarding completed correctly.
<img width="1000" height="800" alt="Offboarding Script Ran in Powershell" src="https://github.com/user-attachments/assets/e98e119d-f973-4577-8e53-21548d5b6b38" />

---

### Step 3 – Stale Account Audit

The audit.ps1 script was created and executed to identify accounts that had not been used in the past 90 days. The script scanned all enabled accounts in Active Directory and identified 5 stale accounts — Lisa Wong, Mike Johnson, James Wilson, Emma Davis, and Robert Taylor — all with a last logon date of December 18 2025. The results were displayed in a formatted table in the console and automatically exported to C:\IAM-Scripts\stale_accounts_report.csv for review. The CSV report and the script source code in Notepad are visible confirming the full audit pipeline worked end to end.


<img width="1000" height="800" alt="Onboarding Script Ran in Powershell" src="https://github.com/user-attachments/assets/d4ca4208-e916-409a-a258-aa13c55bac27" />




## Challenges & Lessons Learned

- RDP access for script development — The noVNC console does not support clipboard paste which made typing long scripts difficult. Enabling RDP through Server Manager's Local Server settings resolved this and is now the preferred method for all administrative work on DC01.
- Idempotency is critical — The onboarding script safely handles duplicate accounts by checking with Get-ADUser before creating each user. This means the script can be run multiple times without failing — only new employees in the CSV get created.
- Forced password change vs hardcoded passwords — Using -ChangePasswordAtLogon $true ensures the temporary password is changed by the user on first login. IAM teams should never know employees' final passwords.
- Stale accounts are a real security risk — Accounts that are enabled but unused represent attack surface. In a real environment the audit script would run on a schedule and flag accounts for review and automatic disabling.
- Key takeaway: Automating the IAM lifecycle — provisioning, deprovisioning, and auditing — is core to enterprise security. Manual account management at scale is error prone and creates orphaned accounts that attackers can exploit.
