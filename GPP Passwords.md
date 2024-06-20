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

```
xfreerdp /u:(username) /p:(password) /v:(Machine IP) /dynamic-resolution
xfreerdp /u:(username) /p:(password) /v:(Machine IP) /dynamic-resolution /drive:share,/home/XXXXXXX
```

Next we need to open up a command prompt and CD to our downloads folder where the file we need to run in located.

```
cd C:\Users\bob\Downloads
```

Next we will import the module 'Get-GPPPassword.ps1'

```
Import-Module .\Get-GPPPassword.ps1
Get-GPPPassword
```

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/0dc95f43-7983-40e7-8694-70561d406aaf)

We can see the output of the command below with the username and password in clear text

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/beaa513a-4fc1-40f4-8a9f-cb0d420273e3)


## Prevention

Microsoft addressed the issue of credentials being exposed in Group Policy Preferences (GPP) by releasing patch KB2962486 in 2014, which prevents new credentials from being cached in SYSVOL. However, many Active Directory environments created after 2015 still inadvertently store credentials in SYSVOL. It's essential for organizations to regularly review their environments to ensure no credentials are exposed. Note that environments built before 2014 likely still have cached credentials, as the patch does not remove existing stored credentials but only prevents new ones from being cached.

## Detection

### HoneyPot

This attack leverages a semi-privileged user with an intentionally incorrect password as bait, typically using service accounts for their infrequent password changes and predictable activity patterns. By ensuring the user's password is older than the modification date of the GPP XML file and scheduling dummy tasks to provoke recent logon attempts, any unexpected successful or failed logon attempts trigger alerts. Event IDs 4625, 4771, and 4776 are key indicators, primarily signaling failed logon attempts due to the incorrect password, highlighting potential malicious activity.

Screen shote of the 3 Event ID's

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/a52f869b-a9df-476f-b6d6-b0b9b91bee15)

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/a3f01700-77ac-4453-9775-f08fed968292)

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/9efad73c-f0f1-4917-af8e-996cc3e9d484)

### Investigate

There are two detection techniques for this attack

1. Auditing access to XML files containing credentials can serve as a red flag for suspicious activity. Using a dummy XML file not linked to any GPO enhances detection, as there is no legitimate reason to access it. Tools like Get-GPPPasswords parse all XML files in the Policies folder. For effective monitoring, generate an event whenever a user reads the dummy file, as any such attempt is likely suspicious.
We can filter the results with Event ID 4663 - Event ID 4663 is generated when an attempt is made to access an object, such as a file, folder, registry key, or other system object, that is being audited for access.

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/e07040ee-9e64-4e21-9aca-b99b0a7d89cf)

2. Monitoring logon attempts of the user whose credentials are exposed is a key method for detecting this attack. Events to watch for include 4624 (successful logon), 4625 (failed logon), and 4768 (TGT requested). A successful logon from this attack would generate a specific event on the Domain Controller, indicating potential abuse.

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/a9b15758-ea43-4733-b04e-eb59c25a70b0)

For service accounts, detecting abnormal logon attempts can be achieved by correlating them with the originating device. Since service accounts typically log on from specific locations, any logon attempt from a workstation is unusual and should be investigated.
