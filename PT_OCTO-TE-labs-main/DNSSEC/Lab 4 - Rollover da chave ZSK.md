
<div style="display: flex; gap: 700px;">
<img src="https://github.com/assisg-git/GHR.DNSSEC_LabPT/blob/main/PT_OCTO-TE-labs-main/_img/LOGO_ICANN.jpg" alt="ICANN" style="width: 40%;"/>
<img src="https://github.com/assisg-git/GHR.DNSSEC_LabPT/blob/main/PT_OCTO-TE-labs-main/_img/LOGO_PT.png" alt=".PT"  style="width: 40%;"/>
</div>

------
```
Criado por: Yazid AKANHO
Modificado por: - Assis Guerreiro .PT

Versão atual: 2025120300 
Versão anterior: 2024020800
```
------

### ### Pré-requisitos

Assumimos que:

1. os vossos servidores autoritativos ns1.grpX.dnssec.lab.dns.pt. e ns2.grpX.dnssec.lab.dns.pt. estão a servir o vosso domínio.
2. A vossa zona está assinada usando BIND9, de um laboratório anterior.
3. Estão a usar dois pares de chaves, uma KSK e uma ZSK.
4. Um RRSet DS foi carregado para o servidor autoritativo do domínio da turma, e está publicado na zona workshop <**lab_domain**>.



> [!WARNING]
>
> Se alguma destas coisas não for verdadeira, terminem os laboratórios anteriores antes de começar este. Se precisarem de ajuda para se atualizarem, um dos vossos simpáticos membros da equipa de laboratório do workshop terá todo o gosto em ajudar.



Antes de continuarem, verifiquem que:

```
$ dig grpX.dnssec.lab.dns.pt dnskey +dnssec +multiline
```
vos mostra uma ZSK e uma KSK. Lembrem-se que a KSK tem flags 257.

```
$ dig grpX.dnssec.lab.dns.pt SOA +dnssec +multiline
```
vos dá o registo SOA para o vosso domínio, e que está assinado.


## Introdução


Neste laboratório, vamos realizar uma rotação manual de chave da ZSK. Dado que não estamos a mudar a KSK, não precisamos de gerar um novo registo DS para a zona pai (isto será feito no caso de uma rotação KSK). A nossa abordagem será:

- Definir alguns temporizadores na ZSK existente para especificar quando esta chave ficará inativa (deixará de ser usada!) e se retirará da zona.
- Criar uma nova ZSK além da existente (uma ZSK "**sucessora**").
- Publicar a nova ZSK no RRSet DNSKEY (para que contenha duas ZSKs).
- Assinar a zona com ambas as ZSKs antiga e nova (neste laboratório, estamos a usar a metodologia de rotação ZSK padrão que é pré-publicação. Assinatura dupla e RRSIG duplo são outras metodologias para rotação ZSK).
- Quando for seguro fazê-lo, parar de assinar com a ZSK antiga.
- Quando for seguro fazê-lo, remover a ZSK antiga.

BIND9 tratará da maioria destes passos, mas ainda terão de gerar uma nova ZSK e fornecer instruções de temporização para a transição.

## Rollover da chave ZSK

### Definir alguns temporizadores na ZSK atual (a ZSK "a retirar")

