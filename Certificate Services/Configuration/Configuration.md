# Configuration of ADCS.

This configuration...  

    removes:
        Default CDP and AIA URIs
        Default website files
        Default Windows Authentication Providers for CertSrv site

    creates/adds:
        New web virtual folder for CRL
        Negotiate Authentication Provider for CertSrv site
        New CDP and AIA URIs

    configures:
        Directory browsing for CRL folder
        Double escaping in the CRL virtual folder
        SSL requirement on CertSrv site
        SSL certificate for the CA server
        Auditing

## Remove default CDP and AIA URIs.
```powershell
Get-CACrlDistributionPoint | ForEach-Object { Remove-CACrlDistributionPoint -Uri $PSItem.Uri -Confirm:$False }
Get-CAAuthorityInformationAccess | ForEach-Object { Remove-CAAuthorityInformationAccess -Uri $PSItem.Uri -Confirm:$False }
```

## Restart CA service.
```powershell
Restart-Service -Name CertSvc
```

## Delete default web site files.
```powershell
Remove-Item -Path C:\inetpub\wwwroot\* -Recurse -Confirm:$False
```

## Create new CRL folder.
```powershell
New-Item -Path C:\inetpub\wwwroot\ -Name CRL -ItemType Directory
```

## Create a new virtual directory for CRL.
```powershell
New-WebVirtualDirectory -Site 'Default Web Site' -Application 'DefaultAppPool' -Name 'CRL' -PhysicalPath C:\inetpub\wwwroot\CRL
```

## Enable directory browsing on the CRL virtual site.
```powershell
Set-WebConfigurationProperty -Location 'Default Web Site/CRL' -Filter 'system.webServer/directoryBrowse' -Name enabled -Value True
```

## Enable double escaping on the CRL virtual site.
```powershell
Set-WebConfigurationProperty -Location 'Default Web Site/CRL' -Filter 'System.WebServer/security/requestFiltering' -Name allowDoubleEscaping -Value True
```

## Require Extended Protection on CertSrv site.
```powershell
Set-WebConfigurationProperty -Location 'Default Web Site/CertSrv' -Filter 'system.webServer/security/authentication/windowsAuthentication/extendedProtection' -Name 'tokenChecking' -Value 'Require'
```

## Require SSL on CertSrv site.
```powershell
Set-WebConfigurationProperty -Location 'Default Web Site/CertSrv' -Filter 'system.webServer/security/access' -Name 'sslFlags' -Value 'Ssl'
```

## Remove default windows authentication providers.
```powershell
Remove-WebConfigurationProperty -Location 'Default Web Site/CertSrv' -Filter 'system.webServer/security/authentication/windowsAuthentication/providers' -Name '.'
```

## Add Negotiate provider.
```powershell
Add-WebConfigurationProperty -PSPath 'MACHINE/WEBROOT/APPHOST' -Location 'Default Web Site/CertSrv' -Filter 'system.webServer/security/authentication/windowsAuthentication' -Name providers -Value Negotiate
```

## Optional: Download rebranding files from your repo to substitute the default IIS files that were deleted previously.
```powershell
@( 'index.html', 'img.png' ) | ForEach-Object { Start-BitsTransfer -Source https://repo.com/CDN/branding/$PSItem -Destination C:\inetpub\wwwroot\ -Dynamic }
```

## Create new CDP and AIA URIs.
```powershell
Add-CACrlDistributionPoint -Uri 'C:\inetpub\wwwroot\CRL\<CAName><CRLNameSuffix><DeltaCRLAllowed>.CRL' -PublishToServer -PublishDeltaToServer -Confirm:$False
Add-CACrlDistributionPoint -Uri 'http://<ServerDNSName>/CRL/<CAName><CRLNameSuffix><DeltaCRLAllowed>.CRL' -AddToCertificateCdp -AddToFreshestCrl -Confirm:$False
Add-CAAuthorityInformationAccess -Uri 'http://<ServerDNSName>/CertEnroll/<CertificateName>.crt' -AddToCertificateAia -Confirm:$False
```

## Restart CA service.
```powershell
Restart-Service -Name CertSvc
```

## Set location to local machine cert store and get web server certificate, stored in a variable.
```powershell
Set-Location -Path Cert:\LocalMachine\My
$webcert = Get-Certificate -Template MyCAWebServerTemplate
```

## Set new cert binding to the Default IIS site.
```powershell
Get-ChildItem -Path IIS:\SslBindings\ | Where-Object Port -eq 443 | Set-ItemProperty -Name Thumbprint -Value $webcert.Certificate.Thumbprint
```

## Enable all audit types in CA, where MYCANAME is the server name
```powershell
New-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\MYCANAME\ -Name AuditFilter -Value 127 -PropertyType DWORD
```

## Cleanup. Remove the default CertEnroll web virtual directory and file system directory.
```powershell
Remove-Item -Path 'IIS:\Sites\Default Web Site\CertEnroll\' -Recurse -Confirm:$False
Remove-Item -Path 'C:\Windows\System32\CertSrv\CertEnroll\' -Recurse -Confirm:$False
```
