
DEBIAN 10/11 (Create CT dari proxmox) IP : 103.103.100.200
=
```
#apt update && apt upgrade -y
```
```
#nano /etc/resolv.conf
```
**Copy paste kedalam resolv.conf**
```
search blok.domain.com
nameserver 103.103.100.200
nameserver 8.8.8.8
```

```
#apt-get install bind9 dnsutils apache2 -y
```
```
#cd /etc/bind
```
```
#nano named.conf.local
```

**Copy Paste ke dalam named.conf.local**
```
zone "blok.domain.com" {
	type master;
	file "/etc/bind/db.domain";
};
zone "100.103.103.in-addr.arpa"{
	type master;
	file "/etc/bind/db.200";
};
```
```
#cp db.local db.domain
```
```
#cp db.127 db.200
```
```
#nano db.domain
```
**Copy paste ke dalam db.domain**
```
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	blok.domain.com. root.blok.domain.com. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	blok.domain.com.
@	IN	A	103.103.100.200
www	IN	A	103.103.100.200
```
```
#nano db.200
```
**Copt paste ke dalam db.200**
```
;
; BIND reverse data file for local loopback interface
;
$TTL	604800
@	IN	SOA	blok.domain.com. root.blok.domain.com. (
			      1		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	blok.domain.com.
200	IN	PTR	blok.domain.com.
```
```
#/etc/init.d/bind9 restart
```

**Cek reserve sudah benar apa belum**
```
#nslookup blok.domain.com
```
Server : 103.103.100.200
Address : 103.103.100.200#53

Name : blok.domain.com
Address : 103.103.100.200

```
#nslookup 103.103.100.200
```
Server : 103.103.100.200
Address : 103.103.100.200#53

200.100.103.103.in-addr.arpa 	name : blok.domain.com

**copy and replace named.conf.option ke folder /etc/bind**

```
#cp db.local db.rpz
```
```
#nano db.rpz
```
**Copy paste kedalam db.rpz**
```
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	rpz.zone. root.rpz.zone. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	rpz.zone.
@	IN	A	103.103.100.200
www	IN	A	103.103.100.200
```

**Download file list domain yang akan di blok dari kominfo**
```
#wget https://trustpositif.kominfo.go.id/assets/db/domains_isp --no-check-certificate
```
```
#awk '{print $1" IN CNAME www"}' domains_isp >> /etc/bind/db.rpz
#awk '{print "*."$1" IN CNAME www"}' domains_isp >> /etc/bind/db.rpz
```
```
#/etc/init.d/bind9 restart
```
**Test**
```
#nslookup cerdas.com 103.103.100.200
```
```
#dig @103.103.100.200 cerdas.com
```
