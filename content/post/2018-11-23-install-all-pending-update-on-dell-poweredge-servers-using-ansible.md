---
title: "Install all firmware updates on Dell PowerEdge servers with Ansible"
date: 2018-11-23T19:37:00+01:00
draft: false
author: VaLouille
type: post
categories:
  - ansible
  - automation
tags:
  - Dell
  - ansible
  - poweredge
  - update
  - firmware
  - BIOS
  - drivers
  - linux
  - drm
  - repository
  - iDRAC
  - automation
  - nfs
  - api
  - OOB
---

> This feature is only available with iDRAC Enterprise License. 

The supported servers are : 

* 12G and 13G PowerEdge Servers: iDRAC 7 and 8 with Firmware version 2.50.50.50 and above
* 14G PowerEdge Servers: iDRAC 9 with Firmware version 3.18.18.18 and above

While looking once more for the most easy way to update Dell servers firmwares, I stepped on this awesome video showing someone from Dell use Ansible to update at once all the firmwares of a server :


{{% youtube "oagvLq6am3s" %}}

So, after some research on how to do it, here are all the steps needed to achieve this using only the CLI from a Linux Out-of-Band machine.

## Overview

In this guide, we'll install Dell EMC Repository Manager to create and maintain a repository containing the updates files for our servers and generate a Catalog.xml file that contains the relation informations. Then, we'll export these updates to a NFS share, and use Ansible + Dell EMC OpenManage Ansible Modules (which uses Dell EMC OpenManage Python SDK internally) to communicate with the iDRACs and trigger the updates that will be download from the NFS share.

> Since iDRACs will generate Server Configuration Profile files to the NFS share as the nobody user, Ansible must be run as root because it'll need the permission to edit these files.

## Prerequisites

### Dell EMC Repository Manager

We need to download Dell Repository Manager from Dell website and install it. It can be found as a .BIN file and installed just by executing it. It also works on Ubuntu/Debian, even if not stated by Dell.


```
chmod +x DRMInstaller_3.1.0.468.bin
./DRMInstaller_3.1.0.468.bin
```

The GUI didn't worked right away for me, as I got the following error while trying to launch it using `drm` command :
```
Graphics Device initialization failed for :  es2, sw
Error initializing QuantumRenderer: no suitable pipeline found
java.lang.RuntimeException: java.lang.RuntimeException: Error initializing QuantumRenderer: no suitable pipeline found
	at com.sun.javafx.tk.quantum.QuantumRenderer.getInstance(QuantumRenderer.java:280)
	at com.sun.javafx.tk.quantum.QuantumToolkit.init(QuantumToolkit.java:221)
	at com.sun.javafx.tk.Toolkit.getToolkit(Toolkit.java:248)
	at com.sun.javafx.application.PlatformImpl.startup(PlatformImpl.java:209)
	at com.sun.javafx.application.LauncherImpl.startToolkit(LauncherImpl.java:675)
	at com.sun.javafx.application.LauncherImpl.launchApplication1(LauncherImpl.java:695)
	at com.sun.javafx.application.LauncherImpl.lambda$launchApplication$154(LauncherImpl.java:182)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.lang.RuntimeException: Error initializing QuantumRenderer: no suitable pipeline found
	at com.sun.javafx.tk.quantum.QuantumRenderer$PipelineRunnable.init(QuantumRenderer.java:94)
	at com.sun.javafx.tk.quantum.QuantumRenderer$PipelineRunnable.run(QuantumRenderer.java:124)
	... 1 more
No toolkit found
```

I needed to install the package `libswt-gtk-3-java` to fix this issue.

But in this configuration, we try to do everything from the CLI, so that doesn't really matter.

#### Optional 
Plugins can be installed in order to create bootable ISO including the firmwares, concatenated BIN files, etc... We can list the available plugins with the following command :
```
drm --list=plugin

Listing Plugins...


Plugin                                       Current Version   Latest Version
--------------                               --------------    --------------
Dell EMC System Update(BIN)                  1.5.3             1.5.3
Dell EMC Bootable ISO Plug-in                902.1             902.1
Dell EMC Server Update Utility x64 Plug-in   782               782
Dell EMC System Update(EXE)                  1.5.3             1.5.3
```

To install a plugin, we can use the following command : `drm --update --plugin="Dell EMC System Update(EXE)"`. 

