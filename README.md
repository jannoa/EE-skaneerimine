# EE-skaneerimine ja analüüsimine

Eesmärk on skaneerida kogu EE Internet ja leida lahendus, mis võimaldaks töödelda saadud skaneeringu tulemusi sellisele kujule, et neid andmeid oleks võimalik hoiustada ajalooliselt, ja et tekiks visuaalne arusaam turvanõrkustega seadmetest ja veebiteenustest EE Internetis.

## Hetke lahenduse ülevaade

1. Shodan.io scan EE Internetist
2. JSON kujul skaneeringu tulemus
3. Saadud JSON analüüsimine
4. Programmeerimiskeele *jq* kasutamine
5. Esmane raport
6. Lahenduse leidmine andmete visualiseerimiseks
7. Logstash <- Elasticsearch -> Kibana lahendus
8. Logstash struktueerib .csv andmed sellisele kujule, et on töödeldavad Elastisearch-i poolt
9. Elasticsearch võimaldab andmeid hoiustada, arhiveerida ja töödelda
10. Kibanaga on võimalik andmeid pärida Elasticsearch-ist ning luua visuaalseid raporteid ja analüüse

### 1. Shodan.io

Idee oli võtta Shodan.io kasutusele kuna tegemist kõige populaarsema skaneeringu tööriistaga, mis leiab laialdast kasutust ja mille tulemustele tihti põhinetakse. Lisaks kui luua tööriist , mis kasutab just Shodani väljundit ehk on ka reaalselt teisi huvilisi, kes tahaksid loodud lahendust kasutada.

