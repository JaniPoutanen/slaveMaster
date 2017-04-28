# slaveMaster
H4 palvelinten hallinta

Tehtävänanto sivulla http://terokarvinen.com/2017/aikataulu-%e2%80%93-palvelinten-hallinta-ict4tn022-2-%e2%80%93-5-op-uusi-ops-loppukevat-2017-p2
h4. a) Master-slave. Aloita tyhjästä koneesta. Tee yhdestä koneesta orja ja toisesta herra. Kokeile, että orja saa herralta modulin. (Voit käyttää labraa, kun se on tyhjä. Laita mukaan ‘puppet cert –list –all’, ‘tail /var/log/auth.log’, ‘tail /var/log/syslog’).

b) Varaa omalle modulillesi aihe kommenttina tälle sivulle. Yksi työ yhdestä aiheesta. 

Ohjeena käytin sivua: http://terokarvinen.com/2012/puppetmaster-on-ubuntu-12-04

Master koneella:
    sudo hostnamectl set-hostname herra
    sudo apt-get -y install puppetmaster    
    sudoedit /etc/puppet/puppet.conf
    
[main]
logdir=/var/log/puppet
vardir=/var/lib/puppet
ssldir=/var/lib/puppet/ssl
rundir=/run/puppet
factpath=$vardir/lib/facter
prerun_command=/etc/puppet/etckeeper-commit-pre
postrun_command=/etc/puppet/etckeeper-commit-post

[master]
# These are needed when the puppetmaster is run by passenger
# and can safely be removed if webrick is used.
ssl_client_header = SSL_CLIENT_S_DN
ssl_client_verify_header = SSL_CLIENT_VERIFY
dns_alt_names = puppet, herra.local

ja /etc/hosts tiedostoon muutetaan sivi:

    127.0.1.1       jani herra.local

    sudo service avahi-daemon restart
    
Seuraavaksi tein Puppet modulin
    sudo apt-get install puppet
    siirryin kansioon /etc/puppet/manifests, johon lisäsin tiedoston site.pp, joka sisällöksi rivi
    class {"orja":}
    
    itse modulin kopioin edellisestä tehtävästä, jossa vaihdetaan virtuaalisivu
   
   LINKKI
   Seuraavaksi tein orja nimisen modulin tiedostoon /etc/puppet/manifests/orja/manifests/init.pp:

    class orja {       
        file { '/home/xubuntu/H4':
                ensure => 'directory',
        }
        file { '/home/xubuntu/H4/heippa.txt':
                content => 'Heippa!,
        }
    }

Tyhjensin certifikaattiavaimet:

    sudo service puppetmaster stop
    sudo rm -r /var/lib/puppet/ssl
    sudo service puppetmaster start

Siirryin orjakoneelle, jossa:
    slave$ sudo apt-get -y install puppet

    sudoedit /etc/puppet/puppet.conf

Add master DNS name under [agent] heading. Puppet will connect to server.

    [agent]
    server = master.local

    sudo puppet agent --enable

    sudo puppet agent –tvd
    
 Muutaman kirjoitusvirheen korjauksen jälkeen tämä toimi ja sain orjakoneen kotihakemistoon kansion H4, jossa oli tiedosto heippa.txt.
 
    sudo puppet cert list -all
    
+ "herra.tielab.haaga-helia.fi" (SHA256) EC:8F:12:DC:65:4F:8F:E4:EA:9E:2E:BF:B5:3F:9F:4B:85:6B:0A:D8:AF:41:6B:DD:44:B0:FF:23:70:8C:D6:48 (alt names: "DNS:herra.local", "DNS:herra.tielab.haaga-helia.fi", "DNS:puppet")
+ "orja.tielab.haaga-helia.fi"  (SHA256) 77:E2:4F:EA:16:0F:93:B9:90:6D:A9:32:DD:18:92:14:8A:47:60:FD:39:97:5A:D0:5A:E9:B2:65:87:C5:AF:FB

    tail /var/log/auth.log
    
