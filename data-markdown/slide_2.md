**WissKAS: The WissKI Adapter Serializer** <!-- .element: style="font-size: 85%; margin-bottom: 1.5em" -->

STAR model -> <span style="width: 20em; text-align: center;">???</span> -> RDFProxy

+++

STAR model -> <span style="width: 20em; text-align: center;">WissKAS</span> -> RDFProxy

+++

<img src="./data-markdown/star.png" alt="Basic RDF Graph" style="max-width: 60%; margin-top: 0.5em;">

+++

<img src="./data-markdown/wisskas.png" style="max-width: 100%;">

+++

```py
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
    person_possession_assertion: int
```

```sparql
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
```

+++

<img src="./data-markdown/wisskas.drawio.svg" style="max-width: 100%; margin-top: 0.5em;">

+++

<img src="./data-markdown/frontend.png" style="max-width: 100%;">

<https://releven.acdh-ch-dev.oeaw.ac.at>
