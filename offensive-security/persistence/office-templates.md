# Office Templates

## Office Templates

It's possible to persist in the userland by abusing Microsof templates - documents that are used as base templates for all new documents created by Office. In this lab, I am abusing Ms Word templates.

### Weaponization

Let's open and edit the base template called `Normal` that can be found at:

```
C:\Users\mantvydas\AppData\Roaming\Microsoft\Templates
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Li2x2sfJkV6f8_MjS1N%2F-Li2zsDQ05O2kciVhYsE%2FAnnotation%202019-06-23%20120121.png?alt=media\&token=871e3903-b1f1-4f8c-8e48-3010b736b014)

Create a new AutoOpen macro and add your VBA code there:

```javascript
Sub AutoOpen()
MsgBox "Ohai from the template :)"
End Sub
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Li2x2sfJkV6f8_MjS1N%2F-Li30PlZTpOU4c5yucuZ%2FAnnotation%202019-06-23%20120805.png?alt=media\&token=575e8429-d29b-4dde-907e-f0f9c1472fd2)

Save the template and exit. We're now ready to create a new document, save it and launch it - at this point, we should get our VBA code executed. Below GIF shows exactly that:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Li2x2sfJkV6f8_MjS1N%2F-Li30gD8VQws4gKB3sFR%2Fword-template.gif?alt=media\&token=90ab2e03-5207-4ce6-95c5-3a12c37bd223)

### References

{% embed url="https://www.mdsec.co.uk/2019/05/persistence-the-continued-or-prolonged-existence-of-something-part-1-microsoft-office/" %}
