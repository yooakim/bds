# HOWTO: Setup Octopus Lab Environment

## Login

```bash
az login  --tenant f23467b9-39bc-4e1f-af1e-9f28ff9b325a 
az account set --subscription 20370D1F-E18A-47F1-9BCC-D3BB9369CB72
```

## Create the DevTest Lab

```bash
az group create --name OctoLab --location westeurope


```

## Setup SQL Server Express

In order to work in with Octopus Deploy we need SQL Exerver Express. Install SQL Server Express as follows:

```powershell
$ProgressPreference='Silent'
$uri = 'https://go.microsoft.com/fwlink/?linkid=853017'
$OutFile = 'sqlexpress.exe'
$ProgressPreference='Silent'
Invoke-WebRequest -UseBasicParsing -Uri $uri -OutFile ".\Downloads\$OutFile" 
Unblock-File ".\Downloads\$OutFile"
& ".\Downloads\$OutFile" /ENU /IACCEPTSQLSERVERLICENSETERMS /QUIET /VERBOSE
```
once the SQL Express instance have been installed you can access it using the following connection string `Server=(local)\SQLEXPRESS;Database=master;Trusted_Connection=True;`

## Install Azure Data Studio (optional)


```powershell
$uri = 'https://go.microsoft.com/fwlink/?linkid=2100711'
$OutFile = 'azuredatastudio.exe'
$ProgressPreference='Silent'
Invoke-WebRequest -UseBasicParsing -Uri $uri -OutFile ".\Downloads\$OutFile" 
Unblock-File ".\Downloads\$OutFile"
& ".\Downloads\$OutFile" /SILENT

```

## Install Octopus Server

```powershell
Set-Location -Path 
$uri = 'https://octopus.com/downloads/latest/WindowsX64/OctopusServer'
$OutFile = 'octopus-x64.msi'
$ProgressPreference='Silent'
Invoke-WebRequest -UseBasicParsing -Uri $uri -OutFile ".\Downloads\$OutFile" 
Unblock-File ".\Downloads\$OutFile"
# Install Octopus Server software but do not configure an instance
& msiexec /i "$ENV:USERPROFILE\Downloads\$OutFile" /quiet RUNMANAGERONEXIT=no
```

## Install Octopus Instance

```
Push-Location
Set-Location 'C:\Program Files\Octopus Deploy\Octopus'
.\Octopus.Server.exe create-instance --instance "OctopusServer" --config "C:\Octopus\OctopusServer.config" --serverNodeName "$env:COMPUTERNAME"
.\Octopus.Server.exe database --instance "OctopusServer" --connectionString "Data Source=(local)\SQLEXPRESS;Initial Catalog=Octopus;Integrated Security=True" --create --grant "NT AUTHORITY\SYSTEM"
.\Octopus.Server.exe configure --instance "OctopusServer" --webForceSSL "False" --webListenPrefixes "http://localhost:80/" --commsListenPort "10943" --usernamePasswordIsEnabled "True"
.\Octopus.Server.exe service --instance "OctopusServer" --stop
.\Octopus.Server.exe admin --instance "OctopusServer" --username "admin" --email "joakim@jwab.net" --password "Lik32Deploy!"
.\Octopus.Server.exe license --instance "OctopusServer" --licenseBase64 "PExpY2Vuc2UgU2lnbmF0dXJlPSJaYS9mV3ZLQTFQNlZZYThZN2xBTG5GNWE0d25lRG92TXFPV1JGa2xRSmhlcXkvdXBPZUIvNEJHVFFObmZPZTg2ZHEzOWFiekl6U05uZjB5aFdaQWdqdz09Ij4NCiAgPExpY2Vuc2VkVG8+Sm9ha2ltIFdlc3RpbiBBQjwvTGljZW5zZWRUbz4NCiAgPExpY2Vuc2VLZXk+NjQzMjUtMjAyMjEtNDE0MjItNTkyMzU8L0xpY2Vuc2VLZXk+DQogIDxWZXJzaW9uPjIuMDwvVmVyc2lvbj4NCiAgPFZhbGlkRnJvbT4yMDE5LTA5LTAxPC9WYWxpZEZyb20+DQogIDxQZXJtaXNzaW9ucz5SZXN0cmljdGVkPC9QZXJtaXNzaW9ucz4NCiAgPFRhc2tDYXA+MjwvVGFza0NhcD4NCiAgPEtpbmQ+U3RhcnRlcjwvS2luZD4NCiAgPFZhbGlkVG8+MjAyMC0wOS0wMTwvVmFsaWRUbz4NCiAgPFByb2plY3RMaW1pdD5VbmxpbWl0ZWQ8L1Byb2plY3RMaW1pdD4NCiAgPE1hY2hpbmVMaW1pdD4xMDwvTWFjaGluZUxpbWl0Pg0KICA8VXNlckxpbWl0PlVubGltaXRlZDwvVXNlckxpbWl0Pg0KICA8V29ya2VyTGltaXQ+MTwvV29ya2VyTGltaXQ+DQogIDxTcGFjZUxpbWl0PjE8L1NwYWNlTGltaXQ+DQo8L0xpY2Vuc2U+"
.\Octopus.Server.exe service --instance "OctopusServer" --install --reconfigure --start --dependOn 'MSSQL$SQLEXPRESS'

netsh advfirewall firewall add rule name="Octopus Deploy Server - HTTP"  dir=in action=allow protocol=TCP localport=80
netsh advfirewall firewall add rule name="Octopus Deploy Server - HTTPS" dir=in action=allow protocol=TCP localport=443
netsh advfirewall firewall add rule name="Octopus Deploy Server - Tentacle Polling" dir=in action=allow protocol=TCP localport=10943

#netsh advfirewall firewall add rule "name=Octopus Deploy Tentacle" dir=in action=allow protocol=TCP localport=10933

netsh advfirewall firewall add rule name='Octopus Deploy Server' dir=in program='C:\Program Files\Octopus Deploy\Octopus\Octopus.Server.exe' action=allow
Pop-Location

# Access the server
Start-Process http://localhost:80/

```

## Add Let's Encrypt Certificate


```
http://localhost/app#/Spaces-1/configuration/letsencrypt
```
