**Towards Declarative LOD Backends** <!-- .element: style="font-size: 120%; margin-bottom: 1.5em" -->

**Generating the RELEVEN Graph API from RDF Path Expressions** <!-- .element: style="font-size: 85%; margin-bottom: 1.5em" -->

Lukas Plank & Kevin Stadler <!-- .element style="margin: 2em 0 0.5em;" -->

<div class="font-size-75">
Austrian Centre for Digital Humanities<br> 
Austrian Academy of Sciences
</div>

+++

#### Recap
RDF Graphs, Semantic Technologies, STAR Model

<br>
	
<ul style="font-size: 0.8em;">
  <li class="fragment"><strong>RDF</strong> - Resource Description Framework</li>
  <li class="fragment">Semantic Triples: <strong>Subject – Predicate – Object</strong>
   <img src="./data-markdown/triple.svg"
         alt="Basic RDF Graph" style="max-width: 60%; margin-top: 0.5em;">
  </li>
  
  
  <li class="fragment"><strong>Explicit linking</strong> of entities via ontologically defined Predicates</li>
  <li class="fragment"><strong>URIs</strong> as global identifiers; <strong>Federation</strong></li>
  <li class="fragment">Automated <strong>Reasoning</strong> and <strong>Inferencing</strong></li>
</ul>


+++

**Semantic Expressiveness and Reification** <!-- .element: style="font-size: 85%; margin-bottom: 1.5em" -->
<img src="./data-markdown/star.svg" alt="Basic RDF Graph" style="max-width: 60%; margin-top: 0.5em;">
	

+++

**Semantic Data Persistence and Retrieval** <!-- .element: style="font-size: 100%; margin-bottom: 3em" -->


<ul style="font-size: 0.8em;">
  <li class="fragment"><strong>Triplestore</strong> or RDF Store: Database for Semantic Triples</li>
  <li class="fragment"><strong>SPARQL</strong>: Query Language for Semantic Data Retrieval

<div class="fragment">

```sparql
select ?s ?p ?o
where {
	?s ?p ?o .
}
```
	
```sparql
select ?s ?p ?o
where {
	?e13 a crm:E13_Attribute_Assignment ;
		crm:P140_assigned_attribute_to ?s ;
		crm:P177_assigned_property_type ?p ;
		crm:P141_assigned ?o .
}
```
</div>

  <li class="fragment"><strong>SPARQL Endpoint</strong>: Network Interface for Data Retrieval</li>
</li>
</ul>

+++

**SPARQL Result Sets** <!-- .element: style="font-size: 85%; margin-bottom: 1.5em" -->
	<div class="flex">
<div class="fragment">

SPARQL results are returned as flat rows <!-- .element: style="font-size: 75%" -->


| Author      | Work    | 
| ----------- | ------- |
| Shakespeare | Hamlet  |
| Shakespeare | Othello |
| Shakespeare | King Lear |
| Goethe      | Faust   |
<!-- .element: class="font-size-75 mx-25" -->

</div>
<div class="fragment bl-1">
API consumers usually expect nested JSON <!-- .element: style="font-size: 80%" -->


```json
{
   "authors":[
      {
         "name":"Shakespeare",
         "work":[
            "Hamlet",
            "Othello",
            "King Lear"
         ]
      },
      {
         "name":"Goethe",
         "work":[
            "Faust"
         ]
      }
   ]
}
```
</div>



+++

How to build a modern* REST API on top of a SPARQL Endpoint? <!-- .element: style="font-size: 85%; margin-bottom: 1.5em" -->

\*configurable, model-driven, typed, auto-documented, ... <!-- .element: style="font-size: 50%; margin-bottom: 1.5em" -->


+++

<p><strong>Our solution: RDFProxy</strong></p>

<p class="fragment">A Python library for running SPARQL queries <br/>and mapping SPARQL result sets to Pydantic models.</p>

+++

**Pydantic - Overview**

<div class="fragment">Pydantic is a data modeling and validation library based on Python's type annotation system</div>

<br/>
<ul style="font-size: 0.7em;">
  <li class="fragment">Standard Python classes and class-level attributes as modelling language</li>
  <li class="fragment">Features a Rust core for high-performance operations over models</li>
  <li class="fragment">Handles parsing, validation and serialization of data</li>
</ul>

<div class="fragment">

```python
from pydantic import BaseModel

class Point(BaseModel):
	x: int
	y: int
	
p1 = Point(x=1, y=2)    # Point(x=1, y=2)
p2 = Point(x=1, y=2.1)  # ValidationError: Input should be a valid integer...
```
</div>


+++

**RDFProxy Architecture** <!-- .element: style="font-size: 85%; margin-bottom: 1.5em" -->

<img src="./data-markdown/adapter.jpg" alt="SPARQLModelAdapter" style="max-width: 75%; margin-top: 0.5em;">

+++

**Example: RDFProxy FastAPI Backend** <!-- .element: style="font-size: 85%; margin-bottom: 1.5em" -->
```sparql
select ?name ?work
where {
  values (?name ?work) {
	("Goethe" "Faust")
	("Shakespeare" "Hamlet")
	("Shakespeare" "Othello")
	("Shakespeare" "King Lear")
  }
}
```

+++

```python
class Author(BaseModel):
	model_config = ConfigDict(group_by="name")

	name: str
	works: Annotated[list[str], SPARQLBinding("work")]


```
```python
adapter = SPARQLModelAdapter(
	target="https://query.wikidata.org/bigdata/namespace/wdq/sparql",
	query=query,
	model=Author,
)

app = FastAPI()

@app.get("/")
def route(
	query_parameters: Annotated[QueryParameters[Author], Query()],
) -> Page[Author]:
	return adapter.get_page(query_parameters)
```

+++

JSON Output

```json
{
  "items":[
	{
	  "name":"Goethe",
	  "works":[
		"Faust"
	  ]
	},
	{
	  "name":"Shakespeare",
	  "works":[
		"Hamlet",
		"Othello",
		"King Lear"
	  ]
	}
  ],
  "page":1,
  "size":100,
  "total":2,
  "pages":1
}
```
<!-- .element style="line-height: 1.4"-->


+++

**RDFProxy Summary** <!-- .element: style="font-size: 100%; margin-bottom: 1.5em" -->


<ul style="font-size: 0.8em;">
  <li class="fragment">RDFProxy aims to provide a generic library solution for building <br/>model-driven REST APIs on top of SPARQL endpoints.</li>
  <li></li>
  <li class="fragment">still early stage of development; 
  <br/>RELEVEN Project as <strong>challenging</strong> initial trial for the library
  </li>
</ul>


