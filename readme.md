# Finding similar foods to change the unhealthy diet using Knowledge Graphs

This project consist of integrating one important publicly available data sets on foods (Open Food Facts).
In the first phase of this project, I transformed the dataset to an unified Knowledge Graph with the use of YARRRML (data serialization language).
Then I implemented AMIE, a graph mining algorithm in order to infer new rules and increase the completeness of the graph.
There is often a trade off between coverage and correctness. Since uncertain rules usually do not create true facts exclusively (otherwise they would be called certain rules),
there will wrong relations will be created too.

# Data
Open Food Facts is a collaborative project built by tens of thousands of volunteers and managed by a non-profit organization.
It consists of a database of food products with ingredients, allergens, nutrition facts and all the tidbits of information we can find on product labels.
The database gathers around 1 683 756 products.
CSV Data Export:
https://static.openfoodfacts.org/data/en.openfoodfacts.org.products.csv

➡️ products_original.csv

# To install
download latest rmlmapper:<br>
https://github.com/RMLio/rmlmapper-java/releases

follow these steps for YARRRML:<br>
https://rml.io/yarrrml/tutorial/getting-started/#writing-rules-on-your-computer

download latest amie3:<br>
https://github.com/lajus/amie/releases/tag/3.0

download latest graphdb:<br>
https://graphdb.ontotext.com/

# Query Editor (excel) Cleaning
1. Filter all products where 'states' value includes 'en:complete'
2. Filter all products which have a 'nutriscore_score' value
3. Delete all unnecessary columns
4. Transform all number type column to text column (to avoid 8,4E14)
5. Delete all backslashes

➡️ products_clean.csv

# Python Cleaning
1. Clean all unnecessary informations (e.g. quantity, proportion in 'ingredients_text')
2. Create dataset of categories, ingredients, countries based on all products
	1. Remove duplicated value
	2. Sort the values (alphabetically sort)
	3. Create 'id' column based on the label (because label can contain spaces)
	4. Export to CSV

➡️ categories.csv ingredients.csv countries.csv

3. Create dataset of products
	1. Create class Product
	2. Generate all products
	3. Export to JSON

➡️ products.csv

🤔 Dataset of products exported to JSON because YARRRML does not support the use of the function **grel:string_split** the creation of new IRIs with CSV dataset.
	
# Mapping (YARRRML to RML to RDF)
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

# GraphDB
## Initialization of the graph
1. Create new repository
2. Import RDF files (products.ttl, categories.tll, ingredients.ttl, countries.ttl) with http://kg-course/mapping for the target named graphs

## Visualize the graph
1. Explore > Visual Graph
2. Search RDF resources (e.g. https://w3id.org/um/ken4256/category/cakes)

# Linking - countries (dbpedia)
```bash
java -jar _limes.jar _config_limes.xml
```
<<<<<<< HEAD

# AMIE - rule mining
```bash
# default run
java -jar _amie.jar statements.tsv
# save into file
java -jar _amie.jar -UseGCOverheadLimit -Xmx4G -oute statements.tsv > output/rules_default.out -maxad 4 -minis 1
# help / documentation
java -jar _amie.jar -h
```
=======
>>>>>>> d2c1bf5a0c5c8fa8618c72023c363a1d5a3e6ebb
