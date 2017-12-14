# EE-skaneerimine ja analüüsimine

Eesmärk on skaneerida kogu EE Internet ja leida lahendus, mis võimaldaks töödelda saadud skaneeringu tulemusi sellisele kujule, et neid andmeid oleks võimalik hoiustada ajalooliselt, ja et tekiks visuaalne arusaam turvanõrkustega seadmetest ja veebiteenustest EE Internetis.

## Hetke lahendus

1. Shodan.io scan EE Internetist
2. JSON kujul skaneeringu tulemus - raskesti loetav, suuremahuline andmehulk
3. Saadud JSON analüüsimine - kuidas andmed struktureeritud on ja tööprotsessi loomine, et leiaks andmetest meid huvitavad faktid
4. Programmeerimiskeele jq kasutamine, et luua päringud, mis võimaldavad töötlemata JSONist kättesaada meid huvitavad andmed
5. jq-ga päritud andmed on sellisel kujul, et saab luua esialgseid raporteid .csv (comma separated values) kujul
6. Leida lahendus, mis võimaldaks .csv kujul salvestatud andmed salvestada ajalooliselt ja luua visuaalseid raporteid
7. Logstash <- Elasticsearch -> Kibana lahendus
8. Logstash struktueerib .csv andmed sellisele kujule, et on töödeldavad Elastisearch-i poolt
9. Elasticsearch võimaldab andmeid hoiustada, arhiveerida ja töödelda
10. Kibanaga on võimalik andmeid pärida Elasticsearch-ist ning luua visuaalseid raporteid ja analüüse

### 1. Shodan.io


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

