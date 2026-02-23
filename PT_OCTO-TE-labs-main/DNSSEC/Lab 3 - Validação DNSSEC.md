
<div style="display: flex; gap: 700px;">
<img src="https://github.com/assisg-git/GHR.DNSSEC_LabPT/blob/main/PT_OCTO-TE-labs-main/_img/LOGO_ICANN.jpg" alt="ICANN" style="width: 40%;"/>
<img src="https://github.com/assisg-git/GHR.DNSSEC_LabPT/blob/main/PT_OCTO-TE-labs-main/_img/LOGO_PT.png" alt=".PT"  style="width: 40%;"/>
</div>

------
```
Criado por: Yazid AKANHO
Modificado por: - Assis Guerreiro .PT

Versão atual: 2025120300 
Versão anterior: 2024020400
```
------


Durante esta prática vamos aceder ao seguinte equipamento:

* **grpX-cli** : cliente
* **grpX-resolv1** & **grpX-resolv2** : recursive servers (resolvers)


### Configurar um servidor recursivo de validação BIND

Usamos o contentor "Resolv 1" (servidor recursivo) [**grpX-resolv1**]. Este contentor já tem os pacotes BIND9 descarregados e instalados.

Mudar para o utilizador root:

```
sudo su -
```

Ir ao diretório /etc/bind:

```
cd /etc/bind
```

Neste ponto devemos configurar algumas opções BIND9. 
Para fazer isto, editar o ficheiro `/etc/bind/named.conf.options`:

```
nano named.conf.options
```

Depois adicionar as opções para indicar (ao resolver) os endereços IP que têm permissão para consultar este resolver, e os endereços IP nos quais o nosso servidor estará a escutar (porta 53). O ficheiro deve ficar como se segue:

```
options {
	directory "/var/cache/bind";

	// If there is a firewall between you and nameservers you want
	// to talk to, you may need to fix the firewall to allow multiple
	// ports to talk. See http://www.kb.cert.org/vuls/id/800113

	// If your ISP provided one or more IP addresses for stable 
	// nameservers, you probably want to use them as forwarders.  
	// Uncomment the following block, and insert the addresses replacing 
	// the all-0's placeholder.

	// forwarders {
	// 	0.0.0.0;
	// };

	//========================================================================
	// If BIND logs error messages about the root key being expired,
	// you will need to update your keys. See https://www.isc.org/bind-keys
	//========================================================================
	dnssec-validation auto;
	listen-on port 53 { localhost; 100.100.0.0/16; };
	listen-on-v6 port 53 { localhost; fd89:59e0::/32; };
	allow-query { localhost; 100.100.0.0/16; fd89:59e0::/32; };
	recursion yes;
};
```

Quando terminar de editar o ficheiro de configuração, verifique a sintaxe da configuração:

```
named-checkconf
```

Depois reiniciar o servidor para que aceite as alterações de configuração:

```
systemctl restart named
```

Verificar o estado do processo bind9:

```
systemctl status named
```

Devem obter algo semelhante ao abaixo:

```
● named.service - BIND Domain Name Server
   Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/service.d
       └─lxc.conf
   Active: **active (running)** since Thu 2021-05-13 01:38:27 UTC; 4s ago
    Docs: man:named(8)
  Main PID: 849 (named)
   Tasks: 50 (limit: 152822)
   Memory: 103.2M
   CGroup: /system.slice/named.service
       └─849 /usr/sbin/named -f -u bind

May 13 01:38:27 resolv1.grpX.dnssec.lab.dns.pt named[849]: **command channel listening on ::1#953**
May 13 01:38:27 resolv1.grpX.dnssec.lab.dns.pt named[849]: managed-keys-zone: loaded serial 6
May 13 01:38:27 resolv1.grpX.dnssec.lab.dns.pt named[849]: zone 0.in-addr.arpa/IN: loaded serial 1
May 13 01:38:27 resolv1.grpX.dnssec.lab.dns.8..pt named[849]: zone 127.in-addr.arpa/IN: loaded serial 1
May 13 01:38:27 resolv1.grpX.dnssec.lab.dns.pt named[849]: zone localhost/IN: loaded serial 2
May 13 01:38:27 resolv1.grpX.dnssec.lab.dns.pt named[849]: zone 255.in-addr.arpa/IN: loaded serial 1
May 13 01:38:27 resolv1.grpX.dnssec.lab.dns.pt named[849]: **all zones loaded**
May 13 01:38:27 resolv1.grpX.dnssec.lab.dns.pt named[849]: **running**
May 13 01:38:27 resolv1.grpX.dnssec.lab.dns.pt named[849]: managed-keys-zone: Key 20326 for zone . is now trusted (acceptance timer>
May 13 01:38:27 resolv1.grpX.dnssec.lab.dns.pt named[849]: resolver priming query complete
```



Usamos o contentor "Resolv 1" (servidor recursivo) [**grpX-cli**]. 
### Testar o vosso novo resolver de validação

Executar os seguintes comandos e confirmar se recebem a flag "ad":

