### POSTFIX
```
apt -y install postfix telnet mutt
postfix status
service postfix restart
less /var/log/mail.log

# každý uživatel, kterému má být možno poslat mail, musí být ve skupině mail
useradd -m -s /bin/bash user1
passwd user1
usermod -G mail user1
mkdir /home/user1/Maildir
chown user1:user1 /home/user1/Maildir
chmod -R 700 /home/user1/Maildir 

# Modifikace DNS pro maily (domena test.spos, adresa 192.168.0.106):
mail    IN      A       192.168.0.106
@       IN      MX 10   mail

cat /etc/mailname  (vypise mail.test.spos)
vim /etc/postfix/main.cf
myorigin = test.spos
myhostname = test.spos
mydomain = test.spos
mydestination = $myhostname, posta.test.spos, green, localhost.localdomain, localhost
mynetworks = 192.168.0.0/24 127.0.0.0/8 
home_mailbox = Maildir/
mailbox_command =

Cert.
openssl genrsa -des3 -out server.key 1024
openssl req -new -key server.key -out server.csr
cp server.key server.key.org
openssl rsa -in server.key.org -out server.key
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
cp server.crt /etc/ssl
cp server.key /etc/ssl

service postfix restart
echo "Test2" | mail -s "Testovaci mail2" radekj@test.spos
ls -l /var/mail/
mutt -f /home/user2/Maildir

telnet localhost 25
  HELO test.spos
  MAIL FROM: pepa@gmail.xx
  RCPT TO: radekj@test.spos
  DATA
  Subject: Testovaci
  Ahoj svete
  .
  QUIT

cat /var/mail/radekj 
cat /var/spool/mail/radekj 

cat /etc/aliases:
hostmaster:	user1
webmaster:	user2

newaliases
```



### DOVECOT
```
apt -y install dovecot-pop3d telnet
apt -y install dovecot-imapd
netstat -tulpn | grep :143
netstat -tulpn | grep :110

vim /etc/dovecot/dovecot.conf
protocols = imap pop3
listen = *, ::

vim /etc/dovecot/conf.d/10-mail.conf
mail_location = mbox:~/mail:INBOX=/var/mail/%u
mail_location = maildir:~/Maildir

vim /etc/dovecot/conf.d/10-ssl.conf 
ssl= yes | no | required
ssl_disable = no
ssl_cert_file = </etc/ssl/server.crt 
ssl_key_file = </etc/ssl/server.key

# nebo pouzit
ssl_cert = </etc/ssl/certs/ssl-cert-snakeoil.pem
ssl_key = </etc/ssl/private/ssl-cert-snakeoil.key

Čtení pošty šifrovaně přes IMAP (nebo POP3), mutt:
openssl genrsa -des3 -out server.key 1024
openssl req -new -key server.key -out server.csr
cp server.key server.key.org
openssl rsa -in server.key.org -out server.key
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
cp server.crt /etc/ssl
cp server.key /etc/ssl

useradd -m -s /bin/bash john
mkdir /home/john/Maildir
chown john:john /home/john/Maildir
chmod -R 700 /home/john/Maildir 

service dovecot restart
```


### MUTT - prohlížení mailů
```
apt-get install mutt
mutt -f /home/user2/Maildir

# pres IMAP
~/.muttrc (v home adresáři uživatele, ze kterého se přihlašujeme)
	- pro imap
	set folder = imap://user:pass@localhost:143/
	set spoolfile = imap://user:pass@localhost:143/
	- pro pop3
	set folder = pop://user:pass@localhost:110/
	set spoolfile = pop://user:pass@localhost:110/
user - uživatel, na kterého se přihlašujeme
chmod 777 .muttrc

```

### test
```
# Test POP3 služby:
telnet localhost 110
# https://tools.ietf.org/html/rfc1939
USER jindra
PASS jindra123
LIST
RETR 1
DELE 1
QUIT

# Dovecot pro IMAP - !!sdílí předchozí nastavení z POP3
telnet localhost 143
# https://tools.ietf.org/html/rfc3501
1 LOGIN jindra jindra123
2 select inbox
5 FETCH 1 BODY[]
6 LOGOUT

netstat -tulpn | grep :143
openssl s_client -connect localhost:995
openssl s_client -connect mail.test.spos:993

SMTP 25; POP3 110; IMAP 143; IMAPS 993; POP3S 995

ufw reset
ufw allow 22/tcp
ufw allow from 192.168.0.0/24 to any port 110		// povolit z LAN
ufw allow from 147.228.0.0/16 to any port 995		// povolit z ZCU
ufw default deny
ufw enable
```
