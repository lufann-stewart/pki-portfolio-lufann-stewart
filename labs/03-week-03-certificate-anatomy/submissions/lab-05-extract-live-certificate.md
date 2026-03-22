
# Lab 05 — Extract a Certificate from a Live Website

## Overview
Briefly describe what this lab was about in your own words.
  >To extract and inspect a certificate from a live website.

What PKI concept were you investigating?
  >Certificate Chain.

---

## Environment
- OS: Windows
- Terminal used (Mac Terminal / Git Bash / WSL): Command Prompt (cmd.exe)
- OpenSSL version (`openssl version`): OpenSSL 3.6.1 
- Website used: santander.com

---

## Certificate Fields

| Field                    | Value from your output |
|--------------------------|------------------------|
| Subject                  |  CN=imperva.com                       |
| Issuer                   |  C=BE, O=GlobalSign nv-sa, CN=GlobalSign Atlas R3 DV TLS CA 2026 Q1                      |
| Not Before               |  Feb  5 14:14:39 2026 GMT                      |
| Not After                |  Aug  4 14:14:39 2026 GMT                      |
| Public Key Algorithm     |  rsaEncryption                      |
| Subject Alternative Name |   DNS:aquanima.com, DNS:santandere-derivatives.es, DNS:xn--autobrse-r4a.de, DNS:*.privatelease.santander.nl, DNS:teamsantander.se, DNS:*.preprod.santanderconsumer.se, DNS:emdia.com.br, DNS:*.santanderconsumer.es, DNS:santander.no, DNS:*.hyundaifinance.de, DNS:*.mihogar.com.uy, DNS:*.s3caceis.com.co, DNS:*.santanderibc.com, DNS:*.stg.santander.no, DNS:*.santander.be, DNS:*.santanderconsumerservices.pt, DNS:*.santanderauto.systems, DNS:*.suressedirektbank.de, DNS:carmine.pt, DNS:*.bancosantander.es, DNS:*.collateral.api.mortgages.gamma.tlzproject.com, DNS:*.eu01.solpheosuite.com, DNS:*.wrvc.co.uk, DNS:*.carmine.pt, DNS:*.universia.net, DNS:www.sannetman.com, DNS:*.e.santanderopenacademy.com, DNS:*.zurichsantanderseguros.cl, DNS:*.zurichsantander.cl, DNS:mihogar.com.uy, DNS:santandere-derivatives.com, DNS:*.acp.santanderconsumerbank.nl, DNS:*.santanderconsumer.dk, DNS:*.santanderconsumerbank.be, DNS:paycar.es, DNS:santanderopenacademy.com, DNS:*.santanderdigitalservices.com, DNS:*.solution4fleet.com.br, DNS:*.dv.euemdia.com.br, DNS:*.dev.santander.no, DNS:*.scb.nu, DNS:santanderworkcafe.je, DNS:gruposantander.com, DNS:santander.com, DNS:*.auth-dev.santanderauto.systems, DNS:gira.com.br, DNS:returncapital.com.br, DNS:santanderdigitalservices.net, DNS:*.santanderopenacademy.com, DNS:imperva.com, DNS:*.santander.de, DNS:*.santanderdigitalservices.net, DNS:*.santanderconsumer.fi, DNS:*.easydata.pt, DNS:*.slz.santander.co.uk, DNS:*.emdia.com.br, DNS:euemdia.com.br, DNS:santanderdigitalservices.com, DNS:*.meine.contents.santander.de, DNS:santandersmusic.com, DNS:*.dev.returncapital.com.br, DNS:motorpay.es, DNS:*.test.santanderconsumer.com, DNS:*.mycampusdigital.com, DNS:*.stg.santander.se, DNS:*.dev.onetrade.pagonxtpayments.com, DNS:*.santander.com.uy, DNS:*.emdia.vc, DNS:*.eu.pre.gruposantander.com, DNS:*.santander.com.mx, DNS:*.negocieipanema.com.br, DNS:santanderconsumerbank.nl, DNS:*.gruposantander.es, DNS:santandereurosdetunomina.com, DNS:santanderconsumerservices.pt, DNS:*.creditodelacasa.com.uy, DNS:*.santander.com, DNS:*.santander.nl, DNS:bancosantander.com, DNS:*.developer.pagonxtpayments.com, DNS:*.santander.no, DNS:*.stg.santanderconsumer.fi, DNS:santandermarcas.com, DNS:*.santanderconsumerbank.nl, DNS:*.api-dev.santanderauto.systems, DNS:*.e.santanderx.com, DNS:*.easyboss.pt, DNS:*.pagonxt.com, DNS:*.santandermarcas.com, DNS:*.workcafe.im, DNS:easymanager.pt, DNS:negocieipanema.com.br, DNS:*.hyundaicapital.de, DNS:*.pp.santanderconsumer.se, DNS:*.gsnetcloud.com, DNS:*.officebanking.cl, DNS:*.paycar.es, DNS:*.santander-homo.cl, DNS:*.bancosantander.com, DNS:*.workcafe.je, DNS:easyboss.pt, DNS:*.easymanager.pt, DNS:*.soysantander.com.uy, DNS:*.suressedirectbank.pt, DNS:zurichsantander.cl, DNS:*.santanderconsumer.pt, DNS:*.dev.santander.se, DNS:*.dev.globalid.onetrade.pagonxtpayments.com, DNS:*.officebanking-qa.cl, DNS:*.santandere-derivatives.com, DNS:*.pre.fraudcio.gamma.tlzproject.com, DNS:*.euemdia.com.br, DNS:*.santanderglobaltech.com, DNS:*.santanderconsumer.co, DNS:*.pp.santanderconsumer.fi, DNS:*.dev.santanderconsumer.fi, DNS:*.hedinautomotive-financieringen.nl, DNS:*.santandersmusic.com, DNS:*.dev.ipanemacm.com.br, DNS:*.motorpay.es, DNS:*.gruposantander.com, DNS:*.gira.com.br, DNS:*.santandere-derivatives.es, DNS:*.santandereurosdetunomina.com, DNS:workcafe.je, DNS:*.santanderworkcafe.je, DNS:*.santandercib.com, DNS:*.returncapital.com.br, DNS:*.timfin.it, DNS:*.xn--autobrse-r4a.de, DNS:*.aquanima.com, DNS:*.teamsantander.se, DNS:*.pre.santanderconsumer.pt, DNS:*.e.universia.net, DNS:*.dev.negocieipanema.com.br, DNS:solution4fleet.com.br, DNS:*.santanderconsumer.se, DNS:*.avida.se, DNS:*.ipanemacm.com.br, DNS:workcafe.im                     |
| Key Usage                |  critical Digital Signature, Key Encipherment                      |
| Extended Key Usage       |  TLS Web Server Authentication, TLS Web Client Authentication                      |

