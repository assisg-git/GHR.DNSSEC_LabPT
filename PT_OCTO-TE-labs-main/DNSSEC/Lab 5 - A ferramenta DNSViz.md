
<div style="display: flex; gap: 700px;">
<img src="https://github.com/assisg-git/GHR.DNSSEC_LabPT/blob/main/PT_OCTO-TE-labs-main/_img/LOGO_ICANN.jpg" alt="ICANN" style="width: 40%;"/>
<img src="https://github.com/assisg-git/GHR.DNSSEC_LabPT/blob/main/PT_OCTO-TE-labs-main/_img/LOGO_PT.png" alt=".PT"  style="width: 40%;"/>
</div>

------
```
Criado por: Yazid AKANHO
Modificado por: - Assis Guerreiro .PT

Versão atual: 2025120300 
Versão anterior: 2024040100
```
------
### Pré-requisitos

Assumimos que:

1. a vossa zona grpX.dnssec.lab.dns.pt. está assinada com DNSSEC
2. Um RRSet DS foi carregado para o servidor autoritativo do domínio da turma, e está publicado na zona do workshop "dnssec.lab.dns.pt".
3. Os vossos resolvers recursivos resolv1 e resolv2 estão configurados e a validar DNSSEC (laboratório de validação DNSSEC).
4. O software dos vossos resolvers recursivos (resolv1 e resolv2) está configurado para usar-se a si próprio como resolvers (em /etc/resolv.conf)

> [!WARNING]
>
> Se alguma destas coisas não for verdadeira, terminem os laboratórios anteriores antes de começar este. Se precisarem de ajuda para se atualizarem, um dos vossos simpáticos membros da equipa de laboratório do workshop terá todo o gosto em ajudar.



Antes de continuarem, verifiquem o seguinte em qualquer dos vossos dois resolvers recursivos:

```
$ dig grpX.dnssec.lab.dns.pt SOA
```
retorna:

- uma "ANSWER" correta
- flag "AD" presente
- e SERVER deve ser o localhost (::1 ou 100.100.X.67 ou 100.100.X.68).


## Introdução

Neste laboratório, vamos verificar a cadeia de confiança de DNSSEC para o vosso domínio de laboratório usando uma ferramenta chamada dnsviz. Para os vossos domínios em produção, podem usar a sua versão online disponível em [https://dnsviz.net/](https://dnsviz.net/).

## Passos

Ir a qualquer dos vossos resolvers recursivos.

Instalar pacotes necessários do dnsviz

```
root@resolv1:~# apt install python3-dns python3-pygraphviz python3-m2crypto
root@resolv1:~# apt-get install dnsviz
```

Executar dnsviz contra o vosso domínio

```
root@resolv1:~# dnsviz probe grpX.dnssec.lab.dns.pt | dnsviz graph -Tpng -O
```

Informar o vosso formador que irá descarregar o ficheiro e abri-lo num ambiente gráfico.

Alternativamente, podem querer descarregar e abrir este ficheiro no vosso próprio laptop pessoal. Para fazer isso, podem fazer o seguinte.

Carregar o ficheiro para a nuvem

```
root@resolv1:~# curl -F "file=@grpX.dnssec.lab.dns.pt.png" https://file.io
```

O resultado esperado é

```
root@resolv1:~# curl -F "file=@grpX.dnssec.lab.dns.pt.png" https://file.io
{"success":true,"status":200,"id":"--------------------------------",
"key":"XXXXXXXXXXXXXXXXXXX",
"path":"/",
"nodeType":"file",
"name":"grpX.dnssec.lab.dns.pt.png",
"title":null,"description":null,"size":144179,"link":"https://file.io/XXXXXXXXXXXXXXXXXXX",
"private":false,"expires":"2024-09-18T09:05:27.433Z",
"downloads":0,"maxDownloads":1,"autoDelete":true,"planId":0,"screeningStatus":"pending",
"mimeType":"image/png",
"created":"2024-09-04T09:05:27.433Z",
"modified":"2024-09-04T09:05:27.433Z"}
```

Podem agora abrir o URL gerado no vosso navegador para descarregar o ficheiro em file.io e abri-lo na vossa máquina local.
