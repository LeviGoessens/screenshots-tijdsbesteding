Making Ansible a bit faster

1. Werkwijze: We zetten steeds de verschillende roles in commentaar en zetten zo een base box op. Daarna worden de nodig lijnen toegevoegd in de Ansible config file (aangezien deze bij een destroy verloren gaan) en vervolgens zetten we de roles uit commentaar. We kunnen nu de tijd meten van het uitvoeren van het playbook.
2. Om de doorlooptijd van het uitvoeren van het playbook weer te kunnen geven voegen we het volgende toe in ansible.cfg:
   - callback_whitelist = profile_tasks
3. Wanneer we nu de webserver (pu001) provisionen, krijgen we de totale doorlooptijd en de tijd van de 20 langstdurende processen
4. Zonder enige versnelling:

```
PLAY RECAP *********************************************************************
pu001                      : ok=84   changed=46   unreachable=0    failed=0    skipped=32   rescued=0    ignored=0

Friday 20 December 2019  14:13:27 +0000 (0:00:02.576)       0:04:56.965 *******
===============================================================================
bertvv.mariadb : Install packages -------------------------------------- 80.02s
bertvv.wordpress : Install Wordpress ----------------------------------- 51.02s
bertvv.rh-base : Install | Ensure specified packages are installed ----- 50.80s
bertvv.httpd : Ensure Apache is installed ------------------------------ 15.76s
bertvv.rh-base : Security | Make sure basic services can pass through firewall --- 7.08s
bertvv.rh-base : Security | Make sure user specified services can pass through firewall --- 5.04s
bertvv.rh-base : Install | Ensure basic systemd services are running ---- 4.60s
bertvv.rh-base : Security | Make sure the firewall is running ----------- 3.81s
Gathering Facts --------------------------------------------------------- 3.55s
bertvv.rh-base : Install | Role/Ansible dependencies -------------------- 2.67s
bertvv.rh-base : Security | Make sure SELinux has the desired state (enforcing) --- 2.58s
bertvv.wordpress : restart firewalld ------------------------------------ 2.58s
bertvv.mariadb : restart mariadb ---------------------------------------- 2.49s
bertvv.wordpress : Download Salts --------------------------------------- 2.48s
bertvv.rh-base : Install | Package management configuration (yum) ------- 2.45s
bertvv.httpd : restart httpd -------------------------------------------- 2.44s
bertvv.rh-base : Users | Add users -------------------------------------- 2.40s
bertvv.rh-base : Install | Ensure specified packages are NOT installed --- 2.27s
bertvv.mariadb : Ensure service is started ------------------------------ 2.01s
bertvv.rh-base : Config | Set protocol version for SSH ------------------ 1.77s

```

5. De totale doorlooptijd is op dit moment: 04:56:965
6. We gaan nu verschillende methodes onderzoeken om deze tijd naar beneden te krijgen.

### SSH multiplexing

7. SSH multiplexing zorgt ervoor dat ansible reeds geopende ssh sessies kan hergebruiken. Dit kan enabled worden door **ssh_args = -o ControlMaster=auto -o ControlPersist=60s** toe te voegen. Achteraf bleek dat dit automatisch enabled is in ansible, en dat de tijd dus amper zou moeten veranderen.

```
PLAY RECAP *********************************************************************
pu001                      : ok=84   changed=46   unreachable=0    failed=0    skipped=32   rescued=0    ignored=0

Saturday 21 December 2019  09:49:49 +0000 (0:00:03.067)       0:04:56.167 *****
===============================================================================
bertvv.mariadb : Install packages -------------------------------------- 85.55s
bertvv.wordpress : Install Wordpress ----------------------------------- 54.31s
bertvv.rh-base : Install | Ensure specified packages are installed ----- 43.41s
bertvv.httpd : Ensure Apache is installed ------------------------------ 16.97s
bertvv.rh-base : Security | Make sure basic services can pass through firewall --- 6.09s
bertvv.rh-base : Security | Make sure user specified services can pass through firewall --- 4.92s
bertvv.rh-base : Install | Ensure basic systemd services are running ---- 3.94s
bertvv.rh-base : Security | Make sure the firewall is running ----------- 3.73s
Gathering Facts --------------------------------------------------------- 3.22s
bertvv.mariadb : restart mariadb ---------------------------------------- 3.13s
bertvv.wordpress : restart firewalld ------------------------------------ 3.07s
bertvv.rh-base : Install | Role/Ansible dependencies -------------------- 2.69s
bertvv.httpd : restart httpd -------------------------------------------- 2.61s
bertvv.wordpress : Download Salts --------------------------------------- 2.49s
bertvv.rh-base : Install | Package management configuration (yum) ------- 2.49s
bertvv.rh-base : Security | Make sure SELinux has the desired state (enforcing) --- 2.22s
bertvv.rh-base : Users | Add users -------------------------------------- 2.00s
bertvv.rh-base : Install | Ensure specified packages are NOT installed --- 1.98s
bertvv.rh-base : Users | Add groups ------------------------------------- 1.80s
bertvv.mariadb : Ensure service is started ------------------------------ 1.71s

```

8. Zoals verwacht is de tijd bijna hetzelfde gebleven.

### SSH pipelining

9. Voeg **pipelining = True** toe aan de config file.
10. SSH pipelining zorgt ervoor dat ansible de commando's meteen door de STDIN kan sturen met behulp van een continue connectie, wat veel sneller is dan de default instellingen.

```
PLAY RECAP *********************************************************************
pu001                      : ok=84   changed=46   unreachable=0    failed=0    skipped=32   rescued=0    ignored=0

Friday 20 December 2019  14:35:17 +0000 (0:00:02.433)       0:04:00.858 *******
===============================================================================
bertvv.mariadb : Install packages -------------------------------------- 70.13s
bertvv.wordpress : Install Wordpress ----------------------------------- 41.46s
bertvv.rh-base : Install | Ensure specified packages are installed ----- 39.78s
bertvv.httpd : Ensure Apache is installed ------------------------------ 12.85s
bertvv.rh-base : Security | Make sure basic services can pass through firewall --- 4.68s
bertvv.rh-base : Install | Role/Ansible dependencies -------------------- 3.80s
bertvv.rh-base : Security | Make sure user specified services can pass through firewall --- 3.60s
bertvv.rh-base : Install | Ensure basic systemd services are running ---- 3.11s
bertvv.mariadb : restart mariadb ---------------------------------------- 3.04s
bertvv.wordpress : restart firewalld ------------------------------------ 2.43s
Gathering Facts --------------------------------------------------------- 2.41s
bertvv.httpd : restart httpd -------------------------------------------- 2.02s
bertvv.rh-base : Install | Package management configuration (yum) ------- 2.00s
bertvv.wordpress : Download Salts --------------------------------------- 1.91s
bertvv.rh-base : Security | Make sure the firewall is running ----------- 1.85s
bertvv.rh-base : Security | Make sure SELinux has the desired state (enforcing) --- 1.71s
bertvv.mariadb : Ensure service is started ------------------------------ 1.56s
bertvv.rh-base : Users | Add users -------------------------------------- 1.45s
bertvv.mariadb : Install server config file ----------------------------- 1.40s
bertvv.mariadb : Set MariaDB root password for the first time (root@localhost) --- 1.32s


```

11. Na het toevoegen van SSH pipelining is de totale doorlooptijd nog slechts 04:00:858

## Mitogen

12. Mitogen zal de trage implementaties waar shell centraal staat vervangen door python equivalenten, waardoor de doorloopsnelheid verlaagd.
13. We downloaden de Mitogen 0.2.9 plugin, en geven het path mee in de configfile: C:/Users/levig/Desktop/school/Linux/github/elnx-1920-sme-LeviGoessens/ansible/Mitogen/dist/mitogen-0.2.9/ansible_mitogen/plugins/strategy

```
PLAY RECAP *********************************************************************
pu001                      : ok=84   changed=46   unreachable=0    failed=0    skipped=32   rescued=0    ignored=0

Friday 20 December 2019  14:57:53 +0000 (0:00:02.225)       0:03:49.564 *******
===============================================================================
bertvv.mariadb : Install packages -------------------------------------- 65.07s
bertvv.wordpress : Install Wordpress ----------------------------------- 42.57s
bertvv.rh-base : Install | Ensure specified packages are installed ----- 33.36s
bertvv.httpd : Ensure Apache is installed ------------------------------ 12.42s
bertvv.rh-base : Security | Make sure basic services can pass through firewall --- 4.52s
bertvv.rh-base : Security | Make sure user specified services can pass through firewall --- 3.46s
bertvv.rh-base : Security | Make sure the firewall is running ----------- 2.95s
bertvv.rh-base : Install | Ensure basic systemd services are running ---- 2.94s
bertvv.mariadb : restart mariadb ---------------------------------------- 2.36s
Gathering Facts --------------------------------------------------------- 2.27s
bertvv.wordpress : restart firewalld ------------------------------------ 2.23s
bertvv.httpd : restart httpd -------------------------------------------- 2.20s
bertvv.rh-base : Install | Role/Ansible dependencies -------------------- 2.04s
bertvv.wordpress : Download Salts --------------------------------------- 1.91s
bertvv.rh-base : Services | Ensure SSH daemon is running ---------------- 1.85s
bertvv.rh-base : Install | Package management configuration (yum) ------- 1.76s
bertvv.rh-base : Security | Make sure SELinux has the desired state (enforcing) --- 1.65s
bertvv.rh-base : Users | Add groups ------------------------------------- 1.62s
bertvv.rh-base : Users | Add users -------------------------------------- 1.51s
bertvv.mariadb : Ensure service is started ------------------------------ 1.48s

```

14. Mitogen slaagt erin om de totale doorlooptijd te reduceren tot 03:49:564, dat is nog net iets beter dan SSH pipelining
15. We kunnen dus stellen dat met het toevoegen van een enkel lijntje code (SSH pipelining) of het downloaden van een plugin + 1 lijntje code (Mitogen) de doorlooptijd toch een stuk korter wordt. Vooral wanneer veel roles moeten geïnstalleerd worden kan dit je heel wat tijd besparen!

16. Gebruikte sites:

- https://adamj.eu/tech/2015/05/18/making-ansible-a-bit-faster/
- https://github.com/jlafon/ansible-profile
- https://dzone.com/articles/speed-up-ansible
- https://www.toptechskills.com/ansible-tutorials-courses/speed-up-ansible-playbooks-pipelining-mitogen/
