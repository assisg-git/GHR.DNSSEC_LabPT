
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


Durante este exercício vamos apenas aceder ao seguinte equipamento:
	* **grpX-soa**: servidores autoritativos escondidos (primário)
	
> [!warning] Aviso Importante!
> Substituam o _**X**_ pelo número do vosso grupo nos endereços IP, nomes de servidores e em qualquer outro local onde seja necessário. 


## Configurar o servidor autoritativo primário (SOA)

Como lembrete, o vosso domínio é "grpX.dnssec.lab.dns.pt", e a zona pai é "dnssec.lab.dns.pt".

> [!IMPORTANT]
>
> O vosso pai está à espera do nome do ficheiro DS no seguinte formato: **DS_YOUR-KSK-key-tag.grpX**. Lembrete para substituir 'X' pelo número do vosso grupo. 
Por exemplo, se a vossa tag KSK é 2347 e o número do vosso grupo é 12, o nome do ficheiro DS deve ser DS_2347.grp12.



Comecem por criar um diretório para armazenar os vossos ficheiros DS para o vosso domínio e certifiquem-se que BIND tem acesso de "escrita":

```
sudo su -
mkdir -p /var/lib/bind/zones/ds
chown -R bind:bind /var/lib/bind/zones/ds
```



### Gerar o vosso registo DS

Executar o seguinte comando para identificar o key id (key tag) da chave KSK, obter o registo DS e guardá-lo no ficheiro necessário:

```
dig @localhost grp8.dnssec.lab.dns.pt DNSKEY +multi


dig @localhost grpX.dnssec.lab.dns.pt DNSKEY | dnssec-dsfromkey -f - grpX.dnssec.lab.dns.pt > /var/lib/bind/zones/ds/DS_YOUR-KSK-key-tag.grpX

```

Verificar o conteúdo do ficheiro gerado:

```
cat /var/lib/bind/zones/ds/DS_YOUR-KSK-key-tag.grpX
```

Que deve conter algo semelhante à seguinte linha:

```
grpX.dnssec.lab.dns.pt. IN DS YOUR-KSK-key-tag 8 2 018A86C0139BA5500AC87A5BAD8FB5D8D4F9672C319B34DB5A7F3BC10A424D6E
```



### Enviar o DS para o vosso pai

Ir à página web do vosso grupo.


Nesta fase, estão a dizer ao mundo que estão assinados e qualquer validador (resolver recursivo com **validação DNSSEC ativada**) validará respostas DNS do vosso domínio. Vamos agora configurar um resolver recursivo de validação DNSSEC no próximo laboratório para realizar essa validação DNSSEC.

Até lá!

