

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

<div style="display: flex; gap: 700px;">
<img src="https://github.com/assisg-git/GHR.DNSSEC_LabPT/blob/main/PT_OCTO-TE-labs-main/_img/grp_network_map.png" alt="ICANN" style="width: 100%;"/>

</div>


Durante este exercício vamos apenas aceder ao seguinte equipamento:
	* **grpX-soa**: servidores autoritativos escondidos (primário)
	
> [!warning] Aviso Importante!
> Substituam o _**X**_ pelo número do vosso grupo nos endereços IP, nomes de servidores e em qualquer outro local onde seja necessário. 


## Configurar o servidor autoritativo primário (SOA)

Para assinar a zona precisamos primeiro de dois pares de chaves: uma ZSK e uma KSK. Pode ser assinada com um único par de chaves (CSK) mas essa não é uma configuração recomendada. 

Posicionem-se na pasta de configuração BIND e depois façam backup do vosso ficheiro de zona:

```
sudo su - 
cd /var/lib/bind/
cp zones/db.grpX zones/db.grpX.backup
```

Criar um diretório para guardar as vossas chaves DNSSEC

```
mkdir -p /var/lib/bind/keys
cd /var/lib/bind/keys
```

Gerar a chave **ZSK**

```
dnssec-keygen -a RSASHA256 -b 2048 -n ZONE grpX.dnssec.lab.dns.pt
```

Gerar a chave **KSK**

```
dnssec-keygen -f KSK -a RSASHA256 -b 2048 -n ZONE grpX.dnssec.lab.dns.pt
```

Mudar propriedade das pastas zones e keys

```
chown -R bind:bind /var/lib/bind/keys
chown -R bind:bind /var/lib/bind/zones
```



Depois, assinar a zona. Aqui temos duas opções: 
1. Assinar a zona manualmente
2. Dizer ao BIND para assinar. 

Existe até uma terceira opção e mais... Qual usar depende de vós. Como estamos em ambiente de laboratório, porque não testar a assinatura manual de zona, e depois assinatura inline BIND, uma após a outra?



## Assinatura manual de zona

```
cd /var/lib/bind/
dnssec-signzone -S -K keys/ -o grpX.dnssec.lab.dns.pt zones/db.grpX
```

Devem obter um resultado semelhante ao seguinte:

```
Fetching grpX.dnssec.lab.dns.pt/ECDSAP256SHA256/5515 (KSK) from key repository.Fetching grpX.dnssec.lab.dns.pt/ECDSAP256SHA256/47091 (ZSK) from key repository.Verifying the zone using the following algorithms: ECDSAP256SHA256.
Zone fully signed:
Algorithm: ECDSAP256SHA256: KSKs: 1 active, 0 stand-by, 0 revoked
                            ZSKs: 1 active, 0 stand-by, 0 revoked
zones/db.grpX.signed
```


Depois, substituir o ficheiro ***db.grpX*** em **`named.conf.local`** com o ***db.grpX.signed*** e reiniciar o servidor usando rndc reload.

Altere a configuração da zona no ficheiro de configuração ***/etc/bind/named.conf.local*** como abaixo:

```
nano /etc/bind/named.conf.local
```

```
zone "grpX.dnssec.lab.dns.pt" {
		type primary;
		file "/var/lib/bind/zones/db.grpX";   <-- db.grpX.signed
		allow-transfer { any; };
		also-notify {100.100.X.130; 100.100.X.131; };
}; 
```


> [!TIP]
>
> Quando terminar, use o comando ***named-checkconf*** para verificar que a configuração BIND está correta..

```
named-checkconf /etc/bind/named.conf
```

e recarregar a zona "grpX.dnssec.lab.dns.pt".  
```
rndc reload grpX.dnssec.lab.dns.pt
ou 
systemctl restart named
```




#### Usar ferramentas de linha de comando para consultar a zona assinada e verificar se a assinatura está efetiva

Podemos agora usar o utilitário *dig* para confirmar que a zona está assinada e experimentar os novos RRs DNSSEC. 

> [!WARNING] 
> 
>
> lembrem-se que nesta fase, apenas assinaram a zona e ainda não estabeleceram a cadeia de confiança. 
 
**QUESTÃO**: Vão obter a flag "ad"? Porquê?

```
# dig @localhost soa grpX.dnssec.lab.dns.pt. +dnssec 
; <<>> DiG 9.16.1-Ubuntu <<>> @localhost soa grpX.dnssec.lab.dns.pt. +dnssec                                                      
; (2 servers found)                                                               
;; global options: +cmd                                                           
;; Got answer:                                                                  
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9591                         
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1                                                                   
;; OPT PSEUDOSECTION:                                                             
; EDNS: version: 0, flags: do; udp: 4096
; COOKIE: 69a0c61239afd9a201000000609c5df711d4eb3a39f90d89 (good)
;; QUESTION SECTION:
;grpX.dnssec.lab.dns.pt.        IN      SOA

;; ANSWER SECTION:
grpX.dnssec.lab.dns.pt. 30 IN   SOA     grpX.dnssec.lab.dns.pt. dnsadmin.grpX.dnssec.lab.dns.pt.grpX.dnssec.lab.dns.pt. 1 604800 86400 2419200 86400
grpX.dnssec.lab.dns.pt. 30 IN   RRSIG   SOA 8 4 30 20210611215606 20210512215606 41110 grpX.dnssec.lab.dns.pt. RmUbjShh4jX
fw384miz1G1703ObV9WrYQOOJVSbzDNchCsLayuW/UQRR w3X6eTXHOCSVOcG2Bamkbals48LYUA9Y/l2tmuaGxKkeQVT5xcy0wY/rbeaN4NgUG+N13BFodOPQumsBERQ+NUDAw898IfkcwcZ3pZFgIAsXplA1 MY4= 
```

