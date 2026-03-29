---
link: https://dashjoin.medium.com/json-schema-schema-org-json-ld-whats-the-difference-e30d7315686a
byline: Andreas Eberhart
site: Medium
date: 2020-08-07T14:08
excerpt: "JSON Schema, Schema.org, JSON-LD: What’s the Difference? Recently, I have seen several questions like “what’s the difference between JSON-LD and JSON Schema” or “can I use JSON Schema and …"
twitter: https://twitter.com/@dashjoin
slurped: 2026-03-28T09:57
title: "JSON Schema, Schema.org, JSON-LD: What’s the Difference?"
tags:
  - json
---



Recently, I have seen several questions like “what’s the difference between JSON-LD and JSON Schema” or “can I use JSON Schema and Schema.org”. I come from a linked data background (which is close to the world of Schema.org) but have recently started using JSON Schema a lot and I have to admit that there is no trivial answer to these questions. There is the obvious similarity in the standard names like “Schema” and “JSON”. If you compare the Schema.org page for [Person](https://schema.org/Person) to this [example](https://json-schema.org/learn/miscellaneous-examples.html) on the JSON Schema page, you have to admit that they kind of look alike. Combine this with the fact that Schema.org touts JSON-LD, which — by design — very much looks like regular JSON completes the confusion. So there definitely are enough reasons to write this article.

## JSON Schema

JSON Schema is to JSON what XML Schema is to XML. It allows you to specify the structure of a JSON document. You can state that the field “email” must follow a certain regular expression or that an address has “street_name”, “number”, and “street_type” fields. [Michael Droettboom](https://twitter.com/MDroettboom)’s book [“Understanding JSON Schema”](https://json-schema.org/understanding-json-schema/reference/object.html#properties) illustrates validation quite nicely with red & green examples.

The main use case for JSON Schema seems to be in JSON APIs where it plays two major roles:

1. Clients and servers can validate request and response objects in a generic way. This makes development a lot easier, since the implementation can “outsource” these checks to a standard component. Once a message passed the validation, you can safely assume that the document adheres to the rules.
2. As with any API, documentation is key when developers write code that uses it. JSON Schema is increasingly used to describe the structure of requests and responses by embedding it in an overall API description. Swagger is probably the most prominent example of this paradigm. Consider the [pet-store example](https://petstore.swagger.io/). Scroll all the way down on the page and you see this JSON Schema definition of “Pet”, which is a basic element in requests and responses of this API (you can find the actual JSON Schema embedded in the [raw Swagger file](https://petstore.swagger.io/v2/swagger.json) — note that currently there are still some differences between the OpenAPI specification and JSON Schema which will be resolved with OpenAPI 3.1).

As with all things related to code, reuse is a good idea. JSON Schema has the ability to import schemas using the [$ref](https://json-schema.org/understanding-json-schema/structuring.html) keyword. There are also efforts to share schemas. [JSON Schema Store](https://www.schemastore.org/json/) is one example. Its main use case is to support syntax highlighting for editors, for instance when editing a swagger file. At the time of writing, it contains over 250 schemas including — drum-roll please / you certainly guessed it — Schema.org. These describe things like [Action](https://json.schemastore.org/schema-org-action) and [Place](https://json.schemastore.org/schema-org-place). So the idea could be to centrally define JSON Schema building blocks that can be re-used in different APIs, making it easier to consume them, maybe even to the point where intelligent software can interact with APIs automatically. But before we get carried away, let’s have a look at Schema.org.

## Schema.org

[Schema.org](https://schema.org/) provides “schemas for structured data on the Internet”. Let’s assume you book a hotel and get a confirmation email. The email contains schema.org markup providing the contents of the email in machine readable form. This allows your [calendar to “understand” the email and add entry automatically](https://developers.google.com/gmail/markup/overview). We put “understand” in quotes because there really is no magic here. This very useful feature is made possible by the fact that the hotel’s IT system and the calendar agree on what a [hotel](https://schema.org/Hotel) is and also agree to represent this concept using markup like this:

{  
  "@context": “http://schema.org",  
  "@type": "Hotel",  
  "name": "ACME Hotel Innsbruck",  
  "[checkinTime](https://schema.org/checkinTime)": "13:00:00-05:00"  
  …  
}

In fact, it is almost like they agree on an API which happens to use email as the transport mechanism (please note that schema.org is not limited to email). It is important to note that Schema.org not only defines concepts or classes. The fields or properties are also standardized. Take [checkinTime](https://schema.org/checkinTime) for example, which is an XML Schema (time and timezone) string defining “the earliest someone may check into a lodging establishment”.

Schema.org not only defines agreed-upon class and property definitions, it also defines a hierarchy of classes and properties (a Hotel is a LodgingBusiness), which properties are allowed for which class (checkinTime can be used for LodgingBusiness and LodgingReservation) and the type of the properties (checkinTime is a DateTime or Time and starRating is a Rating).

Both Schema.org schemas and JSON schemas describe document structures using classes and properties / types and fields. The difference is that Schema.org is based on an Ontology which is published in different formats on [GitHub](https://github.com/schemaorg/schemaorg/tree/main/data/releases/9.0). An ontology:

1. defines classes and properties with agreed upon IRIs like: [https://schema.org/Hotel](https://schema.org/Hotel)
2. describes data where nodes (being an instance of a class) link to other nodes (via properties) forming a linked data graph
3. establishes a class and property taxonomy
4. treats properties as first class citizens which can originate from different classes (called domain) and can have different types as well (called range)

Now let’s get a bit more concrete and look at JSON-LD as one of the possible representations of Schema.org data.

## JSON-LD

The JSON-LD motto is “Data is messy and disconnected. JSON-LD organizes and connects it, creating a better Web.” Let’s take the hotel description as an example. We’re starting with a “normal” JSON representation:

{  
  "name": "ACME Hotel Innsbruck",  
  "[checkinTime](https://schema.org/checkinTime)": "13:00:00-05:00",  
  "starRating": {  
    "bestRating": 10,   
    ...  
  }  
  ...  
}

This JSON document has the following tree structure:

We are already using the Schema.org vocabulary, however there could be other vocabulary and especially a field like “name” is very likely to be ambiguous. Therefore, we specify our vocabulary as follows:

"@context": "http://schema.org/"

This causes “name” to become [http://schema.org/name](http://schema.org/name). The actual [context URL](https://schema.org/docs/jsonldcontext.jsonld) (via a content type redirect) is a simple list that defines a mapping from simple names to Schema.org URLs. [Other examples](https://json-ld.org/contexts/person.jsonld) define datatypes such as string, date, or link (@id).

Our tree is already a graph, however, we are lacking the information of which hotel we mean. In other words, we do not know the ID of the top graph node. We can specify this using:

"@id": "urn:acme-hotel"

We chose a URN, note that any IRI is possible. You could also use the hotel’s URL. The only prerequisite is that other participants can understand and interpret the ID.

Finally, we can add the information that this document describes a hotel:

"@type": "Hotel"

The resulting JSON-LD document is:

{  
  "@context": "http://schema.org/",  
  "@type": "Hotel",   
  "@id": "urn:acme-hotel",  
  "name": "ACME Hotel Innsbruck",  
  "checkinTime": "13:00:00-05:00",  
  "starRating": {  
    "bestRating": 10,   
    ...  
  }  
  ...  
}

which represents this structure:

Press enter or click to view image in full size

If you paste the example into the [JSON-LD playground](https://json-ld.org/playground/) you get this exact graph (we are choosing the table representation where each link above becomes one table row stating that subject predicate object).

Note that the rating node has the ID _:b0 which is an anonymous ID. This means that the rating cannot be referenced by other documents and it only exists as the child of its parent object. The hotel, though, can be referenced by other documents. For instance, a person (http://example.org/joe) can be affiliated with the hotel:

{  
  "@context": "http://schema.org/",  
  "@id": "http://example.org/joe",  
  "affiliation": {  
    "@id": "urn:acme-hotel"  
  }  
}

Both documents can be combined, resulting in a graph with one additional link from Joe to the hotel. Adding additional properties under id would add another property of the hotel.

## OK, Now What?

We looked at JSON Schema, Schema.org, and JSON-LD, and now the question is, can we combine the approaches? Let’s look at three possibilities:

### Validating JSON-LD Using JSON Schema

The first idea that comes to mind would be to use JSON Schema to validate JSON-LD. As a matter of fact, JSON-LD publishes a [JSON schema](https://github.com/json-ld/json-ld.org/blob/master/schemas/jsonld-schema.json). However, the schema only looks at JSON-LD keywords and overall document structure and does not validate the actual ontology. It makes more sense to validate JSON-LD by converting it into an RDF graph and validating it via the underlying ontology using RDF / [OWL / SHACL](https://spinrdf.org/shacl-and-owl.html) tooling.

### Generating JSON Schema from Schema.org

The next approach could be to generate JSON schemas from Schema.org. Remember Schema.org being present on the schema store website? It turns out that one of the main problems is the handling of arrays. In a “normal” JSON schema, you specify whether a property is an array or has a single value. In the linked data world, any property can be repeated. So it is perfectly legal for two sources to state check in times for the hotel. Therefore, the graph data model will always return a list of values when you ask for a property of a given graph node.

One can work around this by defining all properties as “oneOf” single value or array of values, but this leads to complex and convoluted schemas. [Some projects](https://github.com/geraintluff/schema-org-gen) allow you to define the cardinality a priori as a parameter of the generation process. The schema on [schemastore](https://json.schemastore.org/schema-org-place), for example, makes the following choices: smokingAllowed has a single value whereas amenityFeature is an array. This certainly makes sense. The telephone number also has a single value and one can certainly make the case that it should be an array.

### Linting JSON Schema Using Schema.org

The JSON Schema website lists a number of [linter tools](https://json-schema.org/implementations.html#schema-linter) that check the schema itself. One approach could be to encourage the use of Schema.org vocabulary, so a linting suggestion could be to change amenity_feature to amenityFeature since it can easily be mapped to [https://schema.org/amenityFeature](https://schema.org/amenityFeature). The benefit for developers would be that they can reuse the Schema.org documentation. Also, maybe one day their REST services can be understood and consumed by intelligent clients.

## Summary

It is clear that Schema.org and JSON-LD on the one hand and JSON Schema on the other come from different angles and do not really fit together naturally. Nevertheless, I believe that each community can benefit from the other. JSON Schema can learn about ontologies, reuse, and semantics. The linked data community can learn from the pragmatism of JSON Schema. In fact, I think JSON-LD already learned this lesson and is a great improvement over NTriples or RDF/XML. Likewise, approaches such as [JSON API](https://jsonapi.org/) introduce linked data principles to the world of REST APIs.