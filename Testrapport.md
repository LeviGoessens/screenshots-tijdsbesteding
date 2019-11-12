### Testplan Lima2
    ```
    Test uitgevoerd door: Levi Goessens
    op: 11/11/2019 - 12/11/2019

## De server naam en domein

1. De server naam is Lima2. **OK**
2. Lima2 is het domein auotmatisch gejoined. **OK**

## Controleren dat alle volumes aangemaakt zijn

Open Server Manager -> File and Storage Services -> Volumes

Disk1:
1. 'C:' met label 'System' **OK**
2. 'D:' met label 'VerkoopData' **OK**
3. 'E:' met label 'OntwikkelingData' **OK**

Disk2:
1. 'F:' met label 'ITData' **OK**
2. 'G:' met label 'DirData' **OK**
3. 'H:' met label 'AdminData' **OK**
4. 'Y:' met label 'HomeDirs' **OK**
5. 'Z:' met label 'ProfileDirs' **OK**

* Alle volumes zijn 

## Controleren dat alle shares aangemaakt zijn

Open Server Manager -> File and Storage Services -> Shares

* Er zijn 6 shares **Er zijn 8 shares**   
De shares:
1. AdminData **OK**
2. DirData **OK**
3. HomeDirs **OK**
4. ITData **OK**
5. ProfileDirs **OK**
6. ShareVerkoop **OK**   
**7. OntwikkelingData**   
**8. VerkoopData**

## De schaduw kopie

Open de Task Scheduler -> Task Scheduler (local) -> Task Scheduler Library

* Controleren dat een nieuwe task is toegevoegd met de huidige naam 'TEST10'. 
* Deze taak heeft een trigger die om 5pm activeert en elke dag.
* Om een snellte test uit te voeren kan je op de taak klikken en deze manueel uitvoeren. **OK**
* Controleer volume AdminData. Dit volume bevat een nieuwe schaduw kopie. Dit kan je nazien door naar AdminData te gaan -> Properties -> Shadow Copies.
* Het veld "Shadow copies of selected volume bevat een nieuw veld met de huidige tijd en datum. **OK**

## Share Permissies

1. Log in als gebruikder CedricD met wachtwoord Administrator2019

2. Ga naar bestanden -> Computer en ga naar de verschillende fileshares

3. Ga in de fileshare van de afdeling Ontwikkeling en probeer een nieuwe file aan te maken -> dit zou moeten lukken **OK**

4. Ga in de fileshare van homedir en probeer een nieuwe file aan te maken -> dit zou moeten lukken **OK**

5. ga in de fileshare van Directie en probeer een nieuwe file aan te maken -> dit zou NIET mogen lukken. **OK**


## maximumcapaciteit share per user

1. neem een folder of bestand van 150MB en probeer het in de fileshare Ontwikkeling te zetten -> dit moet lukken

2. verwijder het 150MB bestand of folder

3. probeer nu een folder/bestand van 250MB in de fileshare Ontwikkeling te zetten -> dit mag NIET lukken.




