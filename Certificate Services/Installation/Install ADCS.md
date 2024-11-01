# Install Active Directory Certificate Services (ADCS) with Web Enrollment.

## Deploy the ADCS role binaries and related tools.

The feature list of things to be installed with the configuration files can be found [here](./Config/Config.md).

```powershell
Install-WindowsFeature -ConfigurationFilePath C:\TEMP\DeploymentConfigTemplate.xml
```

## Install a standalone Root CA.
```powershell
Install-AdcsCertificationAuthority -CACommonName $env:COMPUTERNAME -CAType StandaloneRootCA -CryptoProviderName "RSA#Microsoft Software Key Storage Provider" -KeyLength 4096 -HashAlgorithmName SHA256 -ValidityPeriod Years -ValidityPeriodUnits 10 -Confirm:$False
```

## Install Web Enrollment feature.
```powershell
Install-AdcsWebEnrollment -Confirm:$False
```