Apr 28 14:24:43 jani-HP-Compaq-8200-Elite-CMT-PC sudoedit: pam_unix(sudo:session): session opened for user root by jani(uid=0)
Apr 28 14:24:55 jani-HP-Compaq-8200-Elite-CMT-PC sudoedit: pam_unix(sudo:session): session closed for user root
Apr 28 14:26:03 jani-HP-Compaq-8200-Elite-CMT-PC sudo:     jani : unable to resolve host herra
Apr 28 14:26:03 jani-HP-Compaq-8200-Elite-CMT-PC sudo:     jani : TTY=pts/2 ; PWD=/etc/puppet/modules/orja/manifests ; USER=root ; COMMAND=sudoedit init.pp
Apr 28 14:26:03 jani-HP-Compaq-8200-Elite-CMT-PC sudoedit: pam_unix(sudo:session): session opened for user root by jani(uid=0)
Apr 28 14:26:23 jani-HP-Compaq-8200-Elite-CMT-PC sudoedit: pam_unix(sudo:session): session closed for user root
Apr 28 14:29:53 jani-HP-Compaq-8200-Elite-CMT-PC sudo:     jani : unable to resolve host herra
Apr 28 14:29:53 jani-HP-Compaq-8200-Elite-CMT-PC sudo:     jani : TTY=pts/2 ; PWD=/home/jani ; USER=root ; COMMAND=/usr/bin/puppet cert list -all
Apr 28 14:29:53 jani-HP-Compaq-8200-Elite-CMT-PC sudo: pam_unix(sudo:session): session opened for user root by jani(uid=0)
Apr 28 14:29:54 jani-HP-Compaq-8200-Elite-CMT-PC sudo: pam_unix(sudo:session): session closed for user root

    tail /var/log/auth.log

Apr 28 14:24:43 jani-HP-Compaq-8200-Elite-CMT-PC sudoedit: pam_unix(sudo:session): session opened for user root by jani(uid=0)
Apr 28 14:24:55 jani-HP-Compaq-8200-Elite-CMT-PC sudoedit: pam_unix(sudo:session): session closed for user root
Apr 28 14:26:03 jani-HP-Compaq-8200-Elite-CMT-PC sudo:     jani : unable to resolve host herra
Apr 28 14:26:03 jani-HP-Compaq-8200-Elite-CMT-PC sudo:     jani : TTY=pts/2 ; PWD=/etc/puppet/modules/orja/manifests ; USER=root ; COMMAND=sudoedit init.pp
Apr 28 14:26:03 jani-HP-Compaq-8200-Elite-CMT-PC sudoedit: pam_unix(sudo:session): session opened for user root by jani(uid=0)
Apr 28 14:26:23 jani-HP-Compaq-8200-Elite-CMT-PC sudoedit: pam_unix(sudo:session): session closed for user root
Apr 28 14:29:53 jani-HP-Compaq-8200-Elite-CMT-PC sudo:     jani : unable to resolve host herra
Apr 28 14:29:53 jani-HP-Compaq-8200-Elite-CMT-PC sudo:     jani : TTY=pts/2 ; PWD=/home/jani ; USER=root ; COMMAND=/usr/bin/puppet cert list -all
Apr 28 14:29:53 jani-HP-Compaq-8200-Elite-CMT-PC sudo: pam_unix(sudo:session): session opened for user root by jani(uid=0)
Apr 28 14:29:54 jani-HP-Compaq-8200-Elite-CMT-PC sudo: pam_unix(sudo:session): session closed for user root
jani@pekka:~$ tail /var/log/syslog
Apr 28 14:24:18 jani-HP-Compaq-8200-Elite-CMT-PC puppet-master[8023]: ' at /etc/puppet/modules/orja/manifests/init.pp:10 on node orja.tielab.haaga-helia.fi
Apr 28 14:24:18 jani-HP-Compaq-8200-Elite-CMT-PC puppet-master[8023]: Unclosed quote after '' in 'Heippa!,
Apr 28 14:24:18 jani-HP-Compaq-8200-Elite-CMT-PC puppet-master[8023]: #011}
Apr 28 14:24:18 jani-HP-Compaq-8200-Elite-CMT-PC puppet-master[8023]: }
Apr 28 14:24:18 jani-HP-Compaq-8200-Elite-CMT-PC puppet-master[8023]: ' at /etc/puppet/modules/orja/manifests/init.pp:10 on node orja.tielab.haaga-helia.fi
Apr 28 14:24:18 jani-HP-Compaq-8200-Elite-CMT-PC puppet-master[8023]: Unclosed quote after '' in 'Heippa!,
Apr 28 14:24:18 jani-HP-Compaq-8200-Elite-CMT-PC puppet-master[8023]: #011}
Apr 28 14:24:18 jani-HP-Compaq-8200-Elite-CMT-PC puppet-master[8023]: }
Apr 28 14:24:18 jani-HP-Compaq-8200-Elite-CMT-PC puppet-master[8023]: ' at /etc/puppet/modules/orja/manifests/init.pp:10 on node orja.tielab.haaga-helia.fi
Apr 28 14:25:03 jani-HP-Compaq-8200-Elite-CMT-PC puppet-master[8023]: Compiled catalog for orja.tielab.haaga-helia.fi in environment production in 0.28 seconds