1. Encontrar a vossa ZSK atual na pasta keys. A ZSK deve ter o valor 256. É essa que queremos. Anotem o seu número e vejam o conteúdo do ficheiro. No nosso exemplo, esta é a ZSK. 

   > [!CAUTION]
   >
   > Lembrem-se que o nome do vosso ficheiro será diferente! Não copiem simplesmente Kmytld.+008+26734 mais tarde no laboratório!

   ```
   root@soa:# cat /var/lib/bind/keys/KgrpX.dnssec.lab.dns.pt.+008+26734.key 
   ; This is a zone-signing key, keyid 26734, for grpX.dnssec.lab.dns.pt.
   ; Created: 20221103220010 (Thu Nov  3 22:00:10 2022)
   ; Publish: 20221103220010 (Thu Nov  3 22:00:10 2022)
   ; Activate: 20221103220010 (Thu Nov  3 22:00:10 2022)
   grpX.dnssec.lab.dns.pt. IN DNSKEY 256 3 8 AwEAAadehqG2E23DsA4MnHcaeTH/bKTHlLftvUKR9i8lVbvWNTydacdQ MsZJPTTFZXHeXFdSmxAxImc/FEGNnk9VRr3FfzfJKbc+s6r17PLWn1bO sUxawKZogOvISPytMcWnhbj8Trs8KOoAekB1PRaiPGsCP/nj68ufvrzl x2AcfDJAWPynNDjgHxeFygifVlM6iYuzmPlpcMAY5LCIS/B1MrfashJh wtj0dldgqJSp6yZHaP8vcrMa6+s5McQcqRpyoR2rpNpl6PiOUBtjE0Ho nwg1XYzSaBAbhLdmQhC4MWL/aNiXp1ybwXSVb8uZqL5k26QlKRNH2eB8 
   YRRtq+B9rIs=
   root@soa:/var/lib/bind/keys# 
   ```

   

2. Usar o comando `dnssec-settime`, para definir um tempo "_inativo_" e um tempo "_eliminar_" para esta chave. Estes parâmetros são necessários pelo BIND9 para gerir a rotação ZSK. No exemplo abaixo, usamos "**+10mi**" para significar "dez minutos a partir de agora" e "**+30mi**" para significar "trinta minutos a partir de agora". A mudança nos temporizadores será registada no próprio ficheiro de chave. No mundo real provavelmente usariam temporizadores muito mais longos.

```
root@soa:/var/lib/bind/keys# sudo dnssec-settime -I +10mi -D +30mi KgrpX.dnssec.lab.dns.pt.+008+26734
./KgrpX.dnssec.lab.dns.pt.+008+26734.key
./KgrpX.dnssec.lab.dns.pt.+008+26734.private
root@soa:/var/lib/bind/keys#
```

Verão duas novas linhas ("Inactive" e "Delete") que aparecem no ficheiro de chave.

```
root@soa:/var/lib/bind/keys# cat KgrpX.dnssec.lab.dns.pt.+008+26734.key 
; This is a zone-signing key, keyid 26734, for grpX.dnssec.lab.dns.pt.
; Created: 20221103220010 (Thu Nov  3 22:00:10 2022)
; Publish: 20221103220010 (Thu Nov  3 22:00:10 2022)
; Activate: 20221103220010 (Thu Nov  3 22:00:10 2022)
; Inactive: 20221104163455 (Fri Nov  4 16:34:55 2022)
; Delete: 20221104165455 (Fri Nov  4 16:54:55 2022)
grpX.dnssec.lab.dns.pt. IN DNSKEY 256 3 8 AwEAAadehqG2E23DsA4MnHcaeTH/bKTHlLftvUKR9i8lVbvWNTydacdQ MsZJPTTFZXHeXFdSmxAxImc/FEGNnk9VRr3FfzfJKbc+s6r17PLWn1bO sUxawKZogOvISPytMcWnhbj8Trs8KOoAekB1PRaiPGsCP/nj68ufvrzl x2AcfDJAWPynNDjgHxeFygifVlM6iYuzmPlpcMAY5LCIS/B1MrfashJh wtj0dldgqJSp6yZHaP8vcrMa6+s5McQcqRpyoR2rpNpl6PiOUBtjE0Ho nwg1XYzSaBAbhLdmQhC4MWL/aNiXp1ybwXSVb8uZqL5k26QlKRNH2eB8 YRRtq+B9rIs=
root@soa:/var/lib/bind/keys# 
```



### Criar uma nova chave ZSK (uma ZSK "sucessora")

Não precisamos de especificar o conjunto completo de parâmetros (nome do algoritmo, tamanho da chave, etc.) quando geramos uma ZSK de substituição porque vamos dizer ao comando dnssec-signzone que estamos a criar uma sucessora da antiga ZSK, e o software certificar-se-á que a nova chave que gera corresponde.

