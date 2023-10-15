# Systemctl-API

Set of endpoints for managing Linux services. REST API server **is based .NET HttpListener** with authorization and error handling.

The goal is to demonstrate the ability of PowerShell to be operating system independent (cross-platform) and used concurrently with the Bash language. It was also prompted by the impossibility to create a full-fledged server for REST API using standard means of Bash or netcat.

![Image alt](https://github.com/Lifailon/Systemctl-API/blob/rsa/Example.gif)

## 🚀 Launch

**Dependencies**:

Only **[PowerShell Core](https://github.com/PowerShell/PowerShell)**

Download latest version the distribution from the official repository on GitHub and install it on your system, here is an example for Ubuntu:

```Bash
deb_latest=$(curl https://api.github.com/repos/PowerShell/PowerShell/releases/latest | grep -Eom 1 "https://.+.deb")
curl -LO $deb_latest
deb_name=$(echo $deb_latest | awk -F / '{print $NF}')
dpkg -i $deb_name
```

Set variables:
```Bash
$ip   = "192.168.3.104"
$port = "8080"
$path = "/var/log/systemctl-api.log"
$cred = "cmVzdDphcGk="
```

Open the specified network port in your firewall.

Start:
`powershell systemctl-api.ps1`

Warning: **use root privileges** to start the server if you need to manage services via POST requests.

## 🔒 Authorization

Basic authentication scheme is used on the basis of **Base64**.

For authorization it is necessary to pass the login and password in Base64 format to the server in the **variable $cred**. To get the login and password in Base64 format, use the construct:

```PowerShell
$user = "rest"
$pass = "api"
[System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("${user}:${pass}"))
```

**For client connectivity, use one of the designs**:

```PowerShell
$user = "rest"
$pass = "api"
$SecureString = ConvertTo-SecureString $pass -AsPlainText -Force
$Credential = New-Object System.Management.Automation.PSCredential($user, $SecureString)
#$Credential = Get-Credential
Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/api/service/cron
```

2nd option, using header:

```PowerShell
$user = "rest"
$pass = "api"
$EncodingCred = [System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("${user}:${pass}"))
$Headers = @{"Authorization" = "Basic ${EncodingCred}"}
Invoke-RestMethod -Headers $Headers -Uri http://192.168.3.104:8080/api/service/cron
```

The server will match the credentials received in the header request from the client.

## 🎉 GET Request

To obtain specific service status:

`Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/api/service/cron`

To get a list of all registered unit services:

`Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/api/service`

To get the list of all registered unit services with their operation status (works slower):

`Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/api/service-status`

## 🎊 POST Request

**Stop** the service:

`Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/api/service/cron -Method Post -Headers @{"Status" = "Stop"}`

**Start** the service:

`Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/api/service/cron -Method Post -Headers @{"Status" = "Start"}`

**Restart** the service:

`Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/api/service/cron -Method Post -Headers @{"Status" = "Restart"}`

## 💎 Other Endpoints

Get a list of all processes with a detailed description of all characteristics in **html format**:

`Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/process`

Get a list of all processes with a detailed description of all characteristics in **json format**:

`Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/api/process`

Get current uptime and average load:

`Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/api/uptime`

Output the state of physical disks (lsblk is used):

`Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/api/disk`

## ⚠️ Error Handling

If the service name is incorrect or the service does not exist, you will receive a message.

**400. Bad Request**:

```PowerShell
PS C:\Users\Lifailon> Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/api/service/cronn
Invoke-RestMethod: Bad Request (service cronn could not be found)
```

In case your login or password is incorrect, you will receive the following message.

**401, Unauthorized**:

`Invoke-RestMethod: Unauthorized (login or password is invalid)`

**404. Not Found endpoint**:

```
Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/api
Invoke-RestMethod: Not Found endpoint
```

**405. Method Not Allowed**:

```
PS C:\Users\Lifailon> Invoke-RestMethod -Credential $Credential -AllowUnencryptedAuthentication -Uri http://192.168.3.104:8080/api/service/cron -Method Put -Headers @{"Status" = "Restart"} | fl   
Invoke-RestMethod: Method Not Allowed
```
