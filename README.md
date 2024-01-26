Script para políticas RPZ/BIND9 que utiliza o site <a href="https://anablock.net.br" rel="noopener" target="_blank"><strong>https://anablock.net.br</strong></a>, uma plataforma desenvolvida por <a href="http://www.patrickbrandao.com" rel="noopener" target="_blank"><strong>Patrick Brandão</strong></a> da <a href="https://www.nuva.com.br" rel="noopener" target="_blank"><strong>Nuva</strong></a>. Ele projetou uma página abrangente que oferece todas as informações necessárias para acompanhar os bloqueios de conteúdo regulados pela Anatel e pelas autoridades da República Federativa do Brasil.

O script utilizará esses dados para automatizar o bloqueio dos domínios solicitados pela Anatel.

Tutorial https://blog.remontti.com.br/7759

# Comandos resumidos 
```
# apt install python3 python3-requests tree
```
## Bind
```
# /etc/bind/named.conf.local
```
```
zone "rpz.zone" {
        type master;
        file "/var/cache/bind/rpz/db.rpz.zone.hosts";
        allow-query { none; };
        allow-transfer { 10.51.51.3; };
        also-notify { 10.51.51.3; };
};
```
```
# mkdir /var/cache/bind/rpz/
```
```
# vim /etc/bind/named.conf.options
```
```
options {
//...
    response-policy {
      zone "rpz.zone" policy CNAME localhost;
    };
//...
```
```
# mkdir /etc/bind/scripts
# cd /etc/bind/scripts
# wget https://raw.githubusercontent.com/remontti/anablock_bind9/main/anablock_bind9.py
```

```
# python3 /etc/bind/scripts/anablock_bind9.py localhost
# echo '00 00   * * *   root    python3 /etc/bind/scripts/anablock_bind9.py bloqueadonobrasil.remontti.com.br'\ >> /etc/crontab
# systemctl restart cron
```

## Unbound

Adaptado por https://t.me/paulojrandrade
```
# mkdir /var/cache/unbound
# mkdir /var/cache/unbound/rpz
# touch /var/cache/unbound/rpz/db.rpz.zone.hosts
# ln -s /var/cache/unbound/rpz/db.rpz.zone.hosts /etc/unbound/db.rpz.zone.hosts
```

```
# vim /etc/unbound/unbound.conf
```

```
rpz:
    name: rpz.dominio.com.br
    zonefile: /etc/unbound/db.rpz.block.zone.hosts
    rpz-action-override: cname
    rpz-cname-override: "sub.dominio.com.br."
```

```
# wget https://raw.githubusercontent.com/remontti/anablock_bind9/main/anablock_unbound.py -O /etc/unbound/anablock_unbound.py
# python3 /etc/unbound/anablock_unbound.py sub.dominio.com.br
```

```
# vim /etc/crontabs/root
```

```
00 00   * * *   root    python3 /etc/unbound/anablock_unbound.py sub.dominio.com.br
```

```
# systemctl restart cron
```







