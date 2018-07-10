# RDF portal guidelines

### 1. Primary resources should be instances of an ontology class

Life science databases usually cover either one or a few subjects, and their content is organized by subject. For example, UniProt is a database of protein sequences, each represented as an instance of the up:Protein class in the UniProt RDF. As another example, ChEMBL is a database on the bioactivity of chemical compounds, and its entries are instances of classes such as cco:Assay, cco:Activity, or cco:Substance. URIs that represent such subjects (hereinafter, referred to as primary resources) should be defined as instances of an ontology class. This is useful for users to understand the semantics of the subjects. THis also helps to reduce the search spaces of SPARQL queries.  

### 2. Primary resources should have human-readable labels

Even though RDF is primarily intended to make data more machine-readable, providing natural-language labels for resources can be useful, especially when writing SPARQL queries or displaying application results. Although it is ideal that all resources have lables, at minimum all primary URIs must be labeled using the rdfs:label property. When multiple labels are needed, we recommend using the skos:altLabel property. For resources with multiple labels in different languages, each label should have a language tag so that labels in a specific language can be selected. On the other hand, language-independent literals, such as numerical values and database entry IDs, should not have language tags.

```An example
chebi:CHEBI_17234 rdfs:label "D-Glucose"@en ;  
                  rdfs:label "D-グルコース""@ja ;  
                  skos:altLabel "Dextrose"@en ;  
                  skos:altLabel ""ブドウ糖""@ja ;  
                  skos:prefLabel "D-Glucose"@en .
```

### 3. Primary resources should provide the local database IDs.

The local database ID is generally placed after the last slash at the end of the primary URI. However, when printing search results and showing them in an application’s user interface, users often find it easier to work with local database IDs rather than full URIs, and local IDs can also be convenient when writing SPARQL queries, for example. To enable this, the primary URI should have a dcterms:identifier property whose value is a literal containing this local ID.

### 4. Links to external resources should be provided in the specified format

With the Semantic Web, it is essential that both users and machines can explore the RDF-based Web of Data. Life science databases often provide abundant cross-links to external database entries, but there are usually several different URIs referring to the same database entry, and there are no general rules as to which URI to use when linking to external databases. Therefore, just converting such databases into RDF may not enhance the Web of Data, because these different URIs, even if they are redirected to the same Internet URI, are regarded as different RDF resources.

To address this problem, we require all external resources to be referred to using the URIs provided by identifiers.org and the rdfs:seeAlso property. This ensures that the same URI will always be used to refer to the same resource in different RDF datasets. One exception to this is that references to the primary resources in an RDF dataset officially released by the database provider must use the URIs defined in the dataset, because datasets do not usually use identifiers.org URIs to describe their own resources. In such cases, it is therefore obligatory to include redundant links to both the canonical and identifers.org URIs.


| Database | Primary class  |  URI Prefix |
|-----|------|-------------|
| UniProt| core:Protein | http://purl.uniprot.org/uniprot/ |
| Ensembl| obo:SO_0001217(protein_coding_gene) | http://rdf.ebi.ac.uk/resource/ensembl/ |
| ChEMBL| cco:Substance | http://rdf.ebi.ac.uk/resource/chembl/molecule/ |
| ExpressionAtlas| atlas:BaseLineExpressionValue | http://rdf.ebi.ac.uk/resource/expressionatlas/ |
|| atlas:DifferentialExpressionRatio | http://rdf.ebi.ac.uk/resource/expressionatlas/ |
| Reactome| biopax3:Pathway | http://identifiers.org/reactome/ |
| BioModels|  ||
| BioSamples| biosd-terms:Sample | http://rdf.ebi.ac.uk/resource/biosamples/sample |
| PubChem| compound | http://rdf.ncbi.nlm.nih.gov/pubchem/compound/ |
| | substance | http://rdf.ncbi.nlm.nih.gov/pubchem/substance |
| MESH | meshv:TopicalDescriptor | http://id.nlm.nih.gov/mesh/ |
| wwPDB| PDBo:datablock | http://rdf.wwpdb.org/pdb/1NH2 |

There are two other exceptions to this rule for external resources. References to articles or books should use the PubMed URI or DOI with the dcterms:references property, and images should use the foaf:depiction property.

### 5. Metadata should be provided

The dataset submitter should provide the following metadata: the dataset providers’ and creators’ names, the version, the date issued, the license, and the NBDC database classification tags. It is particularly important that license information is provided, so users can determine how the dataset can be used. This is also a condition for the dataset to be findable, accessible, interoperable, and reusable (FAIR). The RDF portal only accepts datasets provided with some type of open license. Currently, most datasets are available under the Creative Commons License.

### 6. Widely-used ontologies should be used where possible

Although the semantics of individual RDF datasets are left to their developers, we encourage the use of widely-used ontologies where possible. The DBCLS guidelines for RDFizing databases therefore list the ontologies we recommend.

### 7. The domain and range of each user-defined property should be explicitly defined

When converting a database to RDF, it may be necessary to define new properties, particularly to express relationships between concepts. When doing so, each property’s domain and range should be defined as explicitly as possible. This helps to make queries more efficient and create applications that build SPARQL queries automatically.

### 8. A schema diagram should be provided

When writing SPARQL queries for an RDF dataset, it is a great help to have a schema diagram available. Such diagrams should therefore be provided.

### 9. Sample queries should be provided

It is very helpful to see examples of typical queries when querying RDF datasets using SPARQL. At least one example query should therefore be provided.

### 10. DNA and protein sequence coordinates should be described using FALDO

Many life science databases provide structural and functional annotations to genome or protein sequences. The FALDO ontology should be used to specify the point in a sequence to be annotated. Since this is already used in various RDF datasets, such as UniProt, Ensembl, and DDBJ, using common sequence coordinates enables us to achieve highly interoperable annotations.

**Note**
It is planning to introduce a mandatory use of Human Chromosome Ontology ([HCO](https://github.com/med2rdf/hco)) as an object of the faldo:reference property.

### 11. Structured values should be used for values with units

Structured values should be used to describe numerical values with units by using the Semanticscience Integrated Ontology (SIO) giving at least an sio:SIO_000300 property (i.e., sio:has-value) for the value and an sio:SIO_000221 property (i.e., sio:has-unit) for the unit, as in the example below. Structured values should be typed using an appropriate ontology class, included as an sio:SIO_000216 property (i.e., sio:has-measurement-value). The Units of Measurement Ontology (UO) (http://bioportal.bioontology.org/ontologies/UO) should be used to express units where possible, but another ontology can be used for units not included in the UO.

```An example
ex:m1 sio:SIO_000216 [
  rdf:type		cmo:CMO_0000209;
  sio:SIO_000300	21.5;
  sio:SIO_000221	uo:UO_0000309
] .
```
