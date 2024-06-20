# Kerberoasting Lab

## Objective

Perform a Kerberoasting attack on the Windows 10 machine, find the user account 'svc-iam' then crack password on Kali Linux.
Connect to DC1 and investigate Event Manager to find the attack. 

## What is Kerberoasting

Kerberoasting is a post-exploitation attack in Active Directory where attackers exploit the Service Principal Name (SPN) behavior in Kerberos authentication. By requesting a Kerberos TGS service ticket, which is encrypted with the service account's NTLM password hash, attackers can perform offline password cracking. If the ticket is successfully cracked, it reveals the service account's password. The effectiveness of Kerberoasting depends on the strength of the service account's password and the encryption algorithm used, with AES being more secure but slower to crack compared to RC4 and DES.

## Skills Learned
- Run a Kerberoasting attack
- Filtering the Windows Event Viewer logs
- Cracking spn.txt files with John The Ripper and Hashcat

## Tools Used
- Windows Event Log
- John The Ripper
- Hashcat
- Rubeus
- Powershell
- Kali Linux

## Attack

First we need to RDP to the windows machine

```
xfreerdp /u:(username) /p:(password) /v:(Machine IP) /dynamic-resolution
xfreerdp /u:(username) /p:(password) /v:(Machine IP) /dynamic-resolution /drive:share,/home/XXXXXXX
```

Next we need to open up a command prompt and CD to our downloads folder where the file we need to run in located.

```
cd C:\Users\bob\Downloads
```

Next we run Rubeus with the kerberoast action without specifying a user, it will extract tickets for every user that has an SPN registered

```
.\Rubeus.exe kerberoast /outfile:spn.txt
```

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/0fb964be-9b69-4f53-9647-f782f6ff7ca6)

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/242e4811-2275-4811-9368-9e803666e11c)

Now time to get cracking!! We need to move this file over to Kali to crack the password. We can either use the 'smbclient' command or we can specify the current user's home directory as a shared drive when we first connected.
I will use both Hashcat and John the ripper just for funzies.

### Hashcat

```
hashcat -m 13100 -a 0 spn.txt /usr/share/wordlists/rockyou.txt
```

__Command breakdown:__
__<br>hashcat:__   _This is the executable program that performs the password cracking._
__<br>-m 13100:__   _This specifies the hash type. The 13100 mode is for cracking Kerberos 5 TGS-REP (Ticket Granting Service Reply) tickets, which is often used in Active Directory environments._
__<br>-a 0:__   _This specifies the attack mode. The 0 mode indicates a dictionary attack, where Hashcat will use a list of potential passwords from a specified wordlist._
__<br>spn.txt:__   _This is the input file containing the hashes you want to crack. In this context, spn.txt likely contains Kerberos TGS-REP hashes._
__<br>/usr/share/wordlists/rockyou.txt:__   _This is the path to the wordlist file. The rockyou.txt file is a well-known wordlist that contains millions of common passwords and is often used in password cracking attacks._



![image](https://github.com/Matt4llan/HackTheBox-Kerberoasting/assets/156334555/e3079a53-f861-48d5-b76e-e24c2b76edae)
![image](https://github.com/Matt4llan/HackTheBox-Kerberoasting/assets/156334555/b442e61d-45e8-4bbe-a6ce-564923fe32d5)


### John The ripper

```
sudo john spn.txt --fork=4 --format=krb5tgs --wordlist=/usr/share/wordlists/rockyou.txt --pot=results.pot
```

![image](https://github.com/Matt4llan/HackTheBox-Kerberoasting/assets/156334555/a11d021f-4190-43cd-91aa-b85a2af71b46)

## Prevention

The success of password cracking attacks on service accounts depends on the password's strength. To mitigate this risk, limit the number of accounts with SPNs, disable unused ones, and enforce strong passwords, ideally 100+ random characters. Group Managed Service Accounts (GMSA) are highly recommended as they are automatically managed and have passwords rotated by Active Directory, though their use is limited to certain applications. Whenever possible, implement GMSAs and avoid assigning SPNs to unnecessary accounts, regularly cleaning up invalid SPNs.

## Detection

### HoneyPot

A honeypot user is an excellent detection option for an AD environment. This user should have no real purpose, ensuring no regular service tickets are generated. Any service ticket request for this account is likely malicious and worth investigating. To be effective, the account should be relatively old, have a strong but unchanged password (ideally 2+ years, preferably 5+), and possess some privileges to attract advanced adversaries. It should also have a registered SPN that appears legitimate, such as those for IIS or SQL accounts. Any activity involving this account, whether successful or failed logon attempts, should trigger an alert due to its suspicious nature.

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/cfaa4ad1-6a92-45bd-ad4a-2a55028acc73)


### Investigate

Lets RDP to the DC1 box and check out the Event Viewer and see if we can see the attack.

When a TGS is requested, AD generates event log ID 4769, but this ID is also generated for any user attempting to connect to a service, resulting in a high volume of logs that make detection difficult. However, if all applications in the environment support only AES and generate only AES tickets, event ID 4769 becomes a valuable alert indicator. If RC4 tickets are generated, which is not the default configuration, it should trigger an alert and warrant further investigation.

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/f85ef293-fe1d-4ad5-b0bd-6fc556e4a9d2)

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/49875951-1476-4afd-8137-9c1d7031428d)