---

## Observations

1. What organization does this certificate belong to?  
> O=GlobalSign nv-sa

2. Which Certificate Authority issued it?  
> CN=GlobalSign Atlas R3 DV TLS CA 2026 Q1

3. When does it expire?  
> Aug  4 14:14:39 2026 GMT

4. What domains are listed in the SAN field?  
> DNS:aquanima.com, DNS:santandere-derivatives.es, DNS:xn--autobrse-r4a.de, DNS:*.privatelease.santander.nl, DNS:teamsantander.se, DNS:*.preprod.santanderconsumer.se, DNS:emdia.com.br, DNS:*.santanderconsumer.es, DNS:santander.no, DNS:*.hyundaifinance.de, DNS:*.mihogar.com.uy, DNS:*.s3caceis.com.co, DNS:*.santanderibc.com, DNS:*.stg.santander.no, DNS:*.santander.be, DNS:*.santanderconsumerservices.pt, DNS:*.santanderauto.systems, DNS:*.suressedirektbank.de, DNS:carmine.pt, DNS:*.bancosantander.es, DNS:*.collateral.api.mortgages.gamma.tlzproject.com, DNS:*.eu01.solpheosuite.com, DNS:*.wrvc.co.uk, DNS:*.carmine.pt, DNS:*.universia.net, DNS:www.sannetman.com, DNS:*.e.santanderopenacademy.com, DNS:*.zurichsantanderseguros.cl, DNS:*.zurichsantander.cl, DNS:mihogar.com.uy, DNS:santandere-derivatives.com, DNS:*.acp.santanderconsumerbank.nl, DNS:*.santanderconsumer.dk, DNS:*.santanderconsumerbank.be, DNS:paycar.es, DNS:santanderopenacademy.com, DNS:*.santanderdigitalservices.com, DNS:*.solution4fleet.com.br, DNS:*.dv.euemdia.com.br, DNS:*.dev.santander.no, DNS:*.scb.nu, DNS:santanderworkcafe.je, DNS:gruposantander.com, DNS:santander.com, DNS:*.auth-dev.santanderauto.systems, DNS:gira.com.br, DNS:returncapital.com.br, DNS:santanderdigitalservices.net, DNS:*.santanderopenacademy.com, DNS:imperva.com, DNS:*.santander.de, DNS:*.santanderdigitalservices.net, DNS:*.santanderconsumer.fi, DNS:*.easydata.pt, DNS:*.slz.santander.co.uk, DNS:*.emdia.com.br, DNS:euemdia.com.br, DNS:santanderdigitalservices.com, DNS:*.meine.contents.santander.de, DNS:santandersmusic.com, DNS:*.dev.returncapital.com.br, DNS:motorpay.es, DNS:*.test.santanderconsumer.com, DNS:*.mycampusdigital.com, DNS:*.stg.santander.se, DNS:*.dev.onetrade.pagonxtpayments.com, DNS:*.santander.com.uy, DNS:*.emdia.vc, DNS:*.eu.pre.gruposantander.com, DNS:*.santander.com.mx, DNS:*.negocieipanema.com.br, DNS:santanderconsumerbank.nl, DNS:*.gruposantander.es, DNS:santandereurosdetunomina.com, DNS:santanderconsumerservices.pt, DNS:*.creditodelacasa.com.uy, DNS:*.santander.com, DNS:*.santander.nl, DNS:bancosantander.com, DNS:*.developer.pagonxtpayments.com, DNS:*.santander.no, DNS:*.stg.santanderconsumer.fi, DNS:santandermarcas.com, DNS:*.santanderconsumerbank.nl, DNS:*.api-dev.santanderauto.systems, DNS:*.e.santanderx.com, DNS:*.easyboss.pt, DNS:*.pagonxt.com, DNS:*.santandermarcas.com, DNS:*.workcafe.im, DNS:easymanager.pt, DNS:negocieipanema.com.br, DNS:*.hyundaicapital.de, DNS:*.pp.santanderconsumer.se, DNS:*.gsnetcloud.com, DNS:*.officebanking.cl, DNS:*.paycar.es, DNS:*.santander-homo.cl, DNS:*.bancosantander.com, DNS:*.workcafe.je, DNS:easyboss.pt, DNS:*.easymanager.pt, DNS:*.soysantander.com.uy, DNS:*.suressedirectbank.pt, DNS:zurichsantander.cl, DNS:*.santanderconsumer.pt, DNS:*.dev.santander.se, DNS:*.dev.globalid.onetrade.pagonxtpayments.com, DNS:*.officebanking-qa.cl, DNS:*.santandere-derivatives.com, DNS:*.pre.fraudcio.gamma.tlzproject.com, DNS:*.euemdia.com.br, DNS:*.santanderglobaltech.com, DNS:*.santanderconsumer.co, DNS:*.pp.santanderconsumer.fi, DNS:*.dev.santanderconsumer.fi, DNS:*.hedinautomotive-financieringen.nl, DNS:*.santandersmusic.com, DNS:*.dev.ipanemacm.com.br, DNS:*.motorpay.es, DNS:*.gruposantander.com, DNS:*.gira.com.br, DNS:*.santandere-derivatives.es, DNS:*.santandereurosdetunomina.com, DNS:workcafe.je, DNS:*.santanderworkcafe.je, DNS:*.santandercib.com, DNS:*.returncapital.com.br, DNS:*.timfin.it, DNS:*.xn--autobrse-r4a.de, DNS:*.aquanima.com, DNS:*.teamsantander.se, DNS:*.pre.santanderconsumer.pt, DNS:*.e.universia.net, DNS:*.dev.negocieipanema.com.br, DNS:solution4fleet.com.br, DNS:*.santanderconsumer.se, DNS:*.avida.se, DNS:*.ipanemacm.com.br, DNS:workcafe.im

5. What is this certificate authorized to be used for?  
> TLS Web Server Authentication, TLS Web Client Authentication