Usar a opção **-i** para especificar um intervalo de pré-publicação muito mais curto que o normal, para ser compatível com os temporizadores muito curtos usados no anterior.

```
root@soa:/var/lib/bind/keys# dnssec-keygen -S KgrpX.dnssec.lab.dns.pt.+008+26734 -i5mi
Generating key pair............+++++ ..............................+++++ 
KgrpX.dnssec.lab.dns.pt.+008+12969
root@soa:/var/lib/bind/keys# 
root@soa:/var/lib/bind/keys# chown -R bind:bind /var/lib/bind/keys
root@soa:/var/lib/bind/keys# rndc loadkeys grpX.dnssec.lab.dns.pt
root@soa:/var/lib/bind/keys#
```

Depois verificar os logs e observar o DNS!

Aqui resumimos a série de eventos.

```
root@soa:/var/lib/bind/keys# grep named /var/log/syslog
Nov  4 15:15:51 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): next key event: 04-Nov-2022 16:15:51.577
Nov  4 16:15:51 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): reconfiguring zone keys
Nov  4 16:15:51 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): next key event: 04-Nov-2022 17:15:51.580
Nov  4 16:26:23 soa named[1186]: received control channel command 'loadkeys grpX.dnssec.lab.dns.pt'
Nov  4 16:26:23 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): reconfiguring zone keys
Nov  4 16:26:23 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): next key event: 04-Nov-2022 16:29:55.905
Nov  4 16:29:55 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): reconfiguring zone keys
Nov  4 16:29:55 soa named[1186]: Fetching grpX.dnssec.lab.dns.pt/RSASHA256/12969 (ZSK) from key repository.
Nov  4 16:29:55 soa named[1186]: DNSKEY grpX.dnssec.lab.dns.pt/RSASHA256/12969 (ZSK) is now published
Nov  4 16:29:55 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): next key event: 04-Nov-2022 16:34:55.906
Nov  4 16:29:55 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): sending notifies (serial 5)
Nov  4 16:29:55 soa named[1186]: client @0x7fed1c04b930 100.100.1.130#54173 (grpX.dnssec.lab.dns.pt): transfer of 'grpX.dnssec.lab.dns.pt/IN': IXFR started (serial 4 -> 5)
Nov  4 16:29:55 soa named[1186]: client @0x7fed1c04b930 100.100.1.130#54173 (grpX.dnssec.lab.dns.pt): transfer of 'grpX.dnssec.lab.dns.pt/IN': IXFR ended: 1 messages, 11 records, 2343 bytes, 0.001 secs (2343000 bytes/sec)
Nov  4 16:29:56 soa named[1186]: client @0x7fed1c0af4c0 100.100.1.131#41174 (grpX.dnssec.lab.dns.pt): transfer of 'grpX.dnssec.lab.dns.pt/IN': AXFR started (serial 5)
Nov  4 16:29:56 soa named[1186]: client @0x7fed1c0af4c0 100.100.1.131#41174 (grpX.dnssec.lab.dns.pt): transfer of 'grpX.dnssec.lab.dns.pt/IN': AXFR ended: 1 messages, 28 records, 5030 bytes, 0.001 secs (5030000 bytes/sec)
Nov  4 16:34:55 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): reconfiguring zone keys
Nov  4 16:34:55 soa named[1186]: DNSKEY grpX.dnssec.lab.dns.pt/RSASHA256/12969 (ZSK) is now active
Nov  4 16:34:55 soa named[1186]: DNSKEY grpX.dnssec.lab.dns.pt/RSASHA256/26734 (ZSK) is now inactive
Nov  4 16:34:55 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): next key event: 04-Nov-2022 16:54:55.907
Nov  4 16:34:55 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): sending notifies (serial 6)
Nov  4 16:34:55 soa named[1186]: client @0x7fed2c05dc00 100.100.1.130#44965 (grpX.dnssec.lab.dns.pt): transfer of 'grpX.dnssec.lab.dns.pt/IN': IXFR started (serial 5 -> 6)
Nov  4 16:34:55 soa named[1186]: client @0x7fed2c05dc00 100.100.1.130#44965 (grpX.dnssec.lab.dns.pt): transfer of 'grpX.dnssec.lab.dns.pt/IN': IXFR ended: 1 messages, 10 records, 2067 bytes, 0.001 secs (2067000 bytes/sec)
Nov  4 16:34:56 soa named[1186]: client @0x7fed1c049bc0 100.100.1.131#44942 (grpX.dnssec.lab.dns.pt): transfer of 'grpX.dnssec.lab.dns.pt/IN': AXFR started (serial 6)
Nov  4 16:34:56 soa named[1186]: client @0x7fed1c049bc0 100.100.1.131#44942 (grpX.dnssec.lab.dns.pt): transfer of 'grpX.dnssec.lab.dns.pt/IN': AXFR ended: 1 messages, 28 records, 5030 bytes, 0.001 secs (5030000 bytes/sec)
Nov  4 16:54:55 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): reconfiguring zone keys
Nov  4 16:54:55 soa named[1186]: Removing expired key 26734/RSASHA256 from DNSKEY RRset.
Nov  4 16:54:55 soa named[1186]: DNSKEY grpX.dnssec.lab.dns.pt/RSASHA256/26734 (ZSK) is now deleted
Nov  4 16:54:55 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): next key event: 04-Nov-2022 17:54:55.908
Nov  4 16:54:55 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): sending notifies (serial 7)
Nov  4 16:54:55 soa named[1186]: client @0x7fed1c0a7660 100.100.1.130#42265 (grpX.dnssec.lab.dns.pt): transfer of 'grpX.dnssec.lab.dns.pt/IN': IXFR started (serial 6 -> 8)
Nov  4 16:54:55 soa named[1186]: client @0x7fed1c0a7660 100.100.1.130#42265 (grpX.dnssec.lab.dns.pt): transfer of 'grpX.dnssec.lab.dns.pt/IN': IXFR ended: 1 messages, 38 records, 9338 bytes, 0.001 secs (9338000 bytes/sec)
Nov  4 16:54:56 soa named[1186]: client @0x7fed2405ad30 100.100.1.131#60700 (grpX.dnssec.lab.dns.pt): transfer of 'grpX.dnssec.lab.dns.pt/IN': AXFR started (serial 8)
Nov  4 16:54:56 soa named[1186]: client @0x7fed2405ad30 100.100.1.131#60700 (grpX.dnssec.lab.dns.pt): transfer of 'grpX.dnssec.lab.dns.pt/IN': AXFR ended: 1 messages, 26 records, 4737 bytes, 0.001 secs (4737000 bytes/sec)
Nov  4 16:55:00 soa named[1186]: zone grpX.dnssec.lab.dns.pt/IN (signed): sending notifies (serial 8)
```


