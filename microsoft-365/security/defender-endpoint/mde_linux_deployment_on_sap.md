Documentation for MDE for SAP on Linux
  

This page to be put somewhere here https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/?view=o365-worldwide 

 

SAP Applications on Linux - Defender for Endpoint Deployment Guide  

Conventional security defenses that have been commonly used to protect SAP systems such as isolating infrastructure behind firewalls and limiting interactive Operating System logons are no longer considered sufficient to mitigate modern sophisticated threats. It is essential to deploy modern defenses that will detect and contain threats in real-time.   

 

SAP applications unlike most other workloads require basic assessment and validation before deploying Defender for Endpoint. The Enterprise Security administrators should contact the SAP Basis team prior to deploying Defender. The SAP Basis Team should be cross trained with a basic level of knowledge about Defender.  

 

Recommended SAP OSS Notes 

2248916 - Which files and directories should be excluded from an antivirus scan for SAP BusinessObjects Business Intelligence Platform products in Linux/Unix? - SAP ONE Support Launchpad 

1984459 - Which files and directories should be excluded from an antivirus scan for SAP Data Services - SAP ONE Support Launchpad 

2808515 - Installing security software on SAP servers running on Linux - SAP ONE Support Launchpad 

1730930 - Using antivirus software in an SAP HANA appliance - SAP ONE Support Launchpad 

1730997 - Unrecommended versions of antivirus software - SAP ONE Support Launchpad 

 

SAP Applications on Linux: 

SAP only supports Suse, Redhat and Oracle Linux.  Other distributions are not supported for SAP S4 or NetWeaver applications.   

Suse 15.x, Redhat 8.x or 9.x and Oracle Linux 8.x are strongly recommended.   

Suse 12.x, Redhat 7.x and Oracle Linux 7.x are technically supported but have not been extensively tested 

Suse 11.x, Redhat 6.x and Oracle Linux 6.x may not be supported and have not been tested.   

Suse and Redhat offer specifically tailored distributions for SAP.  These “for SAP” versions of Suse and Redhat may have different packages preinstalled and possibly different kernels 

SAP only support certain Linux Filesystems. In general, XFS and EXT3 are used.  Oracle ASM filesystem is sometimes used for Oracle DBMS and cannot be read by MDE 

Some SAP applications use “standalone engines” such as TREX, Adobe Document Server, Content Server and LiveCache.  These engines require specific configuration and file exclusions  

SAP applications often have Transport and Interface directories with many thousands of small files.  If the number of files is larger than 100,000 then this may impact performance.  It is generally recommended to archive files  

It is strongly recommended to deploy MDE to non-productive SAP landscapes for several weeks before deploying to production.  The SAP Basis team should use tools such as sysstat, KSAR and nmon to verify if CPU and other performance parameters are impacted  

 

 

Prerequisites for MDE on SAP on Linux  

Prerequisites for deploying MDE for Linux on SAP VMs:   

MDE version >= 101.23082.0009 | Release version: 30.123082.0009 or higher must be deployed.  Do not use lower releases  

MDE for 1Linux supports all the Linux releases used by SAP applications  

MDE for Linux requires connectivity to specific Internet endpoints from VMs to update AV Definitions 

MDE for Linux requires some crontab (or other task scheduler) entries to schedule scans, log rotation and MDE updates.  Enterprise Security team will normally manage these entries How to schedule an update of the Microsoft Defender for Endpoint (Linux) | Microsoft Learn 

 

 

The default configuration option for deployment as an Azure Extension for AntiVirus (AV) will be Passive Mode.  This means that the AV component of MDE will not intercept IO calls.  It is recommended to run MDE in Passive Mode on all SAP applications and to schedule a scan once per day.  

 

The Linux crontab is typically used to schedule MDE AV scan and log rotation tasks 

How to schedule scans with Microsoft Defender for Endpoint (Linux) | Microsoft Learn 

 

EDR functionality will be Active whenever MDE for Linux is installed. There is no simple way to disable EDR functionality through command line or configuration.  

 

See the section “Checklist for Troubleshooting Problems on SAP VMs Running MDE on Linux” for more information on troubleshooting EDR 

 

Important Configuration Settings for MDE on SAP on Linux  

It is recommended to check the installation and configuration of MDE with the command mdatp health 

 

The key parameters recommended for SAP applications are: 

healthy = true 

release_ring = Production.  Pre-release and insider rings should not be used with SAP Applications 

real_time_protection_enabled = false. This prevents realtime IO interception  

automatic_definition_update_enabled = true 

definition_status = “up_to_date”.  Run a manual update if another value is seen 

