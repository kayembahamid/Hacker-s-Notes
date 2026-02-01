---
description: Persistence
---

# Create Account

## Create Account

### Execution

{% code title="attacker\@victim" %}
```bash
net user test test123 /add /domain
```
{% endcode %}

### Observations

![commandline arguments](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LINfMVZoMtDoyXu6qvv%2F-LINfFrXDDzNDHs8iJRT%2Faccount-add.png?alt=media\&token=98ffc3c1-28f0-4052-ba57-87d6f1c2e728)

There is a whole range of interesting events that could be monitored related to new account creation:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LINfMVZoMtDoyXu6qvv%2F-LINfIDy9G7rtngKs27R%2Faccount-events.png?alt=media\&token=f983d294-10fa-4adc-ba39-57d5bc95bd95)

Details for the newly added account are logged as event `4720` :

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LINfMVZoMtDoyXu6qvv%2F-LINfJnvoffrohsKGUrk%2Faccount-created.png?alt=media\&token=12c8739c-c9a8-4c65-bf1f-0a9cd6525135)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1136" %}
