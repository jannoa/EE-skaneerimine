# EE-skaneerimine ja analüüsimine

Eesmärk on skaneerida kogu EE Internet ja leida lahendus, mis võimaldaks töödelda saadud skaneeringu tulemusi sellisele kujule, et neid andmeid oleks võimalik hoiustada ajalooliselt, ja et tekiks visuaalne arusaam turvanõrkustega seadmetest ja veebiteenustest EE Internetis.

## Hetke lahendus

1. Shodan.io scan EE Internetist
2. JSON kujul skaneeringu tulemus
3. Saadud JSON analüüsimine
4. Programmeerimiskeele jq kasutamine, et luua päringud, mis võimaldavad töötlemata JSONist kättesaada meid huvitavad andmed
5. jq-ga päritud andmed on sellisel kujul, et saab luua esialgseid raporteid .csv (comma separated values) kujul
6. Leida lahendus, mis võimaldaks .csv kujul salvestatud andmed salvestada ajalooliselt ja luua visuaalseid raporteid
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

Algselt vaatasin sellele andmehulgale otsa eesmärgiga, et kuidas sealt välja võtta meile vajalik ja mitte, et leiaks üles spetsiifilisi turvanõrkustega seadmeid vms. Seega algselt üritasin läheneda Pythoniga, et andmeid töödelda aga kiiresti mõistsin, et ei ole mõistlik lähenemine ja siis tutvustati mulle **jq** programmeerimiskeelt. *jq* võimaldab JSON datast lihtsalt pärida *key* väärtuseid. Seega pidin ainult teadma, millises *key-s* on mind huvitav info ja seda pärima. Tundus lihtne aga kui hakata mõtlema, siis kas oskame lihtsalt öelda, millises *key-s* on täpselt selle teenusele või seadmele vastav info, et seda siis pärida. Ei oska.

### 4. Programmeerimiskeele jq kasutamine, et luua päringud, mis võimaldavad töötlemata JSONist kättesaada meid huvitavad andmed

Enne kui aru sain kui raske on tegelikult leida JSONist meid huvitavat sattusin ühe Shodani spetsiifilise skaneeringu tulemuse peale, mis on nüüdseks küllaltki petlikuks muutunud. Ehk kui tuvastatakse IP tagant mõne kriitilisema CVE-ga turvanõrkus, siis kirjutatakse see sellele skaneeringu tulemusele külge.

Nagu ka *sample.json*-is näha on lisatud juurde *CVE-2014-0160*, mis siis vastab Heartbleed turvanõrkusele
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
Olemasolev käsk võimaldab meil siis käia üle kõik ~524MB andmebaasisist leitavad objectid ja kui leiab kuskilt JSON *key*-st stringi "!CVE-2014-0160", siis filtreeritakse sealt object-ist välja IP = .ip_str, ASN = .asn, timestamp= .timestamp, ID ja kirjeldus. Filtreeritud tulemused viiakse .csv kujule ja kirjutatakse tulemus.csv faili. 

Selle tulemusele põhineb ka kogu ülejäänud lahenduse käik.



## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

What things you need to install the software and how to install them

```
Give examples
```

### Installing

A step by step series of examples that tell you have to get a development env running

Say what the step will be

```
Give the example
```

And repeat

```
until finished
```

End with an example of getting some data out of the system or using it for a little demo

## Running the tests

Explain how to run the automated tests for this system

### Break down into end to end tests

Explain what these tests test and why

```
Give an example
```

### And coding style tests

Explain what these tests test and why

```
Give an example
```

## Deployment

Add additional notes about how to deploy this on a live system

## Built With

* [Dropwizard](http://www.dropwizard.io/1.0.2/docs/) - The web framework used
* [Maven](https://maven.apache.org/) - Dependency Management
* [ROME](https://rometools.github.io/rome/) - Used to generate RSS Feeds

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

* **Billie Thompson** - *Initial work* - [PurpleBooth](https://github.com/PurpleBooth)

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

Thanks for the README.md - PurpleBooth/README-Template.md