edr_early_preview_enabled = “disabled”.  Do not enable on SAP systems.  This may lead to system instability 

conflicting_applications = [ ].  Other AV or security software installed on a VM such as Clam 

supplementary_events_subsystem = "ebpf".  Do not proceed if ebpf is not displayed.  Contact the security admin team 

 

This article has some useful hints on troubleshooting installation issues for MDE: 

Troubleshoot installation issues for Microsoft Defender for Endpoint on Linux | Microsoft Docs 

 

MDE AntiVirus Exclusions for SAP on Linux   

Enterprise Security Team must obtain a full list of AV exclusions from the SAP Administrators (typically the SAP Basis Team)   

 

It is recommended to initially exclude: 

DBMS data files, log files and temp files, including disks containing backup files 

The entire contents of the SAPMNT directory 

The entire contents of the SAPLOC directory 

The entire contents of the TRANS directory 

The entire contents of directories for standalone engines such as TREX 

Hana – exclude /hana/shared, /hana/data and /hana/log - see Note 1730930 

SQL Server – Configure antivirus software to work with SQL Server - SQL Server | Microsoft Learn 

Oracle – See How To Configure Anti-Virus On Oracle Database Server (Doc ID 782354.1) 

DB2 – https://www.ibm.com/support/pages/which-db2-directories-exclude-linux-anti-virus-software  

SAP ASE – contact SAP 

MaxDB – contact SAP 

 

Oracle ASM systems do not need exclusions as MDE cannot read ASM disks. 

 

Customers with Pacemaker clusters should also configure these exclusions: 

mdatp exclusion folder add --path /usr/lib/pacemaker/  (for RedHat /var/lib/pacemaker/) 

mdatp exclusion process add --name pacemakerd 

mdatp exclusion process add --name crm_* 

 

Customers running the Azure Security security policy may trigger a scan using the Freeware Clam AV solution.  It is recommended to disable Clam AV scan after a VM has been protected with MDE: 

sudo azsecd config  -s clamav -d "Disabled" 

sudo service azsecd restart 

sudo azsecd status 

 

This article details how to configure AV exclusions for processes, files and folders per individual VM 

Set up exclusions for Microsoft Defender Antivirus scans | Microsoft Learn 

Common mistakes to avoid when defining exclusions | Microsoft Learn 

 

 

Scheduling a Daily AV Scan 

The recommended configuration for SAP applications disables real-time interception of IO calls for AV scanning.  The recommended setting is real_time_protection_enabled = false 

The following link details how to schedule a scan How to schedule scans with Microsoft Defender for Endpoint (Linux) | Microsoft Learn 

Large SAP systems may have more than 20 SAP application servers each with a connection to the SAPMNT NFS share.  20 or more application servers simultaneously scanning the same NFS server will likely overload the NFS server.  By default MDE for Linux will not scan NFS sources.    

If there is a requirement to scan SAPMNT then this scan should be configured on only one or two VMs only.  

Scheduled scans for SAP ECC, BW, CRM, SCM, Solution Manager and other components should be staggered at different times to avoid all SAP components from overloading a shared NFS storage source shared by all SAP components. 

 

Useful Commands & Links 

 

During manual zypper installation on Suse an error “Nothing provides ‘policycoreutils’”  

Troubleshoot installation issues for Microsoft Defender for Endpoint on Linux | Microsoft Docs 

 

There are several command-line commands that can control the operation of mdatp.  To turn off real-time protection, you can use the command: 

mdatp config real-time-protection --value disabled 

 

This command will tell mdatp to retrieve the latest definitions from the cloud: 

mdatp definitions update 

 

This command will test whether mdatp can connect to the cloud-based endpoints via the network: 

mdatp connectivity test 

 

These commands will update the mdatp software if needed: 

yum update mdatp 

zypper update mdatp  

 

Since mdatp runs as a linux system service, you can control mdatp using the service command, eg: 

service mdatp status 

 

sudo mdatp diagnostic create (this command creates a diagnostic file that can be uploaded to Microsoft support) 

 

 

Useful Links  

Microsoft Endpoint Manager does not support Linux at this time Manage Microsoft Defender for Endpoint configuration settings on devices with Microsoft Endpoint Manager | Microsoft Learn 

 

Microsoft Defender for Endpoint Linux - Configuration and Operation Command List - Microsoft Tech Community 

 

Deploying Microsoft Defender for Endpoint on Linux Servers. - Microsoft Tech Community 

 

Troubleshoot cloud connectivity issues for Microsoft Defender for Endpoint on Linux | Microsoft Docs 

Troubleshoot performance issues for Microsoft Defender for Endpoint on Linux | Microsoft Docs 

 

 
