Import a module:

```powershell
Set-ExecutionPolicy Bypass -Scope CurrentUser
. .\Module.ps1
```

Invoke shell as a domain user:

```powershell
runas /netonly /user:DOMAIN\USER powershell.exe
```