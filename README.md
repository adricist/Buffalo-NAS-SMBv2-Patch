🛠️ Enabling SMB2 on Buffalo LS-QVL/R5 NAS (Firmware v1.75) — A Step-by-Step Guide

Author: Adriano
Assisted by: Microsoft Copilot 🤖
Date: September 2025
System: Buffalo LinkStation LS-QVL/R5 running firmware v1.75
Goal: Enable SMB2 support to restore secure access from Windows 10/11 without relying on deprecated SMBv1

🔍 Background
Buffalo’s stock firmware v1.75 does not officially support SMBv2, causing Windows 10/11 to reject connections to shared folders. Rather than enabling insecure SMBv1 on the client side, this guide shows how to patch the NAS to negotiate SMB2—using ACP Commander and remote shell commands.

🧰 Tools Required
| Tool | Purpose | 
| ACP_Commander.jar | Remote command execution on Buffalo NA | 
| Java Runtime Environment (JRE) | Required to run ACP Commander | 
| Windows Command Prompt | Interface for sending commands | 
| File Explorer | To verify access and inspect output | 



🧭 Step-by-Step Instructions

1. Clear Root Password (Optional)
java -jar ACP_Commander.jar -t 191.168.0.101 -pw yourAdminPassword -cb


2. Assign a Valid Shell to Root
java -jar ACP_Commander.jar -t 191.168.0.101 -pw yourAdminPassword -c "usermod -s /bin/sh root"


3. Inject SMB2 Config Line via Startup Script
java -jar ACP_Commander.jar -t 191.168.0.101 -pw yourAdminPassword -c "/bin/sed -i '/nas_configgen -c samba/a\\    /bin/sed -i \"3i\\    max protocol = SMB2\" /etc/samba/smb.conf' /etc/init.d/smb.sh"


4. Restart Samba to Apply Changes
java -jar ACP_Commander.jar -t 191.168.0.101 -pw yourAdminPassword -c "/etc/init.d/smb.sh restart"


5. Verify the Patch
Dump the top of the config file into a shared folder:
java -jar ACP_Commander.jar -t 191.168.0.101 -pw yourAdminPassword -c "head -n 10 /etc/samba/smb.conf > /mnt/array1/NAS-Media/smbconf_head.txt"

NAS-Media is the name of one the shares I used. Adjust it to suit your case.

Open smbconf_head.txt from Windows Explorer and confirm:
max protocol = SMB2



✅ Final Result
You should now be able to access your NAS from Windows 10/11 using:
\\LS-QVL4C1\NAS-Media


No need to enable SMBv1. Your NAS now negotiates SMB2 securely.

🙌 Credit & Thanks
This solution was engineered with the help of Microsoft Copilot, who guided me through every step—from shell injection to Samba patching. If you're trying to keep legacy hardware alive without compromising security, this method works.
