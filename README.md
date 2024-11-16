### Innovation for Good
# How do we fix the Web?
Now that we've learned about the origins of the Web, the boom of APIs in the 2010s and the potential of Linked Data and Solid, it's time for some hands-on experience.

## Pre-requisites
Install [Visual Studio Code](https://code.visualstudio.com/) on your machine.

Ensure you have a [Docker](https://docs.docker.com/desktop/) or [Podman](https://podman.io/) installation.

Clone this repository and open it in Visual Studio Code. You will be recommended some extensions, install them.

## 1. Exploring the Semantic Web
[RDF](https://www.w3.org/RDF/) is the foundational information model of the Semantic Web. Several different serializations of RDF exist, the most popular of which are [JSON-LD](https://json-ld.org/), [Turtle](https://www.w3.org/TR/turtle/) and [N-Quads](https://www.w3.org/TR/n-quads/).

TODO: Show basic information model

TODO: Contact information as linked data

1. Create your own contact profile as Linked Data, as demonstrated above. (**SIMPLE**)
2. Now what if you want to add some of your interests to this profile? What vocabularies could you use? How would you go about finding them? (**INTERMEDIATE**)


## 2. Querying the Semantic Web
The power of the Semantic Web lies not only in the simplicity of its data model and its global semantics, but also in the ability to query heterogeneous data sources taking advantage of the re-use of vocabularies. In this section we'll be exploring the potential of this querying affordance.

### DBPedia
DBpedia is a public knowledge base that extracts structured data from Wikipedia. It converts Wikipedia's information into a standardized format (RDF triples) that computers can easily process and query using SPARQL. DBPedia is a great first demonstrator of the potential of the Semantic Web, so it's only natural we introduce it here.

Suppose we want to find songs by American rock-icon Bruce Springsteen, we could open up his Wikipedia page and go to the discography of the singer. But what if we want this information as structured data? We might be able to find domain-specific APIs on the Web, but as our queries grow more complex we'll have to build a lot of additional logic to integrate the required data and come to a result.

In SPARQL, with the help of DBPedia, such a query becomes relatively simple:
```
SELECT ?song ?title
WHERE {
  ?song dbpedia-owl:artist [ rdfs:label "Bruce Springsteen"@en ];
         a dbo:Song;
         rdfs:label ?title.
}
```
<small>[Try this query for yourself](https://query.comunica.dev/#datasources=https%3A%2F%2Fdbpedia.org%2Fsparql&query=SELECT%20%3Fsong%20%3Ftitle%0AWHERE%20%7B%0A%20%20%3Fsong%20dbpedia-owl%3Aartist%20%5B%20rdfs%3Alabel%20%22Bruce%20Springsteen%22%40en%20%5D%3B%0A%20%20%20%20%20%20%20%20%20a%20dbpedia-owl%3ASong%3B%0A%20%20%20%20%20%20%20%20%20rdfs%3Alabel%20%3Ftitle.%0A%7D)</small>



### Querying governmental linked data in Flanders

The Flemish government publishes several [base registries](https://basisregisters.vlaanderen.be/), to promote re-use of data and avoid double registrations. One of these base registries is the [buildings and address registry](https://www.vlaanderen.be/digitaal-vlaanderen/onze-oplossingen/gebouwen-en-adressenregister).

The base registries are available through traditional [RESTful APIs](https://docs.basisregisters.vlaanderen.be/docs/api-documentation.html#tag/api-documentation.html) but follow a Linked Data standard defined by the Flemish OSLO initiative.

Following the principles of Linked Data, each building or address also has a unique URI associated with it. For example https://data.vlaanderen.be/doc/adres/3706808 is the URI associated with the address of Digitaal Vlaanderen in Ghent ("Koningin Maria-Hendrikaplein 70, 9000 Gent").

How do we get to this URI? Well, by querying the Linked Data published by the Flemish government.

Digitaal Vlaanderen provides a [SPARQL endpoint](https://data.vlaanderen.be/sparql#) allowing you to query the Linked Data it publishes via the SPARQL query language.

```SPARQL
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT distinct ?adresUri WHERE {
  ?adresUri <https://data.vlaanderen.be/ns/adres#heeftGemeentenaam> ?city .
  ?city rdfs:label "Gent".
  ?adresUri <https://data.vlaanderen.be/ns/adres#heeftStraatnaam> ?naam.
  ?naam rdfs:label "Koningin Maria Hendrikaplein"@nl.
  ?adresUri <https://data.vlaanderen.be/ns/adres#huisnummer> "70".
}
LIMIT 1
```
The query above allows us to find the URI above, let's break this down. The query starts with two lines of `PREFIX` definitions, where aliases are defined that are re-used later in the query. These make our query more concise. Our actual query starts with a `SELECT` statement, indicating we want to retrieve data rather than update or delete it. This keyword is called a [query form](https://www.w3.org/TR/sparql11-query/#QueryForms) and SPARQL supports several of them.

After indicating our query form we define the variables which should be returned in the response of our query, in this case we have one variable `adresUri` of which we want all distinct values. After the `WHERE` keyword follows a clause which defines the basic graph pattern against which the query engine will match its triples or quads.

A basic graph pattern (BGP) in SPARQL is a set of triple patterns where you match subjects, predicates, and objects. Variables (marked with ?) act like wildcards that can match any value. For example
`?city rdfs:label "Gent"` will match any subject having a triple statement with predicate `rdfs:label` having the literal value `"Gent"`.



Now, over to you:

1. Can you modify the query above to find the URI of your home address (if you live in Flanders)? (**SIMPLE**)
2. How many distinct cities in Flanders have a street named "Kerkstraat"? (**INTERMEDIATE**)
3. Can you use your knowledge of SPARQL and Linked Data to figure out in which cities the political party "Groen" has a mayor in the last legislature? (**HARD**)

## 3. Getting started with Solid
For the next part of our exercise we will be using the [Community Solid Server](https://github.com/CommunitySolidServer/CommunitySolidServer).

```sh
npx @solid/community-server -c @css:config/file.json -f data/
```