# Data
Open Food Fact database: CSV Data Export:
https://static.openfoodfacts.org/data/en.openfoodfacts.org.products.csv
<br>➡️ products_original.csv

# To install
download latest rmlmapper:<br>
https://github.com/RMLio/rmlmapper-java/releases

follow these steps for YARRRML:<br>
https://rml.io/yarrrml/tutorial/getting-started/#writing-rules-on-your-computer

download latest amie3:<br>
https://github.com/lajus/amie/releases/tag/3.0

# Query Editor (excel) Cleaning
1. Filter all products where 'states' value includes 'en:complete'
2. Filter all products which have a 'nutriscore_score' value
3. Delete all unnecessary columns
4. Transform all number type column to text column (to avoid 8,4E14)
5. Delete all backslashes
<br>➡️ products_clean.csv

# Python Cleaning
1. Clean all unnecessary informations (e.g. quantity, proportion in 'ingredients_text')
2. Create dataset of ingredients, countries, categories based on all products
	1. Remove duplicated value
	2. Sort the values (alphabetically sort)
	3. Create 'id' column based on the label (because label can contain spaces)
	4. Export to CSV
3. Create dataset of products
	1. Create class Product
	2. Generate all products
	3. Export to JSON
<br>➡️ products.csv

🤔 Dataset of products exported to JSON because YARRRML does not support the use of the function **grel:string_split** the creation of new IRIs with CSV dataset.
	
# Mapping (YARRRML to RDF)
## Product
```bash
yarrrml-parser -i products.yarrrml.yml -o products.rml.ttl
java -jar _rmlmapper.jar -m products.rml.ttl -o products.ttl -s turtle
```
➡️ products.ttl

## Categories
```bash
yarrrml-parser -i categories.yarrrml.yml -o categories.rml.ttl
java -jar _rmlmapper.jar -m categories.rml.ttl -o categories.ttl -s turtle
```
➡️ categories.ttl

## Ingredients
```bash
yarrrml-parser -i ingredients.yarrrml.yml -o ingredients.rml.ttl
java -jar _rmlmapper.jar -m ingredients.rml.ttl -o ingredients.ttl -s turtle
```
➡️ ingredients.ttl

## Countries
```bash
yarrrml-parser -i countries.yarrrml.yml -o countries.rml.ttl
java -jar _rmlmapper.jar -m countries.rml.ttl -o countries.ttl -s turtle
```
➡️ countries.ttl

# Linking
```bash
java -jar _limes.jar _config_limes.xml
```