# Enterprise Linux Lab Report - Troubleshooting

- Student name: Levi Goessens
- Class/group: TIN2-TI-3B (Aalst)

## Instructions

- Write a detailed report in the "Report" section below (in Dutch or English)
- Use correct Markdown! Use fenced code blocks for commands and their output, terminal transcripts, ...
- The different phases in the bottom-up troubleshooting process are described in their own subsections (heading of level 3, i.e. starting with `###`) with the name of the phase as title.
- Every step is described in detail:
  - describe what is being tested
  - give the command, including options and arguments, needed to execute the test, or the absolute path to the configuration file to be verified
  - give the expected output of the command or content of the configuration file (only the relevant parts are sufficient!)
  - if the actual output is different from the one expected, explain the cause and describe how you fixed this by giving the exact commands or necessary changes to configuration files
- In the section "End result", describe the final state of the service:
  - copy/paste a transcript of running the acceptance tests
  - describe the result of accessing the service from the host system
  - describe any error messages that still remain

## Report

### Phase 1: Network access laag

    ```
    Nat kabel: juist
    Host only kabel: verkeerde adapter
    => adapter met 192.168.56.1/24 gebruiken
    ip link: beide kabels zitten in
    ```

### Phase 2: Internet laag

    ```
    ip --color a:
    eth0: 10.0.2.15/24 ==> correct
    eth1: 192.168.56.42/24 ==> correct
    ook in ifcfg-eth1 zijn de instellingen correct
    ip r:
    route is 10.2.2 ==> correct
    cat /etc/resolv.conf
    10.0.2.3 ==> correct
    We moeten nu kunnen pingen:
    - VM -> DG: ping 10.0.2.2 => OK
    - Host -> VM: ping 192.168.56.42 => OKv
    - VM -> DNS: ping 10.0.2.3 => OK
    - VM -> Host => OK


    We kunnen dus nu met ssh vagrant@192.168.56.42 via bash werken ==> dit werkt
    ```

### Phase 3: Transportlaag

    ```
    sudo su -
    systemctl status named.service ==> returned failed
    systemctl start named.service:
    sudo firewall-cmd --list-all
    port 53 wordt wel doorgelaten, maar dat is niet voldoende
    firewall-cmd --add-service dns --permanent
    systemctl restart firewalld.service
    ss -tldn: na de aanpassingen in /etc/named.conf:
    - 192.168.56.42::53 ==> correct
    - 10.0.2.15::53 ==> correct
    - 127.0.0.1::53 ==> correct

    ```

### Phase 4: Applicatielaag

    ```
    vim /etc/named.conf:
    listen-on port 53 { any; }; aanpassen
    allow-transfer {192.168.56.0/24; }; ontbreekt

    We zien hier dat de filename van de forward lookupzone /var/named/cynalco.com is
    (zet voorlopig de reverse lookupzones in commentaar)
    vim /var/named/cynalco.com ==> correct voor dit onderdeel
    systemctl start named.service ==> start nu wel op
    forward zone checken met het volgende commando:
    named-checkzone cynalco.com /var/named/cynalco.com:
    zone cynalco.com/IN: loaded serial 1574933535
    OK

    we kunnen nu de forward lookupzone testen:
    dig @192.168.56.42 golbat.cynalco.com +short
    => we krijgen 192.168.56.42 terug
    vanop onze host:
    nslookup www.cynalco.com 192.168.56.42:
    Server:  UnKnown
    Address:  192.168.56.42

    Name:    karakara.cynalco.com
    Address:  192.0.2.4
    Aliases:  www.cynalco.com

    (we zetten nu de eerste reverse lookupzone uit commentaar)

    vim /var/named/192.168.56.in-addr-arpa:
    dit moet een reverse lookup zone zijn, dus 192.168.56 moet omgekeerd staan:
    $ORIGIN 56.168.192.in-adrr.arpa.
    aangezien er een wijziging is gebeurd verhogen we de serial met 1
    we passen dit ook aan in /etc/named.conf:
    zone "56.168.192.in-addr.arpa" IN

    we kunnen nu een reverse lookup uitvoeren:
    dig @192.168.56.42 -x 192.168.56.42 +short
    => returnt golbat.cynalco.com.
    vanop onze host:
    nslookup 192.168.56.42 192.168.56.42
    Server:  golbat.cynalco.com
    Address:  192.168.56.42

    Name:    golbat.cynalco.com
    Address:  192.168.56.42

    (we zetten de 2de reverse zone uit commentaar)
    in /var/named/2.0.192.in-addr.arpa
    => de . achter cynalco.com ontbreekt telkens => aanpassen en serial verhogen

    acceptatietesten:
    forward lookups voor butterfree failed nog
    in /var/named/cynalco.com zien we inderdaad dat butterfree en beedle omgedraaid zijn
    butterfree 192.168.56.12
    beedle 192.168.56.13
    serial verhogen
    systemctl restart named.service
    => deze test slaagt nu
    reverse lookup voor butterfree faalt ook:
    in /var/named/192.168.56.in-addr.arpa:
    12 en 13 omwisselen, serial verhogen
    systemctl restart named.service
    NS record lookup faalt nog:
    tamatama is de secondary DNS server:
    in /var/named/cynalco.com moeten we het volgende toevoegen:
    IN NS tamatama.cynalco.com

    ```

## End result

    ```
    - We kunnen een forward en reverse lookup uitoveren vanop zowel de vm als vanop onze host
    - Alle acceptatietesten slagen
    ```

## Resources

List all sources of useful information that you encountered while completing this assignment: books, manuals, HOWTO's, blog posts, etc.