### Dell EMC OpenManage Python SDK
Ansible modules require that [Dell EMC OpenManage Python SDK](https://github.com/dell/omsdk) is installed on the system before being able to get installed. So here are the few steps to do it :
```
git clone https://github.com/dell/omsdk.git
sudo apt-get install python-pip
cd omsdk
sudo -H pip install -r requirements-python2x.txt # use requirements-python3x.txt if using Python 3
sh build.sh 1.2 345
cd dist
sudo -H pip install omsdk-1.2.345-py2.py3-none-any.whl
```
> Please refer to the project [GitHub page](https://github.com/dell/omsdk) to install the latest version

### Ansible modules 

We need to install the [Dell-EMC-Ansible-Modules-for-iDRAC](https://github.com/dell/Dell-EMC-Ansible-Modules-for-iDRAC) for our playbook to work :
```
git clone https://github.com/dell/Dell-EMC-Ansible-Modules-for-iDRAC.git
cd Dell-EMC-Ansible-Modules-for-iDRAC
sudo -H python install.py
```

### NFS server

A NFS share will be used by the iDRAC to access the latest firmware versions and install them onto the machine. It'll also be used by the iDRACs to create a Server Configuration Profile file and by Ansible to modify these files. 

A external NFS volume can be used but for the previously stated reasons it'll need to be acessible from the iDRACs and mounted locally on the server that runs Ansible.

For creating an NFS share, we can proceed like this :

> This is not a complete secure configuration of NFS server, please us it for POC only

```
sudo apt-get install rpcbind nfs-kernel-server
```

Then we create a folder for the NFS share :

```
mkdir -p /opt/dell/dellemcrepositorymanager/export
chmod 777 /opt/dell/dellemcrepositorymanager/export
```

Then, modify the /etc/exports file to add the share (adapt the IP to the one from your network) :
```
/opt/dell/dellemcrepositorymanager/export 10.220.0.0/25(rw,fsid=0,insecure,no_subtree_check,async)
```

The service needs to be rebooted for the share to become available :
```
service nfs-kernel-server restart
```

We can check access from the iDRAC using the `Test network connection` button in `Maintenance`, `System Update`, `Manual Update` after changing the `Location Type` to `Network Share` and entered the infos about the IP address of the NFS server and the Share Name.

## Export of the DUPs to NFS share

Once Dell Repository Manager is installed, we must configure it with a repository and export the Dell EMC Update Packages to the NFS share. This can be done using the following commands.

First, let's create the repository. Be aware that if you don't filter on the servers, hundred of gigabytes will be downloaded !
```
$ drm --create --repository=Dell --inputplatformlist="R640,R740xd"

Creating repository Dell
Job Submitted for repository creation
```

We can check the status with the following command :
```
drm --repository=Dell --details

Listing version information for: Dell
Version   Size      Date Created
-------   ----      ------------
1.00      5.63 GB   11/23/18 12:24 A.M
```

The firmwares will be downloaded in /var/dell/drm/store/

We can check the progress using the following command :
```
drm --list=job

Listing Jobs...


Name                                                          Status            Job Type              Next Execution date/time   Last Execution date/time
----                                                          ------            --------              ------------------------   ------------------------
Download_11/23/2018_11:04                                     SUCCESS           DOWNLOAD COMPONENTS   NA                         11/23/18 11:11 A.M
Plugin Download- Dell EMC Server Update Utility x64 Plug-in   SUCCESS           PLUGIN_UPDATE         NA                         11/23/18 11:12 A.M
Plugin Download- Dell EMC Bootable ISO Plug-in                SUCCESS           PLUGIN_UPDATE         NA                         11/23/18 11:27 A.M
Plugin Download- Dell EMC System Update(BIN)                  SUCCESS           PLUGIN_UPDATE         NA                         11/23/18 11:28 A.M
Plugin Download- Dell EMC System Update(EXE)                  SUCCESS           PLUGIN_UPDATE         NA                         11/23/18 11:28 A.M
```

Once everything is downloaded, let's export the firmware binaries and the Catalog.xml file. The Catalog.xml files contains informations about the firmwares, and is used by the iDRAC to determine the updates to install.


```
drm --deployment-type=export --repository=Dell:1.00 --location=/opt/dell/dellemcrepositorymanager/export

Job Submitted for export
```
The location can be an external NFS, in this case, we need to mount it locally of specify it with the `--location` command.


Once again, let's check the progress using the following command :
```
drm --list=job

Listing Jobs...


Name                                                          Status            Job Type              Next Execution date/time   Last Execution date/time
----                                                          ------            --------              ------------------------   ------------------------
Download_11/23/2018_11:04                                     SUCCESS           DOWNLOAD COMPONENTS   NA                         11/23/18 11:11 A.M
Plugin Download- Dell EMC Server Update Utility x64 Plug-in   SUCCESS           PLUGIN_UPDATE         NA                         11/23/18 11:12 A.M
Plugin Download- Dell EMC Bootable ISO Plug-in                SUCCESS           PLUGIN_UPDATE         NA                         11/23/18 11:27 A.M
Plugin Download- Dell EMC System Update(BIN)                  SUCCESS           PLUGIN_UPDATE         NA                         11/23/18 11:28 A.M
Plugin Download- Dell EMC System Update(EXE)                  SUCCESS           PLUGIN_UPDATE         NA                         11/23/18 11:28 A.M
Export_11/23/2018_12:42                                       RUNNING           SHARE                 NA                         NA
```
Let's wait for it to have the `SUCCESS` status before running Ansible.

### Ansible playbook

The Ansible playbook should look like this :

```
---
- hosts: hosts
  connection: local
  name: Update Firmware Inventory
  gather_facts: False

  tasks:
  - name: Update Firmware Inventory
    dellemc_install_firmware:
       idrac_ip: "{{ inveotry_hostname }}"
       idrac_user: "ansible"
       idrac_pwd: "*******"
       share_name: "10.220.0.10:/opt/dell/dellemcrepositorymanager/export"
       share_mnt: "/opt/dell/dellemcrepositorymanager/export"
       catalog_file_name: "Dell_1.00_Catalog.xml"
       reboot: True
       job_wait: True
    tags :
       - installfirmware
```
The share_mnt option can be either the local NFS export directory on the local server, or the local mount of the external NFS volume used. As a Server Configuration Profile file a generated by the server and copied to the NFS share, it must be accessible locally. 

Other options can be provided as specified in the [related documentation](https://github.com/dell/Dell-EMC-Ansible-Modules-for-iDRAC/blob/master/guides/OMAM_1.1_Users_Guide.pdf)

Let's create an inventory file also :
```
# inventory file
---
[hosts]
ipmi-01.example.com
ipmi-02.example.com
```

### Running Ansible !
Once the export is successfully completed, we can run the playbook ! 

> Please be careful that it'll reboot the host to apply the update if needed as we specified `reboot: True` in the playbook

```
sudo -H ansible-playbook dellemc_install_firmware.yml -i inventory -t installfirmware -vvvvv
```

> It'll take a really long time to load and apply the updates, and it can be up to 1 hour. We also need to wait as long as it takes before the updates appears in the iDRAC logs and Job queue.

After this long wait, here is what we can see :

```
changed: [ipmi-01.example.com] => {
    "changed": true,
    "invocation": {
        "module_args": {
            "catalog_file_name": "Dell_1.00_Catalog.xml",
            "idrac_ip": "ipmi-01.example.com",
            "idrac_port": 443,
            "idrac_pwd": "VALUE_SPECIFIED_IN_NO_LOG_PARAMETER",
            "idrac_user": "root",
            "job_wait": true,
            "reboot": true,
            "share_mnt": "/opt/dell/dellemcrepositorymanager/export",
            "share_name": "10.220.0.10:/opt/dell/dellemcrepositorymanager/export",
            "share_pwd": null,
            "share_user": null
        }
    },
    "msg": {
        "@odata.context": "/redfish/v1/$metadata#DellJob.DellJob",
        "@odata.id": "/redfish/v1/Managers/iDRAC.Embedded.1/Jobs/JID_430165520584",
        "@odata.type": "#DellJob.v1_0_0.DellJob",
        "CompletionTime": "2018-11-23T18:25:58",
        "Description": "Job Instance",
        "EndTime": null,
        "Id": "JID_430165520584",
        "JobState": "Completed",
        "JobType": "ImportConfiguration",
        "Message": "Successfully imported and applied Server Configuration Profile.",
        "MessageArgs": [],
        "MessageId": "SYS053",
        "Name": "Import Configuration",
        "PercentComplete": 100,
        "StartTime": "TIME_NOW",
        "Status": "Success",
        "TargetSettingsURI": null,
        "retval": true
    }
}
```