Mais testes:
1. dig DNSKEY *grpX*.dnssec.lab.dns.pt @100.100.X.130
2. dig DNSKEY *grpX*.dnssec.lab.dns.pt @100.100.X.130 +dnssec +multi
3. dig SOA *grpX*.dnssec.lab.dns.pt @100.100.X.130 +dnssec +multi

**QUESTÃO**: obtiveram as respostas? Receberam as assinaturas? Obtiveram a flag "ad"? Porquê?



> [!IMPORTANT]
>
> Quando terminarem a assinatura manual e confirmarem que os vossos nameservers públicos estão a servir a zona assinada, devem:

1. reverter `named.conf.local` para a sua configuração anterior, ou seja, configurar BIND para servir o ficheiro de zona não assinado como antes da configuração de assinatura manual que era: `file "/var/lib/bind/zones/db.grpX"`; 
```
nano /etc/bind/named.conf.local
file "/var/lib/bind/zones/db.grpX.signed"; ---> "/var/lib/bind/zones/db.grpX";
(gravar "Ctrl+o" e sair "Ctrl+x")
```
2. fazer backup do ficheiro de zona assinado (.signed) e eliminar o ficheiro de zona .signed (BIND criará o seu próprio ficheiro de zona assinado no próximo passo)
```
nano /etc/bind/named.conf.local
file "/var/lib/bind/zones/db.grpX.signed"; ---> "/var/lib/bind/zones/db.grpX";
(gravar "Ctrl+o" e sair "Ctrl+x")
cd /var/lib/bind/zones/
mv db.grpX.signed db.grpX.signed.backup
```
3. aumentar o serial no ficheiro de zona não assinado, recarregar BIND e verificar o serial.
```
nano db.grp8
"4         ; Serial" ----> "5         ; Serial"
(gravar "Ctrl+o" e sair "Ctrl+x")
systemctl restart named
dig @localhost soa grp8.dnssec.lab.dns.pt. +dnssec +short
```


## Configurar BIND para assinar a zona

#### Editar ficheiro de configuração
Atualizar a declaração de configuração da zona em `/etc/bind/named.conf.local`,
```
nano /etc/bind/named.conf.local
```
 para ficar como abaixo (adicionar a linha - dnssec-policy "default";):
```
zone "grpX.dnssec.lab.dns.pt" {
	type primary;
	file "/var/lib/bind/zones/db.grpX";
	allow-transfer { any; };
	also-notify {100.100.X.130; 100.100.X.131; };
	dnssec-policy "default";
};
```


Depois, reconfigurar ou reiniciar BIND: usando `rndc reconfig` ou `systemctl restart named`. Verificar sempre o estado após tal operação. 
```
systemctl restart named
systemctl status named
```


Alguns novos ficheiros devem aparecer na pasta */var/lib/bind/zones*.
```
ls -al /var/lib/bind/zones
```
#### Verificar que a vossa zona está assinada
Usamos o comando `rndc dnssec status` para confirmar que a zona está assinada. Devem obter um resultado como:
```
rndc dnssec -status grpX.dnssec.lab.dns.pt

dnssec-policy: default
current time:  Sun Feb 22 22:24:05 2026

key: 25940 (ECDSAP256SHA256), CSK
  published:      yes - since Sun Feb 22 22:20:21 2026
  key signing:    yes - since Sun Feb 22 22:20:21 2026
  zone signing:   yes - since Sun Feb 22 22:20:21 2026

  No rollover scheduled
  - goal:           omnipresent
  - dnskey:         rumoured
  - ds:             hidden
  - zone rrsig:     rumoured
  - key rrsig:      rumoured


```

#### Usar ferramentas de linha de comando para consultar a zona assinada
Podemos agora usar o utilitário *dig* para confirmar que a zona está assinada e experimentar os novos RRs DNSSEC.



> [!WARNING]
>
> lembrem-se que nesta fase, apenas assinaram a zona e ainda não estabeleceram a cadeia de confiança.

**QUESTÃO**: Vão obter a flag "ad"? Porquê?

1. dig DNSKEY *grpX*.dnssec.lab.dns.pt @100.100.X.130
2. dig DNSKEY *grpX*.dnssec.lab.dns.pt @100.100.X.130 +dnssec +multi
3. dig SOA *grpX*.dnssec.lab.dns.pt @100.100.X.130 +dnssec +multi

**QUESTIONS**: obtiveram as respostas? Receberam as assinaturas? Obtiveram a flag "ad"? Porquê?
