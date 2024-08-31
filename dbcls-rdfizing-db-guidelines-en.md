# Guidelines for converting DBCLS database to RDF

The following is an automatic translation of [the DBCLS Guidelines for rdfizing databases, written in Japanese](https://github.com/dbcls/rdfizing-db-guidelines/blob/master/dbcls-rdfizing-db-guidelines-ja.md).

## Introduction - Linked Data Concept

[RDF](http://www.w3.org/TR/rdf11-concepts/) is a framework for describing unambiguous and machine-readable data. However, it does not provide guidance on how to actually convert the data you want to describe into RDF in order to describe what you want to express, and how to make it better in terms of subsequent use. This is a high hurdle to overcome when converting databases to RDF, especially for beginners. Fortunately, there has been a lot of discussion and knowledge accumulated at [BioHackathon](http://biohackathon.org) and [SPARQLthon](http://wiki.lifesciencedb.jp/mw/SPARQLthon) It's time for this community to start working on RDF data. I think it is time to create a guideline to guide this community in converting data to RDF. The goal of these guidelines is to make it possible to create RDF that can be used as a reference to reduce the burden of RDF conversion work and to integrate it appropriately with other data.

In the basic spirit of the [Linked Data initiative](http://www.w3.org/DesignIssues/LinkedData.html) by Tim Berners-Lee
The Linked Data initiative recommends that data be created in accordance with the

>1. use URIs as names for things → name things using URIs
>2. Use HTTP URIs so that people can look up those names.
> Accessible with widely popular software, http://
> Use URIs starting with > in the name so that users can look it up (some URIs are not accessible in widely used software)
>3. When someone looks up a URI, provide useful information, using the
> standards (RDF\*, SPARQL) →
> Provide useful information according to standards such as RDF and SPARQL when accessing URIs
>Include links to other URIs. so that they can discover more things.
> > →
> Include links to other URIs so that further information can be traced

Four principles have been proposed.

RDF is a [Resource Description
Framework](https://ja.wikipedia.org/wiki/Resource_Description_Framework), where URI is an abbreviation for [Uniform
Resource Identifier](https://ja.wikipedia.org/wiki/Uniform_Resource_Identifier). RDF is an abbreviation for [Resource](https://ja. wikipedia.org/wiki/%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9_(WWW)), and does not refer to a specific file format. In fact, there are several formats for describing information using RDF, including RDF/XML, N-Triples, Turtle, and JSON-LD, but the [Turtle](http://www.w3.org/TR/turtle/) format is highly readable not only by machines but also by people, and it has a lot in common with SPARQL Since the [Turtle] () format is highly human readable as well as machine readable and has many similarities with the SPARQL query notation, the Turtle format is used in this guideline.

## 1 Guidelines for converting database content to RDF

The following are recommended guidelines for converting databases to RDF based on the knowledge we have accumulated so far.

### 1.1 Guidelines for designing URIs

#### 1.1.1 Use persistent URIs

The guideline of using persistent URIs is based on the Tim Berners-Lee
by "[Cool URIs never change](http://www.w3.org/Provider/Style/URI)".
It started with the declaration "[[(Japanese translation)](http://www.kanzaki.com/docs/Style/URI)", and also from W3C "Cool URIs for the Semantic Web
[(original)](http://www.w3.org/TR/cooluris/),.
[(Japanese translation)](http://standards.mitsue.co.jp/resources/w3c/TR/2008/NOTE-cooluris-20080331/)"
The document is available to the public.

RDF data describes information about a URI resource. And Linked
In Principle 2 of Data, http://
and in Principle 3, it is recommended that information about a URI resource be obtained using standards such as RDF by accessing the URI in question. However, if the originally accessible URI is changed to a different URI or disappears for some reason, the value of the RDF data is significantly reduced. For example, many life science databases are created by research institutions, and it is not uncommon for the URI of an organization to change due to reorganization, consolidation, or discontinuation. Therefore, if RDF data is described with a URI that uses the domain name of the organization, even if the database is fortunate enough to be operated by a new organization, the URI will not be accessible from the RDF data. To avoid this situation, it is recommended that you consider the following two options

- Obtain your own domain name

One common case is to get your own domain, such as uniprot.org, or "database-name.org". This allows the same URI to persist even if the entity operating the database changes. The disadvantage is that it is expensive to maintain your own domain and creates other permanence issues such as who maintains that domain.

- Use PURL

Another commonly used case is to describe data using a persistent URL, [Persistent URL (PURL)](https://en.wikipedia.org/wiki/Persistent_uniform_resource_locator). RDF is described using the URI of a PURL, and by forwarding (redirecting) from the PURL to the URL of the entity operating the data, the same URI can be used persistently without changing the RDF data even if the entity is moved. For
[purl.jp](http://purl.jp/) service. By managing transfers with the PURL service, you can always point to the most current URI accessible at the time.

#### 1.1.2 URIs that indicate a resource should have an ID at the end of the URI to identify it.

For the path portion of the URI that follows the highly persistent domain portion, it is recommended that the ID that identifies the resource to be described be placed at the end of the URI, and that a slash (/) be used immediately before the ID. For example, UniProt assigns the URI <http://purl.uniprot.org/uniprot/Q6GZX3> to the protein identified by the ID Q6GZX3. By structuring URIs in this way, it is technically easier to construct services that "provide useful information in accordance with standards such as RDF and SPARQL when the URI is accessed," as stated in Principle 3 of Linked Data.

In the URI of an ontology, if a separate page is created for each ID that indicates the concepts that make up the ontology, it is best to use a slash (/) immediately before the ID, as above, and if the entire ontology is provided on a single page, it is best to use a hash (\#) immediately before the ID instead of a slash is a good idea. As an example, in the UniProt ontology `http://purl.uniprot.org/core/`, concepts such as Protein are provided on separate pages, so a slash (/) are used. On the other hand, in the sequence position ontology [FALDO](http://biohackathon.org/resource/faldo), concepts such as Position are provided on the same page, so the URI fragment notation is used, `http:// Hash (\#) is used, as in `biohackathon.org/resource/faldo#Position`.

#### 1.1.3 Handling of IDs containing version information

NCBI Gene, NCBI Protein, etc. provide IDs with and without version numbers (e.g., https://www.ncbi.nlm.nih.gov/protein/NP_003024.1 and https://www.ncbi.nlm.nih. gov/protein/NP_003024 ). The question is which of these IDs should be used. If the emphasis is on linking to other datasets, it is better to use IDs that do not contain version numbers, so that links between datasets are more likely to be maintained. On the other hand, if you want to describe information about a specific version of an ID, you should use an ID that includes a version number. In other words, a more appropriate ID should be used depending on the situation. However, from the perspective of maintaining links between data sets, it is preferable to also provide links to IDs without version numbers, even if IDs with version numbers are used.
In addition, if you want to know the latest version, you can use [`TogoWS`](http://togows.org) to get the latest version of each database with respect to the supported databases in the following way.

````
http://togows.org/entry/ncbi-protein/145579718/version
````

### 1.2 Guidelines for creating RDF

#### 1.2.1 URI resources are defined as instances of classes in the ontology

It is important that the ontology class be specified by `rdf:type` to clearly and concisely express what is meant by the resource indicated by the URI. In particular, it is recommended that the principal resource have a class specified based on the ontology.

````
uniprot:Q6GZX3 rdf:type core:Protein .
````

In the example above, we can see that the Q6GZX3 entry in UniProt is [`core:Protein`](http://purl.uniprot.org/core/Protein). In addition to making it clear what the resource is pointing to, the `rdf:type` can also be specified in a SPARQL search for efficient data exploration. It would be ideal if ontology definitions could be referenced by accessing URIs, but it is not required that this be the case. However, it is common for natural language to have different meanings in different contexts of use, and describing the definition intended by the data creator in the URI reference can be an effective means of avoiding misunderstandings by data users. This is a great advantage when describing scientific data.

The question becomes, "What class should I type for my resource?" If you can find an appropriate class in the existing ontology, you can use it. If you can find an appropriate class in an existing ontology, you can use it, but in many cases you may not be able to find it. UniProt and EBI RDF also define the necessary classes and provide them with the data.

In RDF,

- When `rdfs:Class` is the object of `rdf:type`, its subject is the class
````
ex:Myclass rdf:type rdfs:Class .
````
- When the URI of a class is the object of `rdf:type`, its subject is the instance
````
ex:111 rdf:type ex:Myclass .
````

respectively (a subject is a class even if it is not explicitly typed `rdfs:Class` but is defined as a subclass of some class by `rdfs:subClassOf`). However, making a certain URI an instance as well as a class complicates the semantic handling of data, so please avoid it if you can.

````
#Deprecated  
ex:111 rdf:type rdfs:Class # ← ex:111 is also a class  
ex:111 rdf:type ex:Myclass # ← also an instance, that is  
````

#### 1.2.2 Labeling URI resources

In order for humans to easily understand what the resource indicated by a URI is, it is useful to have the label described in natural language by means of `rdfs:label`. In particular, it is recommended that principal resources be described with concise labels. This has nothing to do with increasing machine readability, but it does increase human readability, which can be useful when creating applications or displaying the results of a SPARQL search in a readable manner.

````
uniprot:P51028 rdfs:label `"`wnt8a`"`@en . `
````

In addition, adding language tags, as in this example, is especially useful for multilingualization. Non-native English speakers may want to add labels or comments in their native language as well. By adding language tags, it is possible to explicitly write in multiple languages, and it is also easy to use only English labels when using them in an application.

````
mpo:MPO_03001 rdfs:label "Thermophilic"@en , "hyperthermic"@en .
````

It is more convenient to have only one label by a certain language so that you do not have to worry about using it. If you want a different label, you can use `skos:altLabel` instead of `rdfs:label`. Also, if there are multiple labels by `rdfs:label` or `skos:altLabel` and you want to specify one of them as a representative label, you can use `skos:prefLabel`.

````
chebi:CHEBI_17234 rdfs:label "D-Glucose"@en ; 
                  rdfs:label "D-Glucose""@en ; 
                  skos:altLabel "Dextrose"@en ; 
                  skos:altLabel ""glucose""@en ; 
                  skos:prefLabel "D-Glucose"@en .
````

#### 1.2.3 Attach ID labels to URI resources

The URI itself serves as a global ID. However, URIs are not suitable for human viewing when displaying SPARQL results because they are symbolic and long strings, and if the URI contains a database-specific (local) ID at the end, the ID can be obtained as a string by truncating the last / of the URI. However, this requires extra work, such as applying string processing in SPARQL. Therefore, it is recommended to use `dcterms:identifier` to describe the ID string of the main resource.

````
uniprot:P51028 dcterms:identifier "P51028" .
````

Please do not use the `dcterms:identifier` property to declare multiple IDs for a resource identified by an ID.

````
Deprecated for ## and below.
pdb:2RH1 dcterms:identifier "2RH1" . ` 
pdb:2rh1 dcterms:identifier "2rh1" . ` 
````

Due to the intrinsic meaning of ID identifiers, different IDs cannot be given, but if for some reason you wish to declare more than one identifier (for example, in both upper and lower case cases, as in the above example, to respond to a query), please declare them using different properties.

````
No problem below #.
pdb:2RH1 dcterms:identifier "2RH1" .
pdb:2rh1 skos:altLabel "2rh1" .
````

#### 1.2.4 Linking to other datasets

In RDF, the Web of Data (Linked Data) is achieved by linking references to external resources, as indicated in Principle 4 of Linked Data. When the cross-reference destination is a database entry, it is common practice to link to a URL where the referenced DB entry can actually be viewed. This is called a "polite URL" in the sense of respecting the original site. However, the problem with using the original URL is that,

- If the URL of a linked page is changed, the change is not automatically applied to the RDF once published.
- The original URL may not be a cool URI (e.g., a dynamically generated page with CGI arguments may not be suitable as an ID).

etc. In addition, multiple URLs may exist to refer to the same item (e.g., [Taxonomy ID](http://info.identifiers.org/taxonomy/9606), for which sites such as NCBI, EBI, UniProt, Bio2RDF, etc. provide different URLs (e.g., [Taxonomy ID](), etc.).

- If individual databases provide RDF using different URIs to point to the same thing, integrated searches will not be possible.

There is also the issue of

One way to avoid these problems is to use [<http://identifiers.org/>](http://identifiers.org/) URIs. Identifiers.org is similar to PURL, but it is a service that specializes in database URIs. Identifiers.org is maintained by the life science community and can be requested even if the database you wish to reference is not registered. If the database you wish to reference is not already registered, you can request it to be added.

For this reason, when writing cross references,

````
Example)  
ex:111 rdfs:seeAlso <http://pfam.xfam.org/family/PF01590> .
ex:111 rdfs:seeAlso <http://identifiers.org/pfam/PF01590> .
````

You can create RDF that is easily connected to external resources by placing reference links to both the polite URL and the Identifiers.org URI, as in the following example.

For databases whose RDF is publicly available, the NBDC RDF portal recommends linking against both the URI used in that RDF and the Identifiers.org URI (if available). Identfiers.org URI for which a triple will be generated on the NBDC RDF portal side so that the URI is an instance of the class representing the database to which it belongs (as of 2017.11).

##### URI prefixes of major databases where RDF is published and resources used for external linking

| Database Name | Class | URI Prefix |
|-----|------|-------------|
| UniProt| core:Protein | http://purl.uniprot.org/uniprot/ |
| Ensembl| obo:SO_0001217(protein_coding_gene) | http://rdf.ebi.ac.uk/resource/ensembl/ |
| ChEMBL| cco:Substance | http://rdf.ebi.ac.uk/resource/chembl/molecule/ |
| ExpressionAtlas| atlas:BaseLineExpressionValue | http://rdf.ebi.ac.uk/resource/expressionatlas/ |
|| atlas:DifferentialExpressionRatio | http://rdf.ebi.ac.uk/resource/expressionatlas/ |
| Reactome| biopax3:Pathway | http://identifiers.org/reactome/ |
| BioModels| ||
| BioSamples| biosd-terms:Sample | http://rdf.ebi.ac.uk/resource/biosamples/sample |
| PubChem| compound | http://rdf.ncbi.nlm.nih.gov/pubchem/compound/ |
| substance | http://rdf.ncbi.nlm.nih.gov/pubchem/substance |
| MESH | meshv:TopicalDescriptor | http://id.nlm.nih.gov/mesh/ |
| wwPDB| PDBo:datablock | http://rdf.wwpdb.org/pdb/1NH2 |


#### 1.2.5 Add a link to the literature information

If you know the bibliographic information on which you are basing your description, please actively add a link to the bibliographic information. Whenever possible, please link to literature information using the [PubMed](http://pubmed.org/) or [DOI](http://doi.org/) IDs. It is recommended that the following prefixes be used to URI these IDs.

- URI of PubMed ID <http://rdf.ncbi.nlm.nih.gov/pubmed/24495517>
- URI of DOI ID <http://doi.org/10.1021/jo0349227>

If a PubMed ID or DOI ID is not available, describe it as an instance of `bibo:Article` in [Bibliographic Ontology](http://bibliontology.com/) if it is a scholarly literature. If it is a book, you can also use `bibo:Book`. Detailed information about the literature (journal name, volume number, page number, publisher, etc.) is also available at [Bibliographic Ontology](http://bibliontology.com/) [[GitHub](https://github.com/ structureddynamics/Bibliographic-Ontology-BIBO)\}]
We recommend the use of (Reference) [How to use with CINII](https://support.nii.ac.jp/ja/cia/api/a_rdf)

When linking to literature information, use [`dcterms:references`](http://purl.org/dc/terms/references)
property is recommended.

````
<a resource> dcterms:references pubmed:24495517 .
````

For more detailed information on the literature, we recommend using [Bibliographic Ontology](http://bibliontology.com/). (Reference)
[How to use in CINII](https://support.nii.ac.jp/ja/cia/api/a_rdf)

````
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix bibo: <http://purl.org/ontology/bibo/> .
@prefix prism: <http://prismstandard.org/namespaces/1.2/basic/> .

<a resource> dcterms:references [ 
  a bibo:Article;.  
  prism:publicationName "Nature science cell";.  
  prism:volume "10";.  
  prism:number "11";.  
  prism:startingPage "123";.  
  prism:endingPage "456";.  
  dcterms:date "2015-12-08" ; 
  seeAlso <http://rdf.ncbi.nlm.nih.gov/pubmed/1234567>  
] .
````

#### 1.2.6 Attaching meta information to data

One of the advantages of converting data to the Semantic Web is that you can add as much metadata as you want to an RDF dataset, making it possible to specify the date it was created, who created it, the data source, the data category, the license, and so on.
The [VoID](http://www.w3.org/TR/void/) is a vocabulary for describing metadata about RDF datasets.
Tools for efficiently constructing metadata for RDF datasets using this vocabulary include, for example, the [VoID Editor](http://voideditor.cs.man.ac.uk/).

Each statement (i.e., each triple) described in RDF can also be given a history (a "provenance"). Especially in the case of scientific data, each statement is often a statement of fact, so the history that led to the statement is important. This history can range from information in articles and books, information obtained by mechanical methods (e.g., homology search using BLAST), and even personal speculation. Explicitly describing the history of arrival (e.g., using only information mentioned in the paper) allows the user to choose how each triple should be used.

Several vocabularies have been proposed for describing provenance information in RDF ([DC terms](http://purl.org/dc/terms/), [PROV-O](http://www.w3.org/TR/prov-o/), [PAV](http://bioportal .bioontology.org/ontologies/PAV)), and several models for describing provenance information have been proposed ([RDF Reification](https://goo.gl/RXvpBE), [NanoPub](http://nanopub OvoPub](http://arxiv.org/pdf/1305.6800.pdf), [VoAG](http://linkedmodel.org/doc/voag/1.0/), etc.). Standardization of RDF data and development of tools/ontologies](http://goo.gl/u32WCo) (page 8).

The Evidence Code Ontology can also be used to describe the level of evidence in annotations, for example.

#### 1.2.7 Link to image

- When linking to an image that represents the entity that the subject URI means, [foaf:depiction](http://xmlns.com/foaf/spec/#term_depiction)
    to be used.
````
an-assay-db:12345 foaf:depiction an-assay-db-image:12345.jpg .
````

- Conversely, if the URI of the image file is the subject and the resource URI of what is depicted in the image is the object, then [foaf:depicts](http://xmlns.com/foaf/spec/#term_depicts)
    to be used.
````
an-assay-db-image:12345.jpg foaf:depicts an-assay-db:12345 .
````

#### 1.2.8 Use URIs, blank nodes, and literals appropriately

The smallest unit of RDF is the subject-predicate-object triad (RDF triple),

- URI or blank node for the subject
- The predicate has a URI
- Object can be a URI, blank node, or literal

can be used. Each triple represents a subject and object connected by a relationship indicated by the predicate. Here,

- When it is appropriate for a URI to uniquely identify the resource (thing or thing) it represents globally in the world of the Web
- Blank nodes are simply for grouping and associating specific RDF triples together, or when global identification is not required.

for the protein. As an example, Q6GZX3 is used in UniProt because the protein identified by Q6GZX3 must be globally identified,
<http://purl.uniprot.org/uniprot/Q6GZX3>
is assigned to the URI.

On the other hand, numerical data such as strings and observed values are represented by literals because they represent values themselves, not identifiers (i.e., IDs). In this case, the meaning of the value (data semantics) can be described more clearly by attaching a data type such as unit to the value. In string literals, language tags can be used to specify the language, such as "protein"@en or "protein"@en, to indicate English or Japanese, and numeric literals, such as To indicate that the numeric literal `123' is an integer, a data type URI can be added, such as `"123"^^xsd:integer` (where `xsd:integer` is the same as <http://www.w3.org/2001/XMLSchema#integer>'s [ QName](https://en.wikipedia.org/wiki/QName) notation). In addition, please refer to the "How to Describe Values with Units" section of these guidelines for information on how to write literals specifying numeric units, etc.

#### 1.2.9 Do not use URIs beginning with https.

Today, HTTPS is being used on many websites. While this is a necessary effort to achieve a secure World Wide Web, there is a problem with using URIs that begin with https when describing RDF. For example, a URI that points to UniProt's Q6GZX3 could be <http://identifiers.org/uniprot/Q6GZX3> for some RDF data and <https://identifiers.org/> for other RDF data. uniprot/Q6GZX3>, even though they both describe the same resource, they are different URIs in the RDF data and will not be connected.
It is not practical to build RDF data or perform SPARQL searches while keeping track of whether a particular website is https or not, and whether the https or http scheme is used for individual URIs in the RDF data to be used. It is not practical to construct RDF data or perform SPARQL searches while knowing which scheme is being used. Therefore, it is recommended that URIs used in RDF data continue to start with 'http://'.


### 1.3 Guidelines for using and building ontologies

#### 1.3.1 Reuse existing ontologies

When describing information in RDF, it is often a non-trivial and difficult question as to which ontology to use for subject classes, predicate properties, and so on. It is recommended to use at least one of the following widely used ontologies or vocabularies, if appropriate.

##### General Vocabulary List

| Vocabulary | Namespace | Reference Links
|-----|-----|--------------|
| RDF 1.0 | http://www.w3.org/1999/02/22-rdf-syntax-ns# | [Concepts and Abstract Syntax [English](http://www.w3.org/TR/2004/REC-rdf-concepts-20040210/) [Japanese](http ://www.asahi-net.or.jp/~ax2s-kmtn/internet/rdf/rdf-concepts.html)]], [Introduction to RDF [English](http://www.w3.org/TR/2004/REC-rdf-primer-20040210/) [ Japanese](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/rdf/rdf-primer.html)], [RDF/XML Syntax Specification [English](http://www.w3.org/TR/2004/REC-rdf-) syntax-grammar-20040210/) [Japanese](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/rdf/rdf-syntax-grammar.html)]
|RDF 1.1 | http://www.w3.org/1999/02/22-rdf-syntax-ns# | [Concepts and Abstract Syntax [English](http://www.w3.org/TR/2014/REC-rdf11-concepts-20140225/)], [Introduction to RDF [English](http://www.w3.org/TR/2014/NOTE-rdf11-primer-20140225/)], [RDF/XML Syntax Specification [English](http://www.w3.org/TR/2014/REC-rdf-syntax-grammar- 20140225/)], [Turtle Syntax Specification [http://www.w3.org/TR/2014/REC-turtle-20140225/]] |
| RDFS 1.0 | http://www.w3.org/2000/01/rdf-schema# | [Specification [English](http://www.w3.org/TR/2004/REC-rdf-schema-20040210/) [Japanese](http://www.asahi-) net.or.jp/~ax2s-kmtn/internet/rdf/rdf-schema.html)]
| RDFS 1.1 | http://www.w3.org/2000/01/rdf-schema# | [Specification [English](http://www.w3.org/TR/2014/REC-rdf-schema-20140225/)]
| OWL 1 | http://www.w3.org/2002/07/owl | [Overview [English](http://www.w3.org/TR/2004/REC-owl-features-20040210/) [Japanese](http://www.asahi-net.or.jp) /~ax2s-kmtn/internet/rec-owl-features-20040210.html)]
| OWL 2 | http://www.w3.org/2002/07/owl | [Overview [English](http://www.w3.org/TR/2012/REC-owl2-overview-20121211/)
| DC | http://purl.org/dc/elements/1.1/ | [Specification [English](http://dublincore.org/documents/dces/)
| DC terms | http://purl.org/dc/terms/ | [Specifications [English](http://dublincore.org/documents/dcmi-terms/)]
| SKOS | http://www.w3.org/2004/02/skos/core | [Overview [English](http://www.w3.org/TR/skos-reference/) [Japanese](http://www.asahi-net.or.jp/~ax2s-kmtn) /internet/skos/REC-skos-reference-20090818.html)], [Introduction to SKOS [English](http://www.w3.org/TR/2009/NOTE-skos-primer-20090818/) [Japanese](http:// www.asahi-net.or.jp/~ax2s-kmtn/internet/skos/note-skos-primer-20090818.html)]]
| FOAF | http://xmlns.com/foaf/0.1/ | [Specifications [English](http://xmlns.com/foaf/spec/)]
| VoID | http://rdfs.org/ns/void# |[Specifications [English](http://www.w3.org/TR/void/)], [Overview [English](http://semanticweb.org/wiki/VoID)]
| UO | http://purl.obolibrary.org/obo/ | [Home](http://code.google.com/p/unit-ontology/), [BioPortal](http://bioportal.bioontology.org/) ontologies/UO)
| QUDT 1.1 | http://qudt.org/1.1/vocab/unit/ | [Home](http://www.linkedmodel.org/catalog/qudt/1.1/index.html), [BioPortal](http://) bioportal.bioontology.org/ontologies/QUDT)
| QUDT 2.0 | http://qudt.org/2.0/schema/qudt/ | [Home](http://qudt.org/doc/2017/DOC_SCHEMA-QUDT-v2.0.html)
| PROV-O | http://www.w3.org/ns/prov# | [Specifications [English](http://www.w3.org/TR/prov-o/)], [BioPortal](http://bioportal.bioontology.org/ontologies/) PROVO) |
| PAV | http://purl.org/pav/ |[Specifications [English](http://www.essepuntato.it/lode/http://purl.org/pav/2.0/)], [BioPortal](http://bioportal. bioontology.org/ontologies/PAV) |
| XSD | http://www.w3.org/2001/XMLSchema# ||
| DCAT | http://www.w3.org/ns/dcat# | [Specifications [English](https://www.w3.org/TR/vocab-dcat/), [Japanese](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/) rdf/REC-vocab-dcat-20140116.html)]
| BIBO | http://purl.org/ontology/bibo/ | [Home](http://bibliontology.com/), [GitHub](https://github.com/structureddynamics/) Bibliographic-Ontology-BIBO)
| Event |<http://purl.org/NET/c4dm/event.owl#> | [Home](http://motools.sourceforge.net/event/event.html)
| GEO | http://www.w3.org/2003/01/geo/wgs84_pos# | [Home](https://www.w3.org/2003/01/geo/)|

http://www.linkedmodel.org/catalog/qudt/1.1/index.html

In addition, [Linked Open Vocabularies
(LOV)](http://lov.okfn.org/dataset/lov/) to find widely used vocabulary.

In addition to the ontologies and vocabularies dealing with general concepts as described above, there are other ontologies that are easy to use when describing life science information.
In particular, [BioPortal](http://bioportal.bioontology.org/) allows you to search for appropriate ontologies and vocabularies in the life sciences.

##### Domain Ontology for Life Science Information


  | Abbreviations | Ontology Names | Namespaces | Reference Links
  |------|-------------|---------|----------|
  |GO | Gene Ontology | http://purl.obolibrary.org/obo/ | [Home](http://geneontology.org/), [BioPortal](http://bioportal.bioontology.org/) ontologies/GO) |
  |PRO | Protein Ontology | http://purl.obolibrary.org/obo/ | [Home](http://pir.georgetown.edu/pro/), [BioPortal](http://bioportal. bioontology.org/ontologies/PR)
  |SO | Sequence Types and Features Ontology | http://purl.obolibrary.org/obo/ | [Home](http://www.sequenceontology.org/), [BioPortal](http://) bioportal.bioontology.org/ontologies/SO)
  |FALDO | Feature Annotation Location Description Ontology | http://biohackathon.org/resource/faldo# | [GitHub](https://github.com/) JervenBolleman/FALDO) |
  |PO | Plant Ontology | http://purl.obolibrary.org/obo/ | [Home](http://plantontology.org/), [BioPortal](http://bioportal.bioontology.org/) ontologies/PO)
  |Taxonomy | Taxnomy Ontology | http://ddbj.nig.ac.jp/ontologies/taxonomy/ | [BioPortal](http://bioportal.bioontology.org/ontologies/) NCBITAXON), [DDBJ](http://tga.nig.ac.jp/ontologies/) |
  |Nucleotide | INSDC Nucleotide Sequence Entry Ontology | http://ddbj.nig.ac.jp/ontologies/nucleotide/ | [DDBJ](http://tga.nig.ac.jp/ ontologies/) |
  |MeSH | Medical Subject Headings | | [BioPortal](http://bioportal.bioontology.org/ontologies/MESH)
  |CL | Cell Ontology | http://purl.obolibrary.org/obo/ | [BioPortal](http://bioportal.bioontology.org/ontologies/CL)
  |FMA | Foundational Model of Anatomy | http://purl.org/sig/ont/fma/ | [Home](http://sig.biostr.washington.edu/projects/fm/), [BioPortal]( http://bioportal.bioontology.org/ontologies/FMA) |
  |UBERON | Uber Anatomy Ontology | http://purl.obolibrary.org/obo/ | [Home](http://uberon.org), [BioPortal](http://bioportal.bioontology.org) /ontologies/UBERON) |
  |SNOMED-CT | Systematized Nomenclature of Medicine - Clinical Terms | http://purl.bioontology.org/ontology/SNOMEDCT/ | [BioPortal](http:// bioportal.bioontology.org/ontologies/SNOMEDCT) |
  |SIO | Semanticscience Integrated Ontology | http://semanticscience.org/resource/ | [Home](http://semanticscience.org/), [BioPortal](http ://bioportal.bioontology.org/ontologies/SIO)
  |EFO | Experimental Factor Ontology | http://www.ebi.ac.uk/efo/ | [Home](http://www.ebi.ac.uk/efo), [BioPortal](http://bioportal. bioontology.org/ontologies/EFO)
  |ECO | Evidence Ontology | http://purl.obolibrary.org/obo | [Home](http://code.google.com/p/evidenceontology/), [BioPortal](http://) bioportal.bioontology.org/ontologies/ECO)
  | EDAM | EDAM bioinformatics operations, data types, formats, identifiers and topics | <http://edamontology.org/> | [Home](http:// edamontology.org/), [BioPortal](http://bioportal.bioontology.org/ontologies/EDAM)
  |CMO | Clinical Measurement Ontology | http://purl.obolibrary.org/obo/ | [Home](http://phenoonto.sourceforge.net/), [BioPortal](http://) bioportal.bioontology.org/ontologies/CMO)
  | MMO | Measurement Method Ontology | http://purl.obolibrary.org/obo/ | [Home](http://phenoonto.sourceforge.net/), [BioPortal](http://purl. bioontology.org/ontology/MMO)
  | XCO | Experimental Conditions Ontology | http://purl.obolibrary.org/obo/ | [Home](http://phenoonto.sourceforge.net/), [BioPortal](http://) bioportal.bioontology.org/ontologies/XCO)
  |OrthO | Ortholog Ontology | http://purl.jp/bio/11/orth# | [Home](http://mbgd.genome.ad.jp/ontology/), [BioPortal](http://bioportal. bioontology.org/ontologies/ORTHO)
  |PIERO | PIERO Enzyme Reaction Ontology | |[Home](http://reactionontology.org/)|
  |GlycoRDF | Glycan Ontology| http://purl.jp/bio/12/glyco/glycan# | [Home](https://github.com/ReneRanzinger/GlycoRDF), [BioPortal](http://) bioportal.bioontology.org/ontologies/GLYCORDF)
  |MONDO | Monarch Disease Ontology| http://purl.obolibrary.org/obo/ | [Home](http://www.obofoundry.org/ontology/mondo.html), [BioPortal]( https://bioportal.bioontology.org/ontologies/MONDO) |
  |HPO | The Human Phenotype Ontology| http://purl.obolibrary.org/obo/ | [Home](https://hpo.jax.org/), [BioPortal](https://bioportal. bioontology.org/ontologies/HP)
  |HCO | The Human Chromosome Ontology| http://identifiers.org/hco/ | [github](https://github.com/med2rdf/hco)


In addition, vocabularies related to language names include [ISO 639-1] (http://id.loc.gov/vocabulary/iso639-1.html) provided by the U.S. Library of Congress and [ISO
639-2](http://id.loc.gov/vocabulary/iso639-2.html) is available.

Note that when using existing ontology or vocabulary URIs in SPARQL or Turtle, the namespace is often denoted by a short form. For example, the URI `rdf:type` above is a short form of `<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>`. This abbreviated form is written according to Turtle's prefixed name notation, and any abbreviated name can be used as long as the correspondence between the abbreviated name and the actual notation is declared without conflict within a single dataset. However, there are widely used short names for major ontologies, and using these short names improves human readability. You can use [prefix.cc](http://prefix.cc/) as a service to find common short names.

#### 1.3.2 Build a new ontology

Often, ontologies are needed when converting data and knowledge to RDF. Typical examples are,

- To specify the class to which the resource belongs
- If you are describing some information as a literal, but it can be conceptualized
- If a predicate is needed that describes the relationship between the subject and object resources

Such as.
From the standpoint of data integration, it is preferable to use an existing ontology, but if no suitable one can be found among the existing ones, a new one must be constructed. In the Semantic Web, ontologies are generally defined using an ontology description language called OWL, which itself is also written in RDF. It is more efficient to use a dedicated ontology editor. The following editors are available to describe the OWL ontology.

- [Protege](http://protege.stanford.edu/)
    Open source ontology editor
- [WebProtege](http://webprotege.stanford.edu/) Protege in the Cloud
- [TopBraid](http://www.topquadrant.com/tools/modeling-topbraid-composer-standard-edition/)
    Commercial ontology editor

#### 1.3.3 Define DOMAIN and RANGE for the property

Properties (RDF predicates; predicate) can be defined as `rdfs:domain` and `rdfs:range`. This makes the meaning of the property more explicit.

````
Example.)  
core:classifiedWith rdfs:domain core:Protein .
core:classifiedWIth rdfs:range core:Concept .
````

The above example is for the `core:classifiedWith` property, which attaches functional annotation information such as GO to a protein entry in UniProt. From the string `classifiedWith`, a human being can infer a certain meaning, such as "it classifies the subject with something", but a machine cannot. A machine cannot.

For machines,

````
uniprot:P51028 core:classifiedWith go:2000044 .
````

And a statement that,

````
uniprot:p51028 ex:fugahoge go:2000044 .
````

The statement "`go:2000044`" also has no more meaning than that there is a relationship between `uniprot:P51028` and `go:2000044`.

The definition of domain and range in the ontology allows the machine to know that this property `core:classifiedWIth` cannot take any resource other than a `core:Protein` type as subject and cannot have any value other than `core:Concept` for information processing It can be used for information processing.

#### 1.3.4 Properly describe ontology classes and properties

When defining a new ontology class, it is important to provide an appropriate description so that users can understand what concepts the class represents.
As for properties, please describe them clearly and concisely using `skos:definition`, `rdfs:comment`, etc.

````
Example)  
core:Active_Site_Annotation rdfs:comment "Amino acid(s) involved in the activity of an enzyme."^^xsd:string;  
````

### 1.4 Guidelines for providing RDF

#### 1.4.1 Regular releases and version information

Currently, many databases are converted to RDF from databases that have already been built and provided in a different format (e.g., relational database). In such cases, there is a problem of synchronization between the data provided as a public service and the data converted to RDF. To avoid large differences between the primary data and the RDF-formatted data, it is desirable to convert the data to RDF on a regular basis. To this end, the conversion from existing formats to RDF should be automated as much as possible, and as much manual work as possible should be eliminated from the process; if the RDF conversion process involves a large amount of manual work, regular RDF conversion becomes significantly more difficult. Manual tasks (annotation, ontology mapping, etc.) should be included in the primary database building process. Of course, this does not apply if the primary database format is RDF.

It is also necessary to assign an appropriate version number when converting to RDF. For versioning, use the `pav:version`.
(for RDF data) and `owl:versionInfo` (for ontologies).

- [pav:version](http://purl.org/pav/version)
- [owl:versionInfo](http://www.w3.org/2002/07/owl#versionInfo)

#### 1.4.2 Attaching license information

Explicitly written licenses allow users to use RDF data with confidence. From the standpoint of ease of use of the data, it is recommended to use the appropriate [license](http://creativecommons.jp/licenses/) of [Creative Commons](http://creativecommons.jp/), etc. In addition, it is debatable whether or not an ontology is a copyrighted work in the first place, but assuming that it is a copyrighted work, and from the standpoint of ease of reuse of parts of the ontology, it is recommended to use [CC0](http://sciencecommons.org/resources/readingroom /ontology-copyright-licensing-considerations/) is said to be good.

````
Example)  
<a RDF data>` dcterms:license <http://creativecommons.org/licenses/by-sa/4.0/> .
<http://creativecommons.org/licenses/by-sa/4.0/> a <http://purl.org/dc/terms/LicenseDocument> .
````

To be fixed: add how it should be described in case of a proprietary license

#### 1.4.3 Provide schema diagram

Because RDF is a mechanism intended to increase machine readability, it is necessarily not suitable for human reading. Currently, however, it is necessary to know the structure of the data when querying RDF data with SPARQL, for example. In such cases, having a schema diagram of the RDF data is a great help in understanding the data structure.

- [Library for RDF Portal schema diagramming](https://drive.google.com/file/d/0B5fGKO5915HjdmxrWVJETmJqT0U/view?usp=sharingl)
    (After downloading the file, [draw io](http://draw.io/)
    (Select a file from "File>Open from>Device..." in the)

#### 1.4.4 Provide SPARQL samples

Provide a set of representative queries on how to use your data in
natural language along with the corresponding SPARQL query

#### 1.4.5 Take CORS measures when exposing SPARQL endpoints

Assuming a case where searches against a SPARQL endpoint are performed by JavaScript from a web application, the public web server that serves as the SPARQL endpoint should be accessible from anywhere, [CORS](https://en.wikipedia. org/wiki/Cross-origin_resource_sharing)
(Cross Origin Resource Sharing) measures are strongly recommended.

Specifically, configure Apache, Nginx, etc. to add the `Access-Control-Allow-Origin: *` header.

````
server { 
    (Omitted)  
  location /sparql { 
    add_header Access-Control-Allow-Origin *;  
    #proxy_pass http://localhost:8890/sparql ;
  } 
} 
````

in the appropriate location, such as <Location> or <VirtuoalHost> for Apache.

````
<IfModule mod_headers.c>  
  Header set Access-Control-Allow-Origin "*"  
</IfModule>  
````

Add a setting such as

Note that if the web server provided by the triple store itself is directly exposed as a SPARQL endpoint, triple store-specific configuration is required. For example, in the case of Virtuoso, please refer to [How to configure](http://virtuoso.openlinksw.com/dataspace/doc/dav/wiki/Main/VirtTipsAndT ricksCORsEnableSPARQLURLs) to change the settings.

#### 10 Rules for Publishing Linked Data in the Life Sciences

[BioHackathon 2014](http://2014.biohackathon.org/) but also Linked Data
Rules for publishing were discussed ([10 simple rules for publishing Linked Data for the Life Sciences](https://github.com/dbcls/bh14/wiki/Ten-simple-rules-for -publishing-Linked-Data-for-the-Life-Sciences). This discussion is being compiled into a paper, which is not yet complete and can be added to.

### 1.5 RDF model easy to use in life sciences

In the following, we accumulate and share best practices in RDF-ization of life science databases as a supplement to the Guidelines for Database RDF-ization.

#### How to describe measurement items and measurements

##### How to write values with units

- How to use blank nodes (recommended)

As written in the W3C's introduction to RDF ([more on structured values: rdf:value](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/rdf/rdf-primer.html#rdfvalue))
When writing values with units, blank nodes can be used to specify values and units as shown below.

````
exproduct:item10245 exterms:weight [ 
  rdf:value "2.4"^^xsd:decimal .
  exterms:units exunits:kilograms .
] .
````

The above example is from [more on structured value: rdf:value](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/rdf/rdf-primer.html#rdfvalue)
The following is an excerpt from the following table. However, it does not say what exactly to use as properties that point to blank nodes (exterms:weight) or units (exterms:units) or classes that represent units (exunits:kilograms).

For units, we recommend using Units of Measurement Ontology (UO: [BioPortal](http://bioportal.bioontology.org/ontologies/UO), [Home](https://code.google.com/p/ If you cannot find a suitable unit in UO, we recommend using Quantities, Units, Dimensions and Data Types Ontologies (QUDT: [BioPortal](http://bioportal. bioontology.org/ontologies/QUDT),.
[Home](http://qudt.org/)).

The property that points to the unit is [`sio:SIO_000221`](http://semanticscience.org/resource/SIO_000221)
(sio:has-unit) is recommended. Also, [`sio:SIO_000300`](http://semanticscience.org/resource/SIO_000300) as a property that points to a number.
(sio:has-value) is recommended.

The property that points to a blank node is [`sio:SIO_000216`](http://semanticscience.org/resource/has-unit.rdf)
(sio:has-measurement-value). Multiple numerical values with units can be stored in the same `sio:has-measurement-value`.
Pointing at a property does not allow specifying a specific numerical value, but can be resolved by properly defining a blank node (by making the blank node an instance of the appropriate ontology class), as shown below.

````
ex:m1 sio:SIO_000216 [ 
  rdf:type cmo:CMO_0000209 ; 
  sio:SIO_000300 21.5 ;
  sio:SIO_000221 uo:UO_0000309`  
] .

# CMO_0000209: Blood fibrinogen level defined by [Clinical Measurement Ontology](http://www.ontobee.org/browser/index.php?o=CMO)  
# UO_0000309: Milligram per square meter defined by [Units of Measurement Onotlogy](http://bioportal.bioontology.org/ontologies/UO) .
# SIO_000216 is has-measurement-value  
# SIO_000221 is has-unit  
````

- How to use unit type specification

A URI representing the unit can also be specified as the value type.

````
ex:e1 sio:SIO_000216 ex:m1 .
ex:m1 rdf:type cmo:CMO_0000209 ; 
      sio:SIO_000300 21.5^^obo:UO_0000309 .
````

#### 1.5.2 How to describe gene and protein sequence coordinate information

When describing coordinate information of sequences of genes, proteins, etc., [FALDO](http://biohackathon.org/resource/faldo): Feature Annotation Location Description Ontology [GitHub] ( https://github.com/JervenBolleman/FALDO) is recommended. FALDO is used in UniProt, Ensembl, DDBJ, and other RDFs. For usage, see [README](https://github.com/JervenBolleman/FALDO/blob/master/README.md) and [FALDO article](https://jbiomedsem.biomedcentral.com/ articles/10.1186/s13326-016-0067-z).
We also recommend the use of HCO (Human Chromosome Ontology) when describing reference sequences in FALDO.

#### 1.5.3 How to write samples

Often you will be describing a sample in a life science database. If the sample is registered in BioSamples RDF, please refer to its URI. For reference, EBI BioSamples plans to mechanically convert user-submitted sample features to RDF using the following description Given a key/value pair,

````
key value
````

The conversion is as follows

````
ex:sample 
  ex:hasCharacteristics [
     ex:propertyType a:mappledOntology ;
     ex:propertyValue "Original description for a key";.
  ] , [
  ex:propertyType a:mappledOntology ;
  ex:propertyValue "Original description for a value";.

  ] .

````

Also, the sample URI should be defined as an instance of sio:SIO_001050 (sio:sample).

````
ex:e1 rdf:type sio:SIO_001050 .
````


* * *

### Reference URL

#### Tools to assist in RDF conversion

This section introduces a set of tools that will be useful in the actual conversion to RDF according to the guidelines.

- [OpenRefine with RDF extension](http://refine.deri.ie/)
- [TogoDB](http://togodb.org/)
- [D2RQ](http://d2rq.org/)
- [-ontop-](http://ontop.inf.unibz.it/)
- [Protege](http://protege.stanford.edu/)
- [WebProtege](http://webprotege.stanford.edu/)
- [TopBraid Composer Standard
    Edition](http://www.topquadrant.com/tools/modeling-topbraid-composer-standard-edition/)
- [TopBraid Composer Maestro
    Edition](http://www.topquadrant.com/tools/IDE-topbraid-composer-maestro-edition/)
- [Silk](http://silk-framework.com/)

* * *

#### A helpful site for RDF conversion

- [HCLS Dataset
    descriptions](http://htmlpreview.github.io/?https://github.com/joejimbo/HCLSDatasetDescriptions/blob/master/Overview.html)
- [LOD Diary](http://www.infocom.co.jp/das/loddiary/linked_data/)
- [Linked Data: Evolving the Web into a Global Data
    Space](http://linkeddatabook.com/editions/1.0/)

(Japanese translation is available at [Linked data :.
A mechanism to make the web a global data space](http://ci.nii.ac.jp/ncid/BB11534438))

#### Page to freely post comments on the RDF Conversion Guidelines

- [comments on RDF conversion guidelines](http://wiki.lifesciencedb.jp/mw/RDFizingDatabaseGuidelineComments)

* * *

#### History

Version 1.5.0 as of "2015-12-07"\^\xsd:date Version 1.4.0 as of
"2015-08-11"\^\xsd:date

- The version number of this guideline is
    [Semantic versioning](http://semver.org/lang/ja/) is adopted.
    (2015-08-10)
    - The major version is the RDF/Ontology version created so far.
        Increment in the case of incompatible revisions that require a major update of the
    - Minor version increments when new information is added
    - Patch version is incremented when there is no semantic change, such as improved wording
    - Wiki for "Te ni ha" and typo fixes
        So rewrite without version change

2015-12-07 Version 1.5.0 Added suggestions for description of literature information. (skwsm)

2016-03-28 Version 1.6.0 Addition regarding deprecation of multiple ID definitions. (skwsm)

2016-10-21 Version 1.7.0
Added about creating schema diagram, added about giving SPARQL samples. (skwsm)


2016-10-21 Version 1.7.1
Change "Properly define classes" item to "Properly describe ontology classes and properties" (since "Properly define" reads like defining various class constraints, etc.). (skwsm)

2017-04-13 Version 1.8.0 "2.2.9 Do not include version in URI" added. (skwsm)

2017-05-15 Version 1.9.0 "2.2.9 Link to Literature Information". Modified (to do on properties to recommended) (skwsm)

2018-03-20 Version 1.9.1 "1.2.9 Do not use URIs beginning with https." Added the following. (skwsm)

2018-06-28 Version 1.9.2 1.2.9 Updated information on QUDT; added MONDO and HPO. (skwsm)

2018-06-28 Version 1.9.3 1.2.9 HCO added; 1.5.2 HCO use added. (skwsm)

