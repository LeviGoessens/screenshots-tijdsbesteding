Making Ansible a bit faster

1. Om de doorlooptijd van het uitvoeren van het playbook weer te kunnen geven voegen we het volgende toe in ansible.cfg:
   - callback_whitelist = profile_tasks
2. Wanneer we nu de webserver (pu001) provisionen, krijgen we de totale doorlooptijd en de tijd per onderdeel
3. Zonder enige versnelling:
   ```
   PLAY RECAP **********************************\***********************************
   pu001 : ok=79 changed=0 unreachable=0 failed=0 skipped=35 rescued=0 ignored=0
   ```

# Thursday 12 December 2019 11:01:00 +0000 (0:00:00.027) 0:00:53.769 **\***

bertvv.wordpress : Install Wordpress ------------------------------------ 3.15s
bertvv.rh-base : Security | Make sure user specified services can pass through firewall --- 2.75s
bertvv.rh-base : Install | Ensure basic systemd services are running ---- 2.36s
bertvv.rh-base : Security | Make sure basic services can pass through firewall --- 2.32s
bertvv.httpd : Ensure Apache is installed ------------------------------- 2.02s
Gathering Facts --------------------------------------------------------- 1.89s
bertvv.mariadb : Install packages --------------------------------------- 1.83s
bertvv.rh-base : Install | Role/Ansible dependencies -------------------- 1.59s
bertvv.rh-base : Install | Package management configuration (yum) ------- 1.41s
bertvv.rh-base : Security | Make sure SELinux is Enforcing -------------- 1.31s
bertvv.rh-base : Users | Add users -------------------------------------- 1.09s
bertvv.rh-base : Users | Set up SSH key for user levi ------------------- 0.93s
bertvv.rh-base : Install | Ensure specified packages are installed ------ 0.90s
bertvv.rh-base : Install | Automatic updates (yum-cron) ----------------- 0.87s
bertvv.rh-base : Install | Ensure specified external repositories are installed --- 0.86s
bertvv.mariadb : Set MariaDB root password for the first time (root@localhost) --- 0.85s
bertvv.rh-base : Users | Add groups ------------------------------------- 0.83s
bertvv.rh-base : Install | Ensure the machine-ID is available ----------- 0.79s
bertvv.mariadb : Configure swappiness ----------------------------------- 0.78s
bertvv.rh-base : Install | Ensure specified packages are NOT installed --- 0.77s

    ```

4. De totale doorlooptijd is op dit moment 53.769 seconden
