# Lateral Movement with Psexec

## Lateral Movement with Psexec

A very old and noisy lateral movement technique can be performed using psexec by SysInternals.

### Execution

Let's connect from workstation `ws01` to the domain controller `dc01` with domain administractor credentials:

{% code title="attacker\@victim" %}
```
.\PsExec.exe -u administrator -p 123456 \\dc01 cmd
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfLoxyuqgipOezzIB-W%2F-LfLqox9BY2R5jrZfQbi%2FAnnotation%202019-05-20%20210729.png?alt=media\&token=4b23bc50-80fe-4d71-b464-4dca83b8b374)

### Observations

The technique is noisy for at least a couple of reasons. Upon code execution, these are some well known artefacts that are left behind which will most likely get you flagged in an environment where SOC is present.

A `psexesvc` service gets created on the remote system and below shows the process ancestry of your command shell:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfLoxyuqgipOezzIB-W%2F-LfLrw4yml0xTbOQBU4j%2FAnnotation%202019-05-20%20211216.png?alt=media\&token=ddf1c60f-0180-4f0f-84a1-24693f2033de)

Proving that `psexec` is actually running as a service:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfLoxyuqgipOezzIB-W%2F-LfLsJKxFp9KSr65n3_r%2FAnnotation%202019-05-20%20211401.png?alt=media\&token=e1d6a809-8f5c-4b32-b9a4-66f1e77aee2a)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfLoxyuqgipOezzIB-W%2F-LfLt4CKasmC-RzgE9C2%2FAnnotation%202019-05-20%20211654.png?alt=media\&token=fb16f1c3-7876-4eac-8a4f-b1e26b25704a)

Additionally, there is quite a bit of SMB network traffic generated when connecting to a remote machine which could be signatured:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfLtsy-vTrqvAGWKS01%2F-LfLu0NlioAMzhkd_Yvf%2FAnnotation%202019-05-20%20212123.png?alt=media\&token=37bbd1cd-07da-4547-9ac4-30286757688e)