1. dig SOA com. @100.100.X.67 
2. dig SOA com. @100.100.X.67 +dnssec
3. dig A www.icann.org @100.100.X.67
4. dig NS icann.org @100.100.X.67
5. dig DNSKEY *grpX*.dnssec.lab.dns.pt @100.100.X.67
6. dig NS *grpX*.dnssec.lab.dns.pt @100.100.X.67 +dnssec
7. dig SOA *grpX*.dnssec.lab.dns.pt @100.100.X.67
8. dig DNSKEY *grpX*.dnssec.lab.dns.pt @100.100.X.130
9. dig DNSKEY *grpX*.dnssec.lab.dns.pt @100.100.X.130 +multi
10. dig SOA *grpX*.dnssec.lab.dns.pt @100.100.X.131 +dnssec +multi

Receberam a flag "ad" para as últimas três consultas dig? Porquê?

1. dig www.dnssec-failed.org @9.9.9.9
2. dig www.dnssec-failed.org @100.100.X.67 
3. dig www.dnssec-failed.org +dnssec @100.100.X.67
4. dig www.dnssec-failed.org +dnssec +cd @100.100.X.67

O que notam? Porquê?  

### Configurar o resolver recursivo do vosso SO para usar o vosso software resolver recursivo de validação local.

Ainda no servidor **grpX-resolv1**, editar o ficheiro de configuração **/etc/resolv.conf** e substituir o que lá estiver apenas pelo seguinte:


```
nameserver 100.100.X.67
```

Guardar e sair. Depois tentem as seguintes consultas:

1. dig SOA com. 
2. dig SOA com. +dnssec
3. dig A www.icann.org
4. dig NS icann.org
5. dig NS *grpX*.dnssec.lab.dns.pt
6. dig NS *grpX*.dnssec.lab.dns.pt +dnssec
7. dig SOA *grpX*.dnssec.lab.dns.pt
8. dig SOA *grpX*.dnssec.lab.dns.pt +dnssec +multi
9. dig DNSKEY *grpX*.dnssec.lab.dns.pt
10. dig DNSKEY *grpX*.dnssec.lab.dns.pt +multi

Obtiveram a flag "ad" em todos os casos?



## Configurar um servidor recursivo Unbound

Usar o contentor "Resolv 2" (servidor recursivo) [**grpX-resolv2**]. Este contentor já tem os pacotes UNBOUND descarregados e instalados.

Começar por configurar UNBOUND. Para fazer isto, mudar para o utilizador root e editar o ficheiro de configuração **/etc/unbound/unbound.conf**:

```
sudo su -
nano /etc/unbound/unbound.conf
```

Adicionar as opções para indicar (ao resolver) interface IP e porta para escutar consultas, os endereços IP que têm permissão para consultar este resolver, e alguns outros parâmetros. O ficheiro deve ficar como se segue:

```
# Unbound configuration file for Debian.
#
# See the unbound.conf(5) man page.
#
# See /usr/share/doc/unbound/examples/unbound.conf for a commented
# reference config file.
#
# The following line includes additional configuration files from the
# /etc/unbound/unbound.conf.d directory.

server:
        interface: 0.0.0.0
        interface: ::0

        access-control: 127.0.0.0/8 allow
        access-control: 100.100.0.0/16 allow
        access-control: fd89:59e0::/32 allow

        port: 53

        do-udp: yes
        do-tcp: yes
        do-ip4: yes
        do-ip6: yes

include: "/etc/unbound/unbound.conf.d/*.conf"
```

Depois verificar a sintaxe do ficheiro de configuração:

```
unbound-checkconf
```

Se tudo parecer bem, devem obter algo semelhante ao seguinte:

```
unbound-checkconf: no errors in /etc/unbound/unbound.conf
```

Depois reiniciar o servidor para que aceite as alterações de configuração:

```
systemctl restart unbound
```

Verificar o estado do processo UNBOUND:

```
systemctl status unbound
```

Devem obter um resultado semelhante ao seguinte:

