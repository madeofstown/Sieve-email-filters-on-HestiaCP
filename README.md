# Sieve-email-filters-on-HestiaCP
 First
 
    apt update && apt upgrade
 then wait for the process to finish and the REBOOT

Second
add the dovecot repo

    curl https://repo.dovecot.org/DOVECOT-REPO-GPG | gpg --import
    gpg --export ED409DA1 > /etc/apt/trusted.gpg.d/dovecot.gpg
 and then

     apt-get install dovecot-sieve dovecot-managesieved dovecot-lmtpd

Finally, Follow these Directions

(1) Edit /etc/dovecot/conf.d/20-managesieve.conf

Uncomment the following line to enable the sieve protocol:

```
protocols = $protocols sieve

```

If any of the following lines are commented, uncomment them:

```
service managesieve-login {
  inet_listener sieve {
    port = 4190
  }

  service_count = 1
  process_min_avail = 0
}

service managesieve {
  process_limit = 1024
}

protocol sieve {
  managesieve_max_line_length = 65536
  managesieve_implementation_string = Dovecot Pigeonhole
}

```

(2) Create global sieve directory along with empty files where rules can be placed that will be executed before or after the user’s rules, respectively.

```
mkdir /etc/dovecot/sieve
touch /etc/dovecot/sieve/before.sieve
touch /etc/dovecot/sieve/after.sieve

```

(3) Edit /etc/dovecot/sieve/default.sieve and paste the following:

```
require ["fileinto", "regex", "date", "relational", "vacation", "imap4flags", "envelope", "subaddress", "copy", "reject"];

# rule:[Spam Filter]
if anyof (header :contains "X-Spam-Flag" "YES", header :contains "X-Spam" "Yes") {                                                                                    
  fileinto "Junk";
  stop;
} 

```

(4) Edit /etc/dovecot/conf.d/90-sieve.conf

Uncomment the “sieve” line and set its value to the following:

```
sieve = file:~/mail/%d/%n/sieve/;active=~/mail/%d/%n/sieve/managed.sieve

```

(That will result in a “sieve” directory being automatically created in the user’s mail directory the first time the RoundCube “Filters” tab in Settings is clicked, and the “default.sieve” copied to it as the first rule.)

Uncomment the “sieve_before” and “sieve_after” lines and set them to the following:

```
sieve_before = /etc/dovecot/sieve/before.sieve
sieve_after = /etc/dovecot/sieve/after.sieve

```

(5) Edit /etc/roundcube/config.inc.php

Go to the $rcmail_config[‘plugins’] line and add ‘managedsieve’ to the list of plugins:

```
$rcmail_config['plugins'] = array('jqueryui','password','managesieve');

```

Then add the following lines below that one:

```
// Dovecot managedsieve TCP port
$rcmail_config['managesieve_port'] = 4190;                                               
                              
// Default contents of filters script (eg. default spam filter)
$rcmail_config['managesieve_default'] = '/etc/dovecot/sieve/default.sieve';
```
