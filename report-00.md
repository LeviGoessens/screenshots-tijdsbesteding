# Enterprise Linux Lab Report Assignment 00

- Student name: Jeroen Poelaert
- Github repo: <https://github.com/HoGentTIN/elnx-1819-sme-jeroenpoelaert>

Describe the goals of the current iteration/assignment in a short sentence:
Het automatiseren van de installatie en configuratie van een LAMP-stack via Ansible

## Test plan

1. On the host system, go to the local working directory of the project repository
2. Execute `vagrant status`
    - There should be one VM, `pu004` with status `not created`. If the VM does exist, destroy it first with `vagrant destroy -f pu004`
3. Execute `vagrant up pu004`
    - The command should run without errors (exit status 0)
4. Log in on the server with `vagrant ssh pu004` and run the acceptance tests. They should succeed.
    ```
    sudo /vagrant/test.runbats.sh
    ```
	Any tests for the LAMP stack may fail, but this is not part of the current assignment.
5. Log off from the server and ssh to the VM as described. You should **not** get a password prompt.
	```
	$ logout
	$ ssh jeroen@192.0.2.50
    ```
## Procedure/Documentation

1. We werken met GitBash in elnx-1819-sme-jeroenpoelaert

2. Aanmaken van de hosts/servers: aanpassen van het bestand vagrant-hosts.yml in de root directory van het project. De format de je gebruikt is als volgt:
    ```
	- name : pu004
	  ip : 192.0.2.50
3. Aanpassen `ansible\site.yml`:

    ```
	- hosts: pu004
      become: true
      roles: 
        - bertvv.rh-base
4. Rol voorzien in de map `ansible\roles`: Alle rollen kunnen via bertvv opgezocht worden. In dit geval: `bertvv.rh-base`. Commando `/scripts/role-deps.sh` werkt eveneens. We gebruiken een aantal functionaliteiten hiervan in `group_vars\all´

5. EPEL repository: toevoegen van volgende lijnen in `ansible\group_vars\all.yml`:

	```
	rhbase_repositories:
      - epel-release	  
6. Required packages voor iedere server: toevoegen van rhbase_install_packages in `ansible\group_vars\all`:

    ```
    rhbase_install_packages:
      - bash-completion
      - vim-enhanced
      - bind-utils
      - git
      - nano
      - tree
      - wget  
7. MOTD: toevoegen van rhbase_motd:

   ```
   rhbase_motd: true
8. Aanpassen `\ansible\group_vars\all.yml` voor de je eigen user te creëren. 

    ```
    rhbase_users:
      - name: jeroen
        comment: 'administrator'
        groups:
          - wheel
    ```
    En ssh key voor de user.
    ```
    rhbase_ssh_user: jeroen
    rhbase_ssh_key: "ssh-rsa **generated key**"
## Test report

- Door het commando `sudo /vagrant/test/runbats.sh` uit te voeren kunnen we voor deze assignment testen of alles correct is. Alle tests moeten slagen. 


![pu004](../test/runbatsreports/pu004testrapport.PNG)


## Resources

    - https://galaxy.ansible.com/bertvv/rh-base