```
● unbound.service - Unbound DNS server     
Loaded: loaded (/lib/systemd/system/unbound.service; enabled; vendor preset: enabled)    
Drop-In: /etc/systemd/system/service.d
	└─lxc.conf     Active: active (running) since Thu 2021-05-13 03:49:11 UTC; 13s ago       Docs: man:unbound(8)
	Process: 571 ExecStartPre=/usr/lib/unbound/package-helper chroot_setup (code=exited, status=0/SUCCESS)
	Process: 574 ExecStartPre=/usr/lib/unbound/package-helper root_trust_anchor_update (code=exited, status=0/SUCCESS)   Main PID: 578 (unbound)      Tasks: 1 (limit: 152822)     Memory: 7.8M     
	CGroup: /system.slice/unbound.service             		└─578 /usr/sbin/unbound -d
May 13 03:49:10 resolv2.grpX.dnssec.lab.dns.pt unbound[178]: [178:0] info: [25%]=0 median[50%]=0 [75%]=0
May 13 03:49:10 resolv2.grpX.dnssec.lab.dns.pt unbound[178]: [178:0] info: lower(secs) upper(secs) recursions
May 13 03:49:10 resolv2.grpX.dnssec.lab.dns.pt unbound[178]: [178:0] info:    0.000000    0.000001 1
May 13 03:49:11 resolv2.grpX.dnssec.lab.dns.pt package-helper[577]: /var/lib/unbound/root.key has content
May 13 03:49:11 resolv2.grpX.dnssec.lab.dns.pt package-helper[577]: success: the anchor is ok
May 13 03:49:11 resolv2.grpX.dnssec.lab.dns.pt unbound[578]: [578:0] notice: init module 0: subnet
May 13 03:49:11 resolv2.grpX.dnssec.lab.dns.pt unbound[578]: [578:0] notice: init module 1: validator
May 13 03:49:11 resolv2.grpX.dnssec.lab.dns.pt unbound[578]: [578:0] notice: init module 2: iterator
May 13 03:49:11 resolv2.grpX.dnssec.lab.dns.pt unbound[578]: [578:0] info: start of service (unbound 1.9.4).
May 13 03:49:11 resolv2.grpX.dnssec.lab.dns.pt systemd[1]: Started Unbound DNS server.
```

### Testar o vosso novo resolver de validação

Executar os seguintes comandos e confirmar se recebem a flag "ad":

1. dig SOA com. @100.100.X.68 
2. dig SOA com. @100.100.X.68 +dnssec
3. dig A www.icann.org @100.100.X.68
4. dig NS icann.org @100.100.X.68
5. dig DNSKEY *grpX*.dnssec.lab.dns.pt @100.100.X.68
6. dig NS *grpX*.dnssec.lab.dns.pt @100.100.X.68 +dnssec
7. dig SOA *grpX*.dnssec.lab.dns.pt @100.100.X.68
8. dig DNSKEY *grpX*.dnssec.lab.dns.pt @100.100.X.130
9. dig DNSKEY *grpX*.dnssec.lab.dns.pt @100.100.X.130 +multi
10. dig SOA *grpX*.dnssec.lab.dns.pt @100.100.X.131 +dnssec +multi

Receberam a flag "ad" para as últimas três consultas dig? Porquê?

Agora tentem o seguinte:

1. dig www.dnssec-failed.org @9.9.9.9
2. dig www.dnssec-failed.org @100.100.X.68 
3. dig www.dnssec-failed.org +dnssec @100.100.X.68
4. dig www.dnssec-failed.org +dnssec +cd @100.100.X.68

O que notam? Porquê?

### Configurar o resolver recursivo do vosso SO para usar o vosso software resolver recursivo de validação local.

Ainda no servidor **grpX-resolv2**, editar o ficheiro de configuração **/etc/resolv.conf** e substituir o que lá estiver apenas pelo seguinte:

```
nameserver 100.100.X.68
```

Guardar e sair. Depois tentem as seguintes consultas:

1. dig SOA com. 
2. dig SOA com. +dnssec
3. dig A www.icann.org
4. dig NS icann.org
5. dig NS *grpX*.dnssec.lab.dns.pt
6. dig NS *grpX*.dnssec.lab.dns.pt +dnssec
7. dig SOA *grpX*.dnssec.lab.dns.pt
8. dig SOA *grpX*.dnssec.lab.dns.pt +dnssec +multi
9. dig DNSKEY *grpX*.dnssec.lab.dns.pt
10. dig DNSKEY *grpX*.dnssec.lab.dns.pt +multi

Obtiveram a flag "ad" em todos os casos?



## Desativar temporariamente a validação DNSSEC para domínios que estejam a falhar a validação

Uma zona pode ficar quebrada devido a problemas com a sua configuração DNSSEC. Nesse caso, resolvers de validação podem retornar respostas DNS com estado bogus, servfail, etc. Os utilizadores por trás desse resolver recursivo serão impactados para esses domínios. Embora seja geralmente responsabilidade do administrador do domínio corrigir o problema, o administrador do resolver recursivo pode tomar medidas para temporariamente desativar a validação DNSSEC para tal domínio que está quebrado ao nível da validação.

### Resolver recursivo BIND

Editar o ficheiro de configuração **/etc/bind/named.conf.options** e adicionar o seguinte:

``` 
validate-except
   {
       "SOME_DOMAIN_NAME_TO_EXCLUDE";
       "ANOTHER.DOMAIN.g/ccTLD";
   };
```

Aplicar a configuração e testar.

### Resolver recursivo Unbound

Para temporariamente parar uma cadeia de confiança quebrada, adicionar o seguinte ao ficheiro **unbound.conf**:

```
server:
    domain-insecure: "A_DOMAIN_TO_TEMPORARILY_DISABLE.g/ccTLD"
```

Reiniciar o serviço e testá-lo usando dig!

Podem usar _dig_ para confirmar se a validação está desativada apenas para o vosso domínio excluído.
