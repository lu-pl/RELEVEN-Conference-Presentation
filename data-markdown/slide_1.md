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

#### SPARQL

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
