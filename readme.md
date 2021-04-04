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

# Linking - product to product
```bash
java -jar _limes.jar _config_limes.xml
```

# AMIE - rule mining
```bash
# default run
java -jar _amie.jar statements.tsv
# save into file
java -jar _amie.jar -oute statements.tsv > output/rules_default.out -maxad 3 -minis 1 -optimai
# help / documentation
java -jar _amie.jar -h
```

# SPARQL REQUESTS
## Prefixes of all requests
```
PREFIX : <http://mapping.example.com/>
PREFIX schema: <https://schema.org/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dbo: <http://dbpedia.org/ontology/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX ken: <https://w3id.org/um/ken4256/>
PREFIX fo: <https://www.bbc.co.uk/ontologies/fo/>
PREFIX food: <http://data.lirmm.fr/ontologies/food#>
```

## Embedding: Creation of similarHealthierProduct relations between products
Added 501061 statements in the graph: http://kg-course/embedding.
Update took 1h 44m 20s. 
```
INSERT {
    GRAPH <http://kg-course/embedding>
    { ?product2 ken:similarHealthierProduct ?product1 }
}
# SELECT DISTINCT ?product1 ?product1Nutriscore ?product2 ?product2Nutriscore
WHERE { 
	?product1 a schema:Product ;
		rdfs:label ?product1Label ;
		food:nutritionScoreFrPer100g ?product1Nutriscore ;
		ken:mainCategory ?mainCategory1 ;
		fo:shopping_category ?category1 ;
		fo:shopping_category ?category11 .
	?category1 a fo:ShoppingCategory ;
		rdfs:label ?category1Label .
	?product2 a schema:Product ;
		rdfs:label ?product2Label ;
		food:nutritionScoreFrPer100g ?product2Nutriscore ;
		ken:mainCategory ?mainCategory2 ;
		fo:shopping_category ?category2 ;
		fo:shopping_category ?category22 .
	?category2 a fo:ShoppingCategory ;
		rdfs:label ?category2Label .

	FILTER(?product1Nutriscore = <https://w3id.org/um/ken4256/nutriscore/a> || ?product1Nutriscore = <https://w3id.org/um/ken4256/nutriscore/b>)
	FILTER(?product2Nutriscore != <https://w3id.org/um/ken4256/nutriscore/a> && ?product2Nutriscore != <https://w3id.org/um/ken4256/nutriscore/b>)
    
	FILTER(?product1 != ?product2)
    
	FILTER(?mainCategory1 != ?category1)
	FILTER(?mainCategory1 != ?category11)
	FILTER(?category1 != ?category11)
    
	FILTER(?mainCategory2 != ?category2)
	FILTER(?mainCategory2 != ?category22)
	FILTER(?category2 != ?category22)
    
	FILTER(?mainCategory1 = ?mainCategory2)
	FILTER(?category1 = ?category2)
	FILTER(?category11 = ?category22)
}
```

## List top number of similarProduct for each product
```
SELECT ?product ?mainCategory (COUNT(?similarProduct) AS ?count)
WHERE { 
	?product a schema:Product ;
		ken:mainCategory ?mainCategory ;
		ken:similarHealthierProduct ?similarProduct.
}
GROUP BY ?product ?mainCategory
ORDER BY DESC(?count)
```

## Count Products of specific (main) category
```
SELECT DISTINCT (COUNT(DISTINCT ?product) AS ?count)
WHERE { 
	?product a schema:Product ;
		ken:mainCategory ?mainCategory ;
		fo:shopping_category ?category1 ;
		fo:shopping_category ?category11 ;
	    	ken:similarHealthierProduct ?similarProduct .
    
	FILTER(?mainCategory = <https://w3id.org/um/ken4256/category/yogurts>)
	FILTER(?category1 = <https://w3id.org/um/ken4256/category/dairies>)
	FILTER(?category11 = <https://w3id.org/um/ken4256/category/fermented-milk-products>)
}
```

## Count number of products specific to a nutriscore (e.g. A)
```
SELECT (COUNT(DISTINCT ?product) AS ?count)
WHERE { 
	?product a schema:Product ;
		food:nutritionScoreFrPer100g ?nutriscore .

	FILTER (?nutriscore = <https://w3id.org/um/ken4256/nutriscore/e>)
}
```

## Get all similar healthier products based on a product (e.g. product id = 0000005016)
```
SELECT DISTINCT ?productOutput ?productOuputLabel
WHERE { 
	?productGiven a schema:Product ;
		rdfs:label ?productGivenLabel ;
		ken:similarHealthierProduct ?productOutput .
	?productOutput a schema:Product ;
		rdfs:label ?productOuputLabel .

	FILTER(?productGiven = <https://w3id.org/um/ken4256/product/0000005016>)
}
```

## Get all similar healthier products based based on a ingredient id and an ingredient label
With this query we have two options to filter products.
1) We can provide the id of an ingredient we want in products (e.g. we want the product to be made of chocolate)
2) We can also provide a part of the label of an ingredient we want (without FILTER NOT EXISTS) or not in products (e.g. we do not want the product to be made of milk)
+) In this example, we allow the products returned to be of nutriscore A or B.
```
SELECT DISTINCT ?productOutput ?nutriscore ?productOuputLabel
WHERE { 
	?productOutput a schema:Product ;
		food:nutritionScoreFrPer100g ?nutriscore ;
		fo:ingredients ?ingredientForId ;
		fo:ingredients ?ingredientForLabel ;
		rdfs:label ?productOuputLabel .
	?ingredientForLabel a fo:Ingredient ;
		rdfs:label ?ingredientLabel .

	FILTER(?nutriscore = <https://w3id.org/um/ken4256/nutriscore/a> || ?nutriscore = <https://w3id.org/um/ken4256/nutriscore/b>)
	FILTER(?ingredientForId = <https://w3id.org/um/ken4256/ingredient/chocolate>)
	FILTER NOT EXISTS {
		FILTER contains(?ingredientLabel, "milk")
	}
}
ORDER BY ?nutriscore
```