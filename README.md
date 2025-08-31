[![FIWARE Banner](https://fiware.github.io/tutorials.CRUD-Operations/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.CRUD-Operations.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)

These tutorials teach FIWARE users about CRUD Operations. The tutorial builds on the data created in the previous example and introduces the concept of
[CRUD operations](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete), allowing users to manipulate the data held within the context.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as
[Postman documentation](https://www.postman.com/downloads/).


# Start-Up

## NGSI-v2 Smart Supermarket

**NGSI-v2** offers JSON based interoperability used in individual Smart Systems. To run this tutorial with **NGSI-v2**, use the `NGSI-v2` branch.

```console
git clone https://github.com/FIWARE/tutorials.CRUD-Operations.git
cd tutorials.CRUD-Operations
git checkout NGSI-v2

./services create
./services start
```

| [![NGSI v2](https://img.shields.io/badge/NGSI-v2-5dc0cf.svg)](https://fiware-ges.github.io/orion/api/v2/stable/) | :books: [Documentation](https://github.com/FIWARE/tutorials.CRUD-Operations/tree/NGSI-v2) | <img src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/postman.svg" height="15" width="15"> [Postman Collection](https://fiware.github.io/tutorials.CRUD-Operations/) |  ![](https://img.shields.io/github/last-commit/fiware/tutorials.CRUD-Operations/NGSI-v2) |
| --- | --- | --- | --- |

## NGSI-LD Smart Farm

**NGSI-LD** offers JSON-LD based interoperability used for Federations and Data Spaces. To run this tutorial with **NGSI-LD**, use the `NGSI-LD` branch.

```console
git clone https://github.com/FIWARE/tutorials.CRUD-Operations.git
cd tutorials.CRUD-Operations
git checkout NGSI-LD

./services create
./services start
```

| [![NGSI LD](https://img.shields.io/badge/NGSI-LD-d6604d.svg)](https://cim.etsi.org/NGSI-LD/official/front-page.html) | :books: [Documentation](https://github.com/FIWARE/tutorials.CRUD-Operations/tree/NGSI-LD) | <img  src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/postman.svg" height="15" width="15"> [Postman Collection](https://fiware.github.io/tutorials.CRUD-Operations/ngsi-ld.html) |  ![](https://img.shields.io/github/last-commit/fiware/tutorials.CRUD-Operations/NGSI-LD) |
| --- | --- | --- | --- |


---

## License

[MIT](LICENSE) © 2018-2024 FIWARE Foundation e.V.
