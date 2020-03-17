## [BlackIP](https://www.maravento.com/p/blackip.html)

**BlackIP** is a project that collects and unifies public blacklists of IP addresses, to make them compatible with [Squid](http://www.squid-cache.org/) and [IPSET](http://ipset.netfilter.org/) ([Iptables](http://www.netfilter.org/documentation/HOWTO/es/packet-filtering-HOWTO-7.html) [Netfilter](http://www.netfilter.org/))

**BlackIP** es un proyecto que recopila y unifica listas negras públicas de direcciones IPs, para hacerlas compatibles con [Squid](http://www.squid-cache.org/) e [IPSET](http://ipset.netfilter.org/) ([Iptables](http://www.netfilter.org/documentation/HOWTO/es/packet-filtering-HOWTO-7.html) [Netfilter](http://www.netfilter.org/))

### DATA SHEET
---

|lst|Black IPs|txt size|tar.gz size|
| :---: | :---: | :---: | :---: |
|blackip.txt|3.170.977|45.3 Mb|9.4 Mb|

### DEPENDENCIES
---

```
git ipset iptables bash tar zip wget squid subversion python ulogd2
```

### GIT CLONE
---
```
git clone --depth=1 https://github.com/maravento/blackip.git
```

### HOW TO USE
---

`blackip.txt` is already optimized. Download it and unzip it in the path of your preference / `blackip.txt` ya viene optimizada. Descárguela y descomprimala en la ruta de su preferencia

#####  Download and Checksum

```
wget -q -N https://raw.githubusercontent.com/maravento/blackip/master/blackip.tar.gz && cat blackip.tar.gz* | tar xzf -
wget -q -N https://raw.githubusercontent.com/maravento/blackip/master/checksum.md5
md5sum blackip.txt | awk '{print $1}' && cat checksum.md5 | awk '{print $1}'
```

### IPSET-SQUID RULES
---

#### [IPSET](http://ipset.netfilter.org/) Rules

This module allows us to perform mass filtering, at a processing speed far superior to other Solutions (See the [benchmark](https://web.archive.org/web/20161014210553/http://daemonkeeper.net/781/mass-blocking-ip-addresses-with-ipset/)). It includes geographical areas with [IPDeny](http://www.ipdeny.com/ipblocks/)) / Este módulo nos permite realizar filtrado masivo, a una velocidad de procesamiento muy superior a otras soluciones (Vea el [benchmark](https://web.archive.org/web/20161014210553/http://daemonkeeper.net/781/mass-blocking-ip-addresses-with-ipset/)). Se incluye zonas geográficas con [IPDeny](http://www.ipdeny.com/ipblocks/))

Edit your Iptables script and add the following lines: / Edite su script de Iptables y agregue las siguientes líneas:
```
# IPSET BLACKZONE (select country to block and ip/range) ###
# http://www.ipdeny.com/ipblocks/
ipset=/sbin/ipset
iptables=/sbin/iptables
route=/path_to_blackip/
zone=/path_to_zones/zones
if [ ! -d $zone ]; then mkdir -p $zone; fi

$ipset -F
$ipset -N -! blackzone hash:net maxelem 1000000
# Uncomment this line if you want to block entire countries
#for ip in $(cat $zone/{cn,ru}.zone $route/blackip.txt); do
# Uncomment this line if you want to block only ips (recommended)
for ip in $(cat $route/blackip.txt); do
    $ipset -A blackzone $ip
done
$iptables -t mangle -A PREROUTING -m set --match-set blackzone src -j NFLOG --nflog-prefix 'Blackzone Block'
$iptables -t mangle -A PREROUTING -m set --match-set blackzone src -j DROP
$iptables -A FORWARD -m set --match-set blackzone dst -j NFLOG --nflog-prefix 'Blackzone Block'
$iptables -A FORWARD -m set --match-set blackzone dst -j DROP
```
You can block entire countries ranges (e.g. China, Rusia, etc) with [IPDeny](http://www.ipdeny.com/ipblocks/) adding the countries to the line: / Puede incluir rangos completos de países (e.g. China, Rusia, etc) con [IPDeny](http://www.ipdeny.com/ipblocks/) agregando los países a la línea:
```
for ip in $(cat $zone/{cn,ru}.zone $route/blackip.txt); do
```
In case of error or conflict, execute: / En caso de error o conflicto, ejecute:
```
sudo ipset flush blackzone # (or: sudo ipset flush)
```
NFLOG: /var/log/ulog/syslogemu.log
```
chown root:root /var/log
apt -y install ulogd2
if [ ! -d /var/log/ulog/syslogemu.log ]; then mkdir -p /var/log/ulog && touch /var/log/ulog/syslogemu.log; fi
usermod -a -G ulog $USER
```

#### [Squid](http://www.squid-cache.org/) Rule

Edit:
```
/etc/squid/squid.conf
```
And add the following lines: / Y agregue las siguientes líneas:

```
# INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS
acl blackip dst "/path_to/blackip.txt"
http_access deny blackip
```

#### Important about BlackIP

- Should not be used `blackip.txt` in [IPSET](http://ipset.netfilter.org/) and in [Squid](http://www.squid-cache.org/) at the same time (double filtrate) / No debe utilizar `blackip.txt` en [IPSET](http://ipset.netfilter.org/) y en [Squid](http://www.squid-cache.org/) al mismo tiempo (doble filtrado)
- `blackip.txt` is a list IPv4. Does not include CIDR / `blackip.txt` es una lista IPv4. No incluye CIDR
- `blackip.txt` does not include private/reserved ranges [RFC1918](https://en.wikipedia.org/wiki/Private_network) (`ianacidr.txt`) / `blackip.txt` no incluye rangos privados/reservados [RFC1918](https://es.wikipedia.org/wiki/Red_privada) (`ianacidr.txt`)
- `blackip.txt` has been tested in Squid v3.5.x / `blackip.txt` ha sido testeada en Squid v3.5.x

#### [Squid-Cache](http://www.squid-cache.org/) Advanced Rules

**Blackip** contains millions of IP addresses, therefore it is recommended: / **Blackip** contiene millones de direcciones IP, por tanto se recomienda:

- Use `betra.txt` to add IP/CIDR that are not in `blackip.txt` (By default it contains some BlackCIDR) / Use `bwextra.txt` para agregar IP/CIDR que no se encuentren en `blackip.txt` (Por defecto contiene algunos BlackCIDR)
- Use `whiteip.txt`; a white list of IPv4 IP addresses (Hotmail, Gmail, Yahoo. Etc) / Use `whiteip.txt`; una lista blanca de direcciones IPs IPv4 (Hotmail, Gmail, Yahoo. etc)
- Use `wextra.txt` to add whitelists of IP/CIDRs that are not included in` whiteip.txt` / Use `wextra.txt` para agregar listas blancas de IP/CIDR que no están incluidas en `whiteip.txt`
- To increase security, close Squid to any other request to IP addresses. Very useful for blocking anonymizers / Para incrementar la seguridad, cierre Squid a cualquier otra petición a direcciones IP. Muy útil para el bloqueo de anonimizadores

```
# INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS
# blackip rules
acl bextra dst "/path_to/bextra.txt"
http_access deny bextra
acl blackip dst "/path_to/blackip.txt"
http_access deny blackip
# whiteip rules
acl wextra dst "/path_to/wextra.txt"
http_access allow wextra
acl whiteip dst "/path_to/whiteip.txt"
http_access allow whiteip
# deny all IPs
acl no_ip url_regex -i [0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}
http_access deny no_ip
```

### UPDATE
---

#### ⚠️ WARNING: BEFORE YOU CONTINUE!

Update and debugging can take and consume many hardware resources and bandwidth. It is not recommended to run it on production equipment / La actualización y depuración puede tardar y consumir muchos recursos de hardware y ancho de banda. No se recomienda ejecutarla en equipos en producción

##### BlackIP Update

>The update process of `blackip.txt` is executed in sequence by the script `bipupdate.sh` / El proceso de actualización de `blackip.txt` es ejecutado en secuencia por el script `bipupdate.sh`

```
wget -q -N https://raw.githubusercontent.com/maravento/blackip/master/bipupdate/bipupdate.sh && chmod +x bipupdate.sh && ./bipupdate.sh
```

##### Important about BlackIP Update

- `tw.txt` containing IPs of teamviewer servers. By default they are commented. To block or authorize them, activate them in `bipupdate.sh`. To update it use `tw.sh` / `tw.txt` contiene IPs de servidores teamviewer. Por defecto están comentadas. Para bloquearlas o autorizarlas activelas en `bipupdate.sh`. Para actualizarla use `tw.sh`
- You must activate the rules in [Squid](http://www.squid-cache.org/) before using `bipupdate.sh` / Antes de utilizar `bipupdate.sh` debe activar las reglas en [Squid](http://www.squid-cache.org/)

##### Check execution (/var/log/syslog):

```
BlackIP: Done 06/05/2019 15:47:14
```

##### WhiteIP Update

>`whiteip.txt` is already updated and optimized. The update process of `whiteip.txt` is executed in sequence by the script `wipupdate.sh` / `whiteip.txt` ya esta actualizada y optimizada. El proceso de actualización de `whiteip.txt` es ejecutado en secuencia por el script `wipupdate.sh`

```
wget -q -N https://raw.githubusercontent.com/maravento/blackip/master/bipupdate/wlst/wipupdate.sh && chmod +x wipupdate.sh && ./wipupdate.sh
```

### SOURCES
---

##### Black IPs

###### Actives

- [Abuse.ch Feodo Tracker](https://feodotracker.abuse.ch/blocklist/?download=ipblocklist)
- [adservers yoyo](https://pgl.yoyo.org/adservers/iplist.php?format=&showintro=0)
- [BL Myip](https://myip.ms/files/blacklist/general/full_blacklist_database.zip)
- [Cinsscore](http://cinsscore.com/list/ci-badguys.txt)
- [Emerging Threats Block](http://rules.emergingthreats.net/fwrules/emerging-Block-IPs.txt)
- [Emerging Threats compromised](http://rules.emergingthreats.net/blockrules/compromised-ips.txt)
- [Firehold Forus Spam](https://raw.githubusercontent.com/firehol/blocklist-ipsets/master/stopforumspam_7d.ipset)
- [Greensnow](http://blocklist.greensnow.co/greensnow.txt)
- [IPDeny](http://www.ipdeny.com/ipblocks/)
- [Malc0de IP Blacklist](http://malc0de.com/bl/IP_Blacklist.txt)
- [Malwaredomain IP List](https://www.malwaredomainlist.com/hostslist/ip.txt)
- [Maxmind](https://www.maxmind.com/es/proxy-detection-sample-list)
- [MyIP BL](https://myip.ms/files/blacklist/general/latest_blacklist.txt)
- [Open BL](http://www.openbl.org/lists/base.txt)
- [opsxcq proxy-list](https://raw.githubusercontent.com/opsxcq/proxy-list/master/list.txt)
- [Project Honeypot](https://www.projecthoneypot.org/list_of_ips.php?t=d&rss=1)
- [Ransomwaretracker](https://ransomwaretracker.abuse.ch/downloads/RW_IPBL.txt)
- [Rulez BruteForceBlocker](http://danger.rulez.sk/projects/bruteforceblocker/blist.php)
- [Spamhaus](https://www.spamhaus.org/drop/drop.lasso)
- [StopForumSpam 180](https://www.stopforumspam.com/downloads/listed_ip_180_all.zip)
- [The LashBack UBL](http://www.unsubscore.com/blacklist.txt)
- [TOR BulkExitList](https://check.torproject.org/cgi-bin/TorBulkExitList.py?ip=1.1.1.1)
- [TOR Node List](https://www.dan.me.uk/torlist/?exit)
- [Ultimate Hosts IPs Blacklist](https://github.com/mitchellkrogza/Ultimate.Hosts.Blacklist). [Mirror](https://hosts.ubuntu101.co.za/ips.list)
- [Zeustracker](https://zeustracker.abuse.ch/blocklist.php?download=badips)

###### Inactive

- [Blocklist](https://lists.blocklist.de/lists/all.txt) and [Blocklist Export](https://www.blocklist.de/downloads/export-ips_all.txt). Replaced by [Ultimate Hosts IPs Blacklist](https://github.com/mitchellkrogza/Ultimate.Hosts.Blacklist)
- [Firehold Level 1](https://raw.githubusercontent.com/firehol/blocklist-ipsets/master/firehol_level1.netset) (Excluded for containing CIDR)
- [OpenBL](https://www.openbl.org/lists/base.txt) (Server Down. Last Updated Known Apr 1/2016. Added to: `oldips.txt`)
- [StopForumSpam Toxic CIDR](https://www.stopforumspam.com/downloads/toxic_ip_cidr.txt) (Excluded for containing CIDR)
- [UCEPROTECT IP Blacklists / BACKSCATTERER.ORG Blacklist](http://wget-mirrors.uceprotect.net/) (Server Down. Last Updated Known Ago 17/2017. Added to: `oldips.txt`)

##### White IPs

###### Actives

- [Amazon AWS](https://ip-ranges.amazonaws.com/ip-ranges.json) (Excluded for containing CIDR)
- [Microsoft Azure Datacenter](https://www.microsoft.com/en-us/download/details.aspx?id=41653) (Excluded for containing CIDR)

###### Inactives

- [O365IPAddresses](https://support.content.office.net/en-us/static/O365IPAddresses.xml) (No longer support. [See This post](ocs.microsoft.com/es-es/office365/enterprise/urls-and-ip-address-ranges?redirectSourcePath=%252fen-us%252farticle%252fOffice-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2))

##### Work Lists

###### Internals

- [Black IP/CIDR Extra](https://raw.githubusercontent.com/maravento/blackip/master/bipupdate/blst/bextra.txt)
- [IANA CIDR](https://raw.githubusercontent.com/maravento/blackip/master/bipupdate/wlst/ianacidr.txt)
- [Old IPs](https://raw.githubusercontent.com/maravento/blackip/master/bipupdate/blst/oldips.txt)
- [Teamviewer IPs](https://raw.githubusercontent.com/maravento/blackip/master/bipupdate/wlst/tw.txt)
- [White IP/CIDR extra](https://raw.githubusercontent.com/maravento/blackip/master/bipupdate/wlst/wextra.txt)
- [White IPs](https://raw.githubusercontent.com/maravento/blackip/master/bipupdate/wlst/whiteips.txt)

###### Externals

- [White URLs](https://raw.githubusercontent.com/maravento/blackweb/master/bwupdate/lst/whiteurls.txt)

##### Work Tools

###### Internals

- [cidr2ip](https://raw.githubusercontent.com/maravento/blackip/master/bipupdate/tools/cidr2ip.py)
- [Debug IPs](https://raw.githubusercontent.com/maravento/blackip/master/bipupdate/tools/debugbip.py)
- [Teamviewer Capture](https://raw.githubusercontent.com/maravento/blackip/master/bipupdate/wlst/tw.sh)


### CONTRIBUTIONS
---

We thank all those who contributed to this project. Those interested may contribute sending us new "Blacklist" links to be included in this project / Agradecemos a todos aquellos que han contribuido a este proyecto. Los interesados pueden contribuir, enviándonos enlaces de nuevas "Blacklist", para ser incluidas en este proyecto

Special thanks to: [Jhonatan Sneider](https://github.com/sney2002)

### DONATE
---

BTC: 3M84UKpz8AwwPADiYGQjT9spPKCvbqm4Bc

### LICENCES
---

[![GPL-3.0](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl.txt)

[![CreativeCommons](https://licensebuttons.net/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)
[maravento.com](http://www.maravento.com) is licensed under a [Creative Commons Reconocimiento-CompartirIgual 4.0 Internacional License](http://creativecommons.org/licenses/by-sa/4.0/).

© 2020 [Maravento Studio](http://www.maravento.com)

### DISCLAIMER
---

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
