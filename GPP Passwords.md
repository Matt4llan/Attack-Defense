# AS-REProasting Lab

## Objective

To abuse GPP Passwords, we will use the Get-GPPPassword function from PowerSploit, which automatically parses all XML files in the Policies folder in SYSVOL, picking up those with the cpassword property and decrypting them once detected. We will then find this attacl in Event viewer.

## What is GPP Passwords

SYSVOL is a network share on all Domain Controllers that contains logon scripts, group policy data, and other domain-wide data. Group Policy Preferences (GPP), introduced with Windows Server 2008, allow credentials to be stored in SYSVOL under \<DOMAIN>\SYSVOL<DOMAIN>\Policies. These credentials, often found in XML policy files, include usernames and encrypted passwords. The encryption key used by AD to encrypt these files is publicly available, allowing anyone to decrypt the stored credentials. Since the SYSVOL folder is accessible to all 'Authenticated Users' in the domain, both users and computers can access and potentially exploit this information.

>Microsoft published the AES private key on MSDN: https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be?redirectedfrom=MSDN

Here is an example XML file containing an encrypted password looks like (note that the property is called cpassword)

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/574ba130-5603-41b4-9b35-3d26194f5917)


## Skills Learned
- Run a GPP Password attack
- Filtering the Windows Event Viewer logs
- running Powershell Scripts

## Tools Used
- Windows Event Log
- Powershell
- Kali Linux
- Powersploit

## Attack

To start the attack we need to connect to our WIN10 box and run the powershell script 'Get-GPPPassword.ps1'


## Prevention



## Detection

### HoneyPot


### Investigate