Hetkel on murekohtadeks loomulikult see, et saada Shodanist JSON väljundit, peab maksma ja saada kogu Eesti skaneeringut, läheb see 42 eurot. Kas ka mõistlik - see on vaieldav ja tasuks võibolla mõelda teiste teenuste peale nagu [Censys](https://censys.io/data) või [Rapid7 Sonar](https://github.com/rapid7/sonar/wiki). Organisatsioonidel on ka võimalus teha leping, millega tekitatakse reaalajas andmevoog.

### 2. JSON kujul skaneeringu tulemus

Olen hetkel töödelnud ühte 2017. aasta, augusti Shodan-i JSON andmehulka, millest ühe nested-objecti näidis [sample.json](https://github.com/jannoa/EE-skaneerimine/blob/master/sample.json). Terve andmehulk on ~524MB.

### 3. Saadud JSON analüüsimine

Algselt vaatasin sellele andmehulgale otsa eesmärgiga, et kuidas sealt välja võtta meile vajalik ja mitte, et leiaks üles spetsiifilisi turvanõrkustega seadmeid vms. Seega algselt üritasin läheneda Pythoniga, et andmeid töödelda aga kiiresti mõistsin, et ei ole mõistlik lähenemine ja siis tutvustati mulle **jq** programmeerimiskeelt. *jq* võimaldab JSON datast lihtsalt pärida *key* väärtuseid. Seega pidin ainult teadma, millises *key-s* on mind huvitav info ja seda pärima. Tundus lihtne aga kui hakata mõtlema, siis kas oskame lihtsalt öelda, millises *key-s* on täpselt selle teenusele või seadmele vastav info, et seda siis pärida - ei oska.

### 4. Programmeerimiskeele jq kasutamine

Enne kui aru sain kui raske on tegelikult leida JSONist meid huvitavat sattusin ühe Shodani spetsiifilise skaneeringu tulemuse peale, mis on nüüdseks küllaltki petlikuks muutunud. Ehk kui tuvastatakse IP tagant mõne kriitilisema CVE-ga turvanõrkus, siis kirjutatakse see sellele skaneeringu tulemusele külge.

Nagu ka *sample.json*-is näha, on lisatud juurde *CVE-2014-0160*, mis siis vastab Heartbleed turvanõrkusele
```
"opts": {
"vulns": ["!CVE-2014-0160"],
"heartbleed": "2017/08/29 09:57:30 196.196.216.13:2087 - SAFE\n"
},
```
Mõtlesin, et see on hea kohta kust siis alustada *jq* keele kasutamist ja kuidas koostada erinevaid päringuid, et leida nt kõik hostid andemhulgast, kes on siis haavatavad Heartbleed turvanõrkusele.

Hetke lahendus on:
```
jq -r '.. | select(.isp?)| select(..=="!CVE-2014-0160") | ['.ip_str', '.asn', '.timestamp', "1", "Heartbleed"] | @csv' >> tulemus.csv shodan_data.json
```
Olemasolev käsk võimaldab meil siis käia üle kõik ~524MB andmebaasisist leitavad objectid ja kui leiab kuskilt JSON *key*-st stringi "!CVE-2014-0160", siis filtreeritakse sealt object-ist välja IP = .ip_str, ASN = .asn, timestamp= .timestamp, ID ja kirjeldus. Filtreeritud tulemused viiakse .csv kujule ja kirjutatakse *tulemus.csv* faili. 

Sellele tulemusele põhineb ka kogu ülejäänud lahenduse näidis.

**Miks aga selline päring ei ole realistlik lähenemine**

Väljatoodud *jq* päring on tehtud eeldusel, et teame, et andmehulgas kuskil on täpselt selline string, mida otsime ehk *"!CVE-2014-0160"*. 

Mida aga teha siis kui me ei tea, kas väärtus mida otsime on kuskil andmehulgas või mitte? Eks siis anname kasutajale teada, et ei leitud tulemusi?

Aga siis kui see väärtus, mida otsime on andmehulgas olemas aga mitte küll täpselt sellisel kujul, mida me otsime? Eks peab arendama *jq* päringu, mis on võimeline *regex*-i abiga erinevaid võimalusi tagastama? Olen üritanud sellist lahendust luua aga siiani pole õnnestunud... 

Ning kuidas sa üldse tead, mida sa otsid? Tihti turvanõrkustega seadmeid leiab ainult üles nende püsivara või väga spetsiifilise tarkvara versiooni järgi või lausa mingi string veebibanneris. Aga teada millises JSON *key*-s  just see väärtus on mida sa otsid  - see on jällegi peaaegu võimatu kui just ei ole seda oma silmaga näinud, nagu ma sattusin peale:
```
"vulns": ["!CVE-2014-0160"],
```
Mõtlesin, et äkki siis lahendus selline, et kõigepealt kasutad Shodan.io enda otsingumootorit, et teha kindlaks kas Shodani skaneeringud on leidnud selliseid seadmeid või teenuseid üldse. Aga ka selleks on vaja algset arusaama mida otsid ehk kas oled leidnud CVE-st või mõnest muus turvanõrkuse teavitusest piisavalt infot, et hakkata otsinguid tegema. Kui lõpuks leiad õige väärtuse, millega saad Shodan.io-st tulemused, siis saab hakata neid vaatama, et kas on viiteid, kuidas see väärtus võiks JSON-is välja näha ja millises *key* väärtuses olla võiks. Selline tööprotsess ei tundu väga mõistlik ja kui efektiivne see oleks, ei oska öelda - hetkel ei ole omale uut Shodani dataseti ostnud ja ei saa võrrelda oma *jq* tulemusi siis Shodan-i otsingumootori omaga.

Hetkel siis kõige suurem **murekoht** ongi - leida milline tööprotsess oleks kõige efektiivsem ja reaalselt ka töötaks ja kas see on võimalik üldse Shodaniga töötades?

### 5. Esmane raport

Kasutades *jq* päringut
```
jq -r '.. | select(.isp?)| select(..=="!CVE-2014-0160") | ['.ip_str', '.asn', '.timestamp', "1", "Heartbleed"] | @csv' >> tulemus.csv shodan_data.json
```
Saame sellise [tulemuse](https://github.com/jannoa/EE-skaneerimine/blob/master/tulemus.csv)

Esmane tulemus on täiesti aktsepteeritav annab - ASN-ide kaupa ülevaate, kui palju on Eestis Heartbleed turvanõrkustega teenuseid. Andmed sisaldavad ka täpseid timestampe, mis võimaldavad lõpuks luua ajaloolist ülevaadet. Lisaks on andmetel juures enda poolt lisatud identifikaator *ID* ja lühikirjeldus *DESC*, mis tulevad kasuks back-end funktsionaalsuses. 

### 6. Lahenduse leidmine andmete visualiseerimiseks

Andmed olemas, üritasin leida lihtsat lahendust, et joonistada mõned graafikud, et saaks siis anda head kiiret ülevaadet kogutud andmetest. Kiiresti mõistsin, et tavapärased tekstitöötlus tarkvarad ja muud veebiteenused kas ei jaksa jõudluse koha pealt või ei paku piisavalt funktsionaalsust, et nende andmete midagi reaalselt ka näidata või visualiseerida.

Seega hakkasin otsima monitooringu/analüüsi tööriistu nagu Kibana ja Grafana. Ning mitmetel põhjustel olen hetkel otsustanud Kibana kasuks. 

### 7. Logstash <- Elasticsearch -> Kibana lahendus

Kibanast üksinda loomulikult ei piisanud. Kibana on väga hea tööriist erinevate graafikute joonistamiseks ja andmete analüüsimiseks aga oma info saab ta Elasticsearch-ist.

Selleks aga, et meie poolt töödeldud andmed jõuaksid Elastisearch-i peab kasutama Logstash mootorit, mis struktureerib *tulemused.csv* failis Elasticsearch-i jaoks.

Tänu sellele 


















