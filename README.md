# slaveMaster
H4 palvelinten hallinta

Tehtävänanto sivulla http://terokarvinen.com/2017/aikataulu-%e2%80%93-palvelinten-hallinta-ict4tn022-2-%e2%80%93-5-op-uusi-ops-loppukevat-2017-p2
h4. a) Master-slave. Aloita tyhjästä koneesta. Tee yhdestä koneesta orja ja toisesta herra. Kokeile, että orja saa herralta modulin. (Voit käyttää labraa, kun se on tyhjä. Laita mukaan ‘puppet cert –list –all’, ‘tail /var/log/auth.log’, ‘tail /var/log/syslog’).

b) Varaa omalle modulillesi aihe kommenttina tälle sivulle. Yksi työ yhdestä aiheesta. 

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
 
 



