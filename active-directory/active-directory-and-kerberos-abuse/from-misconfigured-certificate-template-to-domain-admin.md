# From Misconfigured Certificate Template to Domain Admin

## From Misconfigured Certificate Template to Domain Admin

This is a quick lab to familiarize with ECS1 privilege escalation technique, that illustrates how it's possible to elevate from a regular user to domain administrator in a Windows Domain by abusing over-permissioned Active Directory Certificate Services (ADCS) certificate templates.

This lab is based on [Certified Pre-Owned: Abusing Active Directory Certificate Services](https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf) whitepaper by [Will Schroeder](https://twitter.com/harmj0y) and [Lee Christensen](https://twitter.com/tifkin_) from [SpecterOps](https://specterops.io/).

### Finding Vulnerable Certificate Templates

Once in an AD environment, we can find vulnerable certificate templates by using `Certify`, a tool released by SpecterOps as part of their research mentioned above:

{% code title="attacker\@target" %}
```
certify.exe find /vulnerable
```
{% endcode %}

Below shows a snippet of the redacted output from `Certify`, that provides information about a vulnerable certificate:

![Vulnerable certificate template identified by Certify](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2FB3IQGDnjpZQ7sBFuS4sq%2Fvuln-template.png?alt=media\&token=b1bbc1c2-2b06-4674-b9eb-dcf9e6c68bff)

In the above screenshot, note the following 3 key pieces of information, that tell us that the certificate template is vulnerable and can be abused for privilege escalation from regular user to domain administrator:

* `msPKI-Certificates-Name-Flag: ENROLLEE_SUPPLIES_SUBJECT` field field, which indicates that the user, who is requesting a new certificate based on this certificate template, can request the certificate for another user, meaning any user, including domain administrator user.\
  \
  Below shows the same certificate template setting via GUI when inspecting certificate templates via `certsrv.msc`:\ <img src="https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2FlSycJqxClh9Mu0UwEGHP%2Fsuppy-in-request.png?alt=media&#x26;token=56524651-56c3-49b7-bf07-b637b814016a" alt="" data-size="original"><br>
*   `PkiExtendedKeyUsage: Client Authentication`, which indicates that the certificate that will be generated based on this certificate template can be used to authenticate to computers in Active Directory.\
    \
    Below shows the same setting via GUI when inspecting certificate templates via `certsrv.msc`:

    <img src="https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2FKf5mPwUetOfyorjJp53S%2Fclient-authentication.png?alt=media&#x26;token=3150eaa4-66ed-4914-85d9-bdb91c33c1d0" alt="" data-size="original"><br>
* `Enrollment Rights: NT Authority\Authenticated Users`, which indicates that any authenticated user in the Active Directory is **allowed to request** new certificates to be generated based on this certificate template.\
  \
  Below shows the same setting via GUI when inspecting certificate templates via `certsrv.msc`:\
  ![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2FUbTeSE1Keqjbt1X6d1St%2Fenroll-anyone.png?alt=media\&token=ce634afd-bf3a-48e5-b08b-3ff00904838c)

### Requesting Certificate with Certify

Once the vulnerable certificate template has been identified, we can request a new certificate on behalf of a domain administator using `Certify` by specifying the following parameters:

* `/ca` - speciffies the Certificate Authority server we're sending the request to;
* `/template` - specifies the certificate template that should be used for generating the new certificate;
* `/altname` - specifies the AD user for which the new certificate should be generated.

{% code title="attacker\@target" %}
```
certify.exe request /ca:<$certificateAuthorityHost> /template:<$vulnerableCertificateTemplateName> /altname:<$adUserToImpersonate>
```
{% endcode %}

Below shows that the certificate in `PEM` format has been issued successfully:

![New certificate was issued off of the vulnerable certificate template](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2FRjXQVCo6ikwaiLdjvOED%2Fimage.png?alt=media\&token=b2944e07-3d81-4e96-b643-f4014af87878)

### Converting PEM to PFX

As mentioned above, the certificate we just retrieved is in a `PEM` format.

To use it with a tool like `Rubeus` to request a Kerberos Ticket Granting Ticket (TGT) for the user for which we minted the certificate, we need to convert the certificate to `PFX` format.

To do this, copy the certificate content printed out by `Rubeus` and paste it to a file called `cert.pem`.

Then, convert it to `cert.pfx` with Open SSL (in Linux) like so:

{% code title="attacker\@target" %}
```
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```
{% endcode %}

### Requesting TGT with Certificate

Once we have the certificate in `cert.pfx`, we can request a Kerberos TGT for the user for which we minted the new certificate:

{% code title="attacker\@target" %}
```
Rubeus.exe asktgt /user:<$adUserToImpersonate> /certificate:cert.pfx /ptt
```
{% endcode %}

Below shows that a new TGT for the target user (Domain Admin in our case) using [Rubeus](https://github.com/GhostPack/Rubeus) was requested and injected in to the current logon session (because of the `/ptt`):

![Using rubeus to request a TGT for a user for which we minted the certificate](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2F6LNs6J7p9yoOCFNfYJ3G%2Ftgt-retrieved.png?alt=media\&token=23d3fcef-ac44-413b-bc31-248ce0430979)

At this point, we can test if we elevated our privileges to domain administrator by listing the administrative `c$` share on a server that we don't normally have local administrator privileges on:

![Listing a C$ share to confirm administrator access on a server](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2FVXZOM6jTlwriFT4Ouxz7%2Ftesting-access.png?alt=media\&token=b5188c0c-cf72-434b-9859-fde4c5672547)

### Bonus: Requesting Certificate Manually

This is a bonus section that shows how we can request a new certificate for a targeted user without Rubeus, but with a Certificate Signing Request (CSR) file crafted manually and later submitted to Active Directory Certificate Services self-service web portal.

#### Crafting Certificate Signing Request File

Create a new file `cert.cnf` with the following contents (modify fields as deemed appropriate):

{% code title="cert.cnf" %}
```
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
req_extensions     = req_ext
[ req_distinguished_name ]
countryName                 = GB
stateOrProvinceName         = State or Province Name (full name)
localityName               = Locality Name (eg, city)
organizationName           = Organization Name (eg, company)
commonName                 = Common Name (e.g. server FQDN or YOUR name)
[ req_ext ]
subjectAltName = otherName:1.3.6.1.4.1.311.20.2.3;UTF8:$adUserToImpersonate
```
{% endcode %}

The most important is line 12, which defines the `subjectAltName` field, which is a `samaccountname` of the user in Active Directory, which we want to ultimately impersonate (i.e. domain administrator) for which we will be requesting the certificate.\
\
`Samaccountname` value in this file is defined in the variable `$adUserToImpersonate` - you'd need to change it to the administrator's `samaacountname` you want to impersonate.

Once the `cert.cnf` file is ready, generate the actual Certificate Signing Request with `openssl` (in Linux):

```
openssl req -out cert-request.csr -newkey rsa:2048 -nodes -keyout key.key -config cert.cnf
```

Below shows how a base64 encoded Certificate Signing Request file `cert-request.csr` was created:

![Certificate Signing Request being generated with open ssl](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2Fc7E2pPuLZuJHgdqFpKDd%2Fimage.png?alt=media\&token=11a62b63-f947-4d59-92d4-ddf6b87c7a0f)

Now, copy the contents of the `cert-request.csr` as we will need it in the last step of this process as described below.

#### Requesting Certificate via CertSrv Web Portal

Navigate to `https://$adcs/certsrv`, where `$adcs` is the Active Directory Certificate Services host and click `Request a certificate`:

![Requesting certificates via ADCS web self service portal](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2FSfMISUmpCU88ri2u39pM%2Fimage.png?alt=media\&token=50a85fa8-5a62-4708-80bb-cdef3b67324c)

Click `advanced certificate request`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2F4oj4LoB5LBSfW3DkDh7z%2Fimage.png?alt=media\&token=28dbc3ed-9dcb-4dbd-9f99-7ee635feaa06)

Finally, select the vulnerable certificate template you want to base your new rogue certificate on, paste the contents of the `cert-request.csr` into the request field and hit `Submit` to retrieve the new certificate for your target user:

![Portal for submitting advanced certificate request](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LFEMnER3fywgFHoroYn%2Fuploads%2Fr6R7Enl3SQtqWwuV8YeW%2Fimage.png?alt=media\&token=5fe2d48b-e6ee-46b2-9703-c2f1620b0089)

### References

{% embed url="https://posts.specterops.io/certified-pre-owned-d95910965cd2" %}

{% embed url="https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf" %}