Pelos logs, podemos ver que vários eventos aconteceram:
 
- Às 16:29:55, a nova ZSK foi obtida e publicada no DNSKEY da zona.
- Isto gerou um novo ficheiro de zona para ser transferido para os secundários `IXFR started (serial 4 -> 5)`. Se consultarem a zona pelas suas chaves, notarão que existem duas ZSKs lá. Mas apenas uma está ativa.
- Às 16:34:55, as chaves da zona foram reconfiguradas, a chave 12969 ficou ativa enquanto 26734 ficou inativa. Isto também criou uma atualização de zona e transferência de zona entre o primário e os secundários. Neste passo, ainda existem duas ZSK no DNSKEY mas agora mudaram de papéis.
- Às 16:54:55, a chave 26734/RSASHA256 foi eliminada (removida) da zona.
- E novamente, um novo ficheiro de zona ficou disponível para transferir para cada NS secundário. Nesta fase, apenas a ZSK ativa permanece na zona.


> [!IMPORTANT]
>
> Na vida real, devem antecipar todas estas mudanças e definir tempo suficiente entre eventos. Isto permitir-vos-á reagir adequadamente em caso de qualquer comportamento inesperado.
>
> É também crítico ter um sistema de monitorização adequado e rotinas de manutenção no local para observar estes eventos em detalhe e tomar as ações necessárias quando e onde necessário.
