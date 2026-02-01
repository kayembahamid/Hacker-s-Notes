# BloodHound with Kali Linux: 101

## What is BloodHound

> BloodHound is a single page Javascript web application, built on top of [Linkurious](http://linkurio.us/), compiled with [Electron](http://electron.atom.io/), with a [Neo4j](https://neo4j.com/) database fed by a PowerShell ingestor.
>
> BloodHound uses graph theory to reveal the hidden and often unintended relationships within an Active Directory environment. Attackers can use BloodHound to easily identify highly complex attack paths that would otherwise be impossible to quickly identify. Defenders can use BloodHound to identify and eliminate those same attack paths. Both blue and red teams can use BloodHound to easily gain a deeper understanding of privilege relationships in an Active Directory environment.
>
> BloodHound is developed by [@\_wald0](https://www.twitter.com/_wald0), [@CptJesus](https://twitter.com/CptJesus), and [@harmj0y](https://twitter.com/harmj0y).
>
> From [https://github.com/BloodHoundAD/BloodHound](https://github.com/BloodHoundAD/BloodHound)

## Installing BloodHound

It is surprisingly easy to install BloodHound on Kali Linux:

attacker@kali

```bash
apt-get install bloodhound
```

Part of the installation process will install the Neo4j database management solution required for BloodHound; Neo4j will need to be configured before use.

## Installation, configuration and run (stepper)

{% stepper %}
{% step %}
### Install

On Kali:

```bash
apt-get install bloodhound
```

This also installs Neo4j.
{% endstep %}

{% step %}
### Configure Neo4j

Start Neo4j console:

attacker@kali

```bash
neo4j console
```

Open http://localhost:7474/ and change the default Neo4j account password (default neo4j:neo4j). You will need these credentials when logging into BloodHound.

![Neo4j Setup](<../../.gitbook/assets/image (192)>)
{% endstep %}

{% step %}
### Run BloodHound

Start BloodHound:

attacker@kali

```bash
bloodhound
```

Log in with the Neo4j credentials you set earlier.

![BloodHound Login](<../../.gitbook/assets/image (193)>)
{% endstep %}
{% endstepper %}

## Enumeration & Data Ingestion

BloodHound is a data visualization tool — it requires enumerated Active Directory data to be useful. The enumeration process produces a JSON (zipped) file that describes relationships and permissions between AD objects. That JSON (zipped) file is imported into BloodHound, which then visualizes attack paths (for attackers) or privilege issues (for defenders).

## SharpHound (ingestor)

The AD enumeration ingestor for BloodHound is called SharpHound. See the project repository: https://github.com/BloodHoundAD/BloodHound/tree/master/Ingestors

If you run SharpHound from an account that is not a domain member or not authenticated as a domain user you may see errors like in the screenshot below.

![SharpHound error when not domain joined/authenticated](<../../.gitbook/assets/image (194)>)

If you are on a machine that is a domain member but are authenticated as a local user, and you have domain credentials, get a shell for that user:

attacker@victim

```powershell
runas /user:spotless@offense powershell

# if machine is not a domain member
runas /netonly /user:spotless@offense powershell
```

Run SharpHound from the victim machine (PowerShell ingestor):

attacker@victim

```powershell
. .\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All -JSONFolder "c:\experiments\bloodhound"
```

Invoke-BloodHound will produce a zipped JSON file.

![SharpHound output zip](<../../.gitbook/assets/image (195)>)

Drag-and-drop the generated .zip into the BloodHound GUI to ingest the data. After ingestion, use the built-in queries to visualize findings.

![BloodHound queries demo](<../../.gitbook/assets/image (196)>)

## Execution (using queries)

After ingestion you can run built-in queries such as All Domain Admins, Shortest Path to Domain Admins, etc., to help identify privilege escalation paths.

## Example — User to Exchange Trusted Subsystem

This contrived example shows how user offense\spotless could escalate to assume privileges of the Exchange Trusted Subsystem group when on the victim network:

![User to Exchange Trusted Subsystem example](<../../.gitbook/assets/image (197)>)

The example indicates:

* offense\spotless is admin of the DC01$ machine (possible to pass the machine account hash with Mimikatz and get an elevated shell)
* an offense\administrator session exists on DC01$ (dump LSASS or token impersonation)
* by combining these, an attacker could assume Exchange Trusted Subsystem privileges.

What is the Exchange Trusted Subsystem?

```
net group "Exchange Trusted Subsystem"
Group name     Exchange Trusted Subsystem
Comment        This group contains Exchange servers that run Exchange cmdlets on behalf of users via the management service. Its members have permission to read and modify all Exchange configuration, as well as user accounts and groups. This group should not be deleted.
```

## Example — User to Domain Admin via AdminTo and MemberOf

Another example where user spotless could become a Domain Admin: spotless is admin of DC01$ where an admin session is established. If that session is compromised, spotless becomes a Domain Admin.

![User to Domain Admin via AdminTo and MemberOf](<../../.gitbook/assets/image (198)>)

## Example — User to Domain Admin via Weak ACEs

This example shows how user spotless can become a Domain Admin by abusing weak ACEs on the Domain Admins group. In this case, spotless could add themselves to Domain Admins with:

```
net group "domain admins" spotless /add /domain
```

and gain Domain Admin privileges.

![User to Domain Admin via weak ACEs](<../../.gitbook/assets/image (199)>)

For manual exploration of AD ACL/ACE misconfigurations (AddMember, GenericWrite, GenericAll, etc.) see: [Abusing Active Directory ACLs/ACEs](abusing-active-directory-acls-aces.md)

## References

{% embed url="https://github.com/BloodHoundAD/BloodHound/wiki" %}
