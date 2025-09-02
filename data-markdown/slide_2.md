<!--

Why do we need a dedicated serialization tool?

Lukas explained the purpose of RDFProxy, but in the case of Releven we are still stuck with a problem: we have a *very complex model* for which we don't want to write our Pydantic models and SPARQL queries by hand. So how do we go from the models to a working RDFProxy backend?

By using WissKAS, another technical outcome of the project. What does WissKAS help us do? Releven is particularly challenging in terms of the affordances of the STAR model

This is just the basic shape, but once we start to put together different types of assertions about entities such as pepole, places or texts, things get much more complicated quickly.

Example: this is the people model, which we can inspect in the WissKI pathbuilder format. In it all possible assertions about people come together, such as assertions of their name appellations, birth, death, gender or other social properties. This is the full-fledged knowledge graph information we have about them, which is useful and powerful for inference. But sometimes we need less: a simple view of the data.

Let's say we are interested in just some basic details about a person. Using WissKAS, we can automatically derived a Pydantic model and corresponding query from the formal definitions.

The full toolchain used in the Releven project then looks like this:

The first use case for rdfproy: the Releven website, which allows browsing the data created in the Releven project.

-->

STAR model &rarr; <span style="width: 20em; text-align: center;">???</span> &rarr; RDFProxy

+++

**WissKAS: The WissKI Adapter Serializer**

+++

<img src="./data-markdown/star.png" alt="Basic RDF Graph" style="max-width: 60%; margin-top: 0.5em;">

+++

<iframe src="./pathbuilder.html" style="height: 80vh; width: 100%;" />

+++

<!--
uv run wisskas "wisski_pathbuilder.xml" endpoints \
  -p aaao "https://ontology.swissartresearch.net/aaao/" \
  -p crm "http://www.cidoc-crm.org/cidoc-crm/" \
  -p lrmoo "http://iflastandards.info/ns/lrm/lrmoo/" \
  -p rdfschema "http://www.w3.org/2000/01/rdf-schema#" \
  -p star "https://r11.eu/ns/star/" \
  -p skos "http://www.w3.org/2004/02/skos/core#" \
  -p r11 "https://r11.eu/ns/spec/" \
  -p r11pros "https://r11.eu/ns/prosopography/" \
  -c -0 \
  -li person person_display_name person_id_assignment.* person_id_assignment_identifier.person_id_assignment_identifier_plain person_possession_assertion#


from pydantic import AnyUrl, BaseModel
from rdfproxy import ConfigDict

class IdentityInOtherServices(BaseModel):
    person_id_assignment_identifier: AnyUrl
    person_id_assignment_external_authority: AnyUrl

class Person(BaseModel):
    model_config = ConfigDict(
        group_by="id",
    )
    person: AnyUrl

    person_display_name: str | None
    person_id_assignment: list[IdentityInOtherServices]
    person_possession_assertion: IdentityInOtherServices



PREFIX crm: <http://www.cidoc-crm.org/cidoc-crm/>
PREFIX lrmoo: <http://iflastandards.info/ns/lrm/lrmoo/>
PREFIX rdfschema: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX star: <https://r11.eu/ns/star/>


SELECT
  ?person
    ?person_display_name
    ?person_id_assignment
      ?person_id_assignment_identifier
      ?person_id_assignment_external_authority
    ?person_possession_assertion

WHERE {

  ?person a crm:E21_Person .

  OPTIONAL {
    ?person rdfschema:label ?person_display_name .
  }

  OPTIONAL {
    ?person ^crm:P140_assigned_attribute_to ?person_id_assignment .
    ?person_id_assignment a crm:E15_Identifier_Assignment .

    OPTIONAL {
      ?person_id_assignment crm:P37_assigned ?person_id_assignment_identifier .
      ?person_id_assignment_identifier a crm:E42_Identifier .
    }

    OPTIONAL {
      ?person_id_assignment crm:P14_carried_out_by ?person_id_assignment_external_authority .
      ?person_id_assignment_external_authority a lrmoo:F11_Corporate_Body .
    }
  }
  OPTIONAL { SELECT (COUNT(?person_possession_assertion) AS ?person_possession_assertion_count) ?person WHERE {
      ?person ^crm:P141_assigned ?Person_person_possession_assertion .
      ?person_person_possession_assertion a star:E13_crm_P51 .
  } GROUP BY ?Person }
  BIND (COALESCE(?person_possession_assertion_count, 0) AS ?person_possession_assertion)
}



-->

<div style="display: grid; grid-template-columns: repeat(2, 1fr); align-items: center; justify-items: center;">
<iframe src="./model.html" style="height: 90vh; width: 100%;"></iframe>
<iframe src="./query.html" style="height: 90vh; width: 100%;"></iframe>
</div>

+++

<img src="./data-markdown/wisskas.drawio.svg" style="filter:invert(1)">

+++

<a href="https://releven.acdh-ch-dev.oeaw.ac.at" target="_blank"><img src="./data-markdown/frontend.png" style="max-width: 100%;"></a>

+++

**WissKAS summary**

<ul style="font-size: 0.8em;">
<li class="fragment">takes WissKI pathbuilder definitions as input</li>
<li class="fragment">generates limited model using simple filter language
    <ul>
    <li class="fragment">Pydantic model</li>
    <li class="fragment">SPARQL query</li>
    </ul></li>
<li class="fragment"><b>RDFProxy endpoint from declarative specification!</b></li>
</ul>
