# DATA
Open Food Fact database: CSV Data Export:
https://static.openfoodfacts.org/data/en.openfoodfacts.org.products.csv

# TO INSTALL
download latest rmlmapper:
https://github.com/RMLio/rmlmapper-java/releases

follow these steps for YARRRML:
https://rml.io/yarrrml/tutorial/getting-started/#writing-rules-on-your-computer

download latest amie3:
https://github.com/lajus/amie/releases/tag/3.0

# MAPPING
## Product
```bash
yarrrml-parser -i products.yarrrml.yml -o products.rml.ttl
java -jar _rmlmapper.jar -m products.rml.ttl -o products.ttl -s turtle
```
## Categories
```bash
yarrrml-parser -i categories.yarrrml.yml -o categories.rml.ttl
java -jar _rmlmapper.jar -m categories.rml.ttl -o categories.ttl -s turtle
```
## Ingredients
```bash
yarrrml-parser -i ingredients.yarrrml.yml -o ingredients.rml.ttl
java -jar _rmlmapper.jar -m ingredients.rml.ttl -o ingredients.ttl -s turtle
```
## Countries
```bash
yarrrml-parser -i countries.yarrrml.yml -o countries.rml.ttl
java -jar _rmlmapper.jar -m countries.rml.ttl -o countries.ttl -s turtle
```

# LINKING
```bash
java -jar _limes.jar _config_limes.xml
```