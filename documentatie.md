# **Testplan Bravo 2**

Bravo 2 is een backup domein controller, daarom moet eerst Alfa 2 draaien, zodat Bravo 2 het domein kan joinen en kan repliceren van Alfa2.

## Voer het script 1_RUNFIRST.ps1 uit

1. Het script zorgt ervoor dat de aanmelding automatisch gebeurd met volgende waarden:
    - **Username:** Administrator
    - **Password:** Admin2019

2. Het script verandert de computernaam
3. Kan gechecked worden met het volgende commando in powershell: $env:computername | Select-Object
4. Als bravo2 teruggegeven wordt is de naam goed geconfigureerd.

## Voer het script 2_InstallDCDNS.ps1 uit

Het script gaat het volgende doen:

1. Het script past de tijdzone aan. 
    - **Get-TimeZone:**
        - Id: Romance Standard Time
    - **Get-Culture**
        - Name: eng-BE       
        
Bovenstaande kan getest worden via de gui of "ipconfig /all".

2. De adapternaam zou "LAN" moeten zijn. Dit kan getest worden door in Powershell het volgende in te geven:   
  C:\> Get-NetAdapter -Name "*"

3. Netwerkinstellingen. 
    - **ipconfig:**
        - IP-address: 172.18.1.67
        - Subnetmask: 255.255.255.224
        - Default Gateway: 172.18.1.98
        - DNS: - 172.18.1.66
               - 172.18.1.67
 
 4. De DNS controleren.   
      Via volgend commando: **Get-DnsClientServerAddress**   
      - De IPv4 adressen van DNS dat je zou moeten uitkomen zijn:
         - 172.18.1.66, 172.18.1.67 en 127.0.0.1
 
 5. ADDS controleren gaat als volgt. Onderstaande commando gaat testen of er wordt aangemeld met administrator, DNS geïnstalleerd is en of DSRM goed is ingesteld.   
      Commando: **Test-ADDSDomainControllerInstallation -InstallDns -Credential (Get-Credential) -DomainName (Read-Host "Domain to promote into")**
 
 6. Controle of firewall uitstaat. Deze zou op "off" moeten staan.   
      Commando: **netsh advfirewall show private|public|domain**
 
 7. Controle of domein "red.local" gejoind is.   
      Commando: **Get-WmiObject -Class Win32_ComputerSystem**

##  Voer het script 3_ConfigDCDNS.ps1 uit
1. Kijken of DNS repliceert:   
     Commando: **Get-WMIObject –namespace “Root\MicrosoftDNS” –class MicrosoftDNS_Zone | Format-List Name**
     Nadien commando: **Get-WMIObject –namespace “Root\MicrosoftDNS” –class MicrosoftDNS_Zone | Where-Object {$_.Name –eq “red.local”}**
  
