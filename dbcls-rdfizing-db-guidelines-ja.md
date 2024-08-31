# DBCLSデータベースRDF化ガイドライン

## はじめに 〜 Linked Data構想

[RDF](http://www.w3.org/TR/rdf11-concepts/)は、曖昧性が少なく、機械可読性の高いデータを記述する枠組みです。しかし、実際に記述したいデータをどのようにRDF化すれば、表現したい内容が記述できるのか、また、その後の利用の観点からより良いものになるのか、という指針は提供されていません。このことが、データベースをRDF化する際に、特に初心者にとって高いハードルとなっています。幸い、これまで [BioHackathon](http://biohackathon.org) や [SPARQLthon](http://wiki.lifesciencedb.jp/mw/SPARQLthon) などでも、様々な議論がなされ、知見も蓄積されてきました。そろそろこのコミュニティでデータをRDF化する際に指針となるようなガイドラインを作る時期にきたと思います。本ガイドラインが目指すのは、それを参照することで、RDF化作業の負担が減り、また他のデータと適切に統合して利用できるようなRDFを作成できるようにすることです。

基本精神として、Tim Berners-Leeによる、 [Linked Data構想](http://www.w3.org/DesignIssues/LinkedData.html)
に則ったデータ作成が推奨されます。Linked Data構想では、

>1.  Use URIs as names for things → モノやコトにURIを使って名前をつける
>2.  Use HTTP URIs so that people can look up those names. →
>    広く一般に普及しているソフトウェアでアクセスできる、http://
>    から始まるURIを名前に使用することでユーザがそれについて調べられるようにする（広く普及しているソフトウェアではアクセスで>きないURIもあるため）
>3.  When someone looks up a URI, provide useful information, using the
>    standards (RDF\*, SPARQL) →
>    URIにアクセスした時にRDFやSPARQLなどの標準に沿って有用な情報を提供する
>4.  Include links to other URIs. so that they can discover more things.
>    →
>    他のURIへのリンクを含めるようにすることで、さらなる情報を辿れるようにする

という4つの原則が提唱されています。

なお、RDFは[Resource Description
Framework](https://ja.wikipedia.org/wiki/Resource_Description_Framework)の略称で、URIは[Uniform
Resource Identifier](https://ja.wikipedia.org/wiki/Uniform_Resource_Identifier)の略称です。RDFは、ウェブにおいてURIで指し示される[リソース](https://ja.wikipedia.org/wiki/%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9_(WWW))の情報を表現するための一つの枠組み(規格)であり、特定のファイル形式を指すものではありません。実際、RDFを用いて情報を記述する形式にはRDF/XML, N-Triples, Turtle, JSON-LDなど複数ありますが、[Turtle](http://www.w3.org/TR/turtle/)形式は機械だけでなく人から見ても可読性が高く、SPARQLクエリの表記方法と共通点が多いことから、本ガイドラインではTurtle形式に従った表記をします。

## 1 データベースコンテンツをRDF化する際のガイドライン

以下、これまで蓄積してきた知見から、データベースをRDF化する際に推奨される指針を挙げます。

### 1.1 URIを設計する際のガイドライン

#### 1.1.1 永続性の高いURIを利用する

永続性の高いURIを使うという指針は、Tim Berners-Lee
による「[クールなURIは変わらない](http://www.w3.org/Provider/Style/URI)
[(日本語訳)](http://www.kanzaki.com/docs/Style/URI)」という宣言から始まっており、W3Cからも「セマンティックウェブのためのクールなURI
[(原文)](http://www.w3.org/TR/cooluris/),
[(日本語訳)](http://standards.mitsue.co.jp/resources/w3c/TR/2008/NOTE-cooluris-20080331/)」
という文書が公開されています。

RDFデータは、 URIリソースに関する情報を記述します。そしてLinked
Dataの原則2で、http://
から始まるURIを利用することを推奨し、原則3で、URIリソースに関する情報を、当該URIにアクセスすることでRDFなどの標準を用いて得られることを推奨しています。しかし、もともとアクセスできたURIが、何らかの理由で違うURIに変わってしまう場合や消滅してしまった場合は、RDFデータの価値は著しく下がります。例えば、生命科学系データベースの多くが研究機関で作られていますが、組織の改組、統合、廃止等の理由で、組織のURIが変更になることは珍しくありません。このため、組織のドメイン名を用いたURIでRDFデータを記述していると、運良くデータベースが新しい組織で運営されることになっても、RDFデータからはそのURIにアクセスできなくなります。このような事態を回避するため、下記の2つの方法を検討することが推奨されます。

-   独自ドメインを取得する

よくあるケースとして、uniprot.orgのように「データベース名.org」のような独自ドメインを取得する方法があります。これにより、データベースを運用する実体が変わっても、同じURIを持続して利用することができます。欠点は、独自ドメインを維持するのに費用がかかることや、誰がそのドメインを維持するのか、といった別の永続性問題が発生することです。

-   PURLを利用する

他によく用いられるケースとして、永続的なURLである[Persistent URL(PURL)](https://en.wikipedia.org/wiki/Persistent_uniform_resource_locator)を利用してデータを記述する方法があります。RDFはPURLのURIを用いて記述し、PURLからデータを運用する実体のURLに転送(リダイレクト)することで、実体が移転した場合にもRDFデータの変更を伴わずに同じURIを持続して利用することができます。NBDCでは、研究機関のデータベース運用のために
[purl.jp](http://purl.jp/) サービスを提供しています。PURLサービスで転送を管理することにより、常にそのときアクセスできる最新のURIを指し示すことができます。

#### 1.1.2 リソースを示すURIはそれを識別するためのIDをURIの末尾に記述する

永続性の高いドメイン部分に続く、URIのパス部分については、モノやコトを表す記述対象のリソースを識別するIDがURIの最後に来るようにし、その直前にスラッシュ(/)を用いることが推奨されます。例えばUniProtではQ6GZX3というIDで識別されるタンパク質について <http://purl.uniprot.org/uniprot/Q6GZX3> というURIを割り当てています。このようにURIを構成することで、Linked Dataの原則3で示される、「URIにアクセスした時にRDFやSPARQLなどの標準に沿って有用な情報を提供する」サービスの構築が技術的に実現し易くなります。

なお、オントロジーのURIでは、そのオントロジーを構成する概念を示すIDごとに個別のページを作る場合は、上記と同じようにIDの直前にスラッシュ(/)を、オントロジー全体が１つのページで提供される場合は、IDの直前をスラッシュの代わりにハッシュ(\#)としておくのが良いでしょう。例として、UniProtのオントロジー `http://purl.uniprot.org/core/` ではProteinなどの概念が個別のページで提供されているため、`http://purl.uniprot.org/core/Protein` のようにスラッシュ(/)が使用されています。一方、配列位置情報のオントロジー[FALDO](http://biohackathon.org/resource/faldo) では、Positionなどの概念が同じページで提供されているため、URIフラグメントの記法を用いて`http://biohackathon.org/resource/faldo#Position`のようにハッシュ(\#)が使用されています。

#### 1.1.3 バージョン情報を含むIDの扱い

NCBI Gene や NCBI Protein等では、バージョン番号を含むIDと含まないIDが提供されています（例: https://www.ncbi.nlm.nih.gov/protein/NP_003024.1 と https://www.ncbi.nlm.nih.gov/protein/NP_003024 ）。このようなIDを利用する場合、どちらのIDを利用するべきか、ということが問題になります。他のデータセットとのリンクを重視する場合は、バージョン番号を含まないIDを利用する方が、データセット間のリンクは維持されやすいでしょう。一方で、特定のバージョンのIDに関する情報を記述したい場合であれば、バージョン番号を含むIDを利用するべきです。すなわち、状況に応じてより適切なIDを利用する必要があります。ただし、データセット間のリンクを維持するという観点から、バージョン番号付きのIDを利用する場合であっても、バージョン番号のないIDへのリンクも提供する方が望ましいといえます。
なお、最新のバージョンを知りたい場合は、[`TogoWS`](http://togows.org) を利用することで、対応しているデータベースに関しては、次のような方法で、各データベースの最新バージョンを取得することができます。

```
http://togows.org/entry/ncbi-protein/145579718/version
```

### 1.2 RDFを作成する際のガイドライン

#### 1.2.1 URIリソースはオントロジーのクラスのインスタンスとして定義する

URIで示されるリソースが何を意味しているのか、ということを明確かつ簡潔に表現するのに、オントロジーのクラスが`rdf:type`で指定されていることは重要です。特に、主たるリソースには、オントロジーに基づいてクラスを指定することが推奨されます。

```
uniprot:Q6GZX3 rdf:type core:Protein .
```

上の例では、UniProtのQ6GZX3エントリーが、[`core:Protein`](http://purl.uniprot.org/core/Protein)であることが分かります。リソースの指し示すものが明確になる上に、SPARQL検索の際にも、`rdf:type`を指定することで効率的なデータ探索を行うことができます。オントロジーの定義がURIにアクセスすることで参照できると理想的ですが、そうなっていることは必須ではありません。ただし、自然言語では利用される文脈が異なると違った意味を持つことが普通に起こります。URIの参照先にデータ作成者が意図した定義を記述することは、データ利用者の誤解を避ける上で有効な手段になります。これは科学データを記述する場合には、大きなメリットといえます。

ここで問題になるのは、「では、自らのリソースに、どのクラスをタイプ指定すればいいのか？」ということです。既存のオントロジーに適当なクラスが見つかればそれを利用すればいいですが、見つからない場合も多いと思われます。その場合は、自らのデータ用に、簡単なオントロジーを作成する必要があります。UniProtやEBI RDFも、必要なクラスをそれぞれ定義してデータとともに提供しています。

なお、RDF において、

-   `rdfs:Class`を`rdf:type`の目的語とするとき、その主語はクラスであることを
```
ex:Myclass rdf:type rdfs:Class .
```
-   あるクラスのURIを`rdf:type`の目的語とするとき、その主語はインスタンスであることを
```
ex:111 rdf:type ex:Myclass .
```

それぞれ表します（ある主語が、明示的に`rdfs:Class`をタイプ指定されていなくても、`rdfs:subClassOf`によって、あるクラスのサブクラスとして定義されている場合も、その主語はクラスになります）。しかし、あるURIをクラスであると同時にインスタンスとすることはデータの意味的な処理が複雑になるので、避けられるのであれば避けて下さい。

```
#非推奨  
ex:111 rdf:type rdfs:Class # ← ex:111 はクラスでもあり  
ex:111 rdf:type ex:Myclass # ← インスタンスでもある、ことになる  
```

#### 1.2.2 URIリソースにラベルをつける

URIで示されるリソースが何なのか、ということを人間が容易に理解するためには、ラベルが`rdfs:label`により自然言語で記述されていることは有用です。特に、主たるリソースには簡潔なラベルを記述することが推奨されます。このことは、機械可読性を高めることとは関係ありませんが、人間可読性を高めるので、アプリケーションをつくる際や、SPARQL検索の結果を読みやすく表示する際に便利になります。

```
uniprot:P51028 rdfs:label `“`wnt8a`”`@en .`
```

また、この例の様に言語タグを付けることは、特に多言語化の際に有用です。英語ネイティブ以外であれば、母国語でもラベルを付けたい場合やコメントを追記したい場合があります。言語タグを付けることで、明示的に複数の言語で記述することができますし、アプリケーションで利用する際に英語ラベルだけを利用する、というようなことも簡単に行えます。

```
mpo:MPO_03001 rdfs:label "Thermophilic"@en ,"高熱性"@ja .
```

ある言語によるラベルは一つだけにしておいたほうが、利用する際に悩まないで済むので便利だと思います。異なるラベルを付けたい場合には、`rdfs:label`の代わりに`skos:altLabel`が利用できます。また、`rdfs:label`や`skos:altLabel`などによって複数のラベルが存在する場合に、そのうちの一つを代表ラベルとして明示したい場合は、`skos:prefLabel`が利用できます。

```
chebi:CHEBI_17234 rdfs:label "D-Glucose"@en ;  
                  rdfs:label "D-グルコース""@ja ;  
                  skos:altLabel "Dextrose"@en ;  
                  skos:altLabel ""ブドウ糖""@ja ;  
                  skos:prefLabel "D-Glucose"@en .  
```

#### 1.2.3 URIリソースにIDラベルをつける

URIはそれ自体がグローバルなIDとして機能しています。しかし、URIは記号的ですし文字列として長いので、SPARQL結果を表示する際に人間が見るのには向いていません。URIの末尾にデータベース固有の(ローカルな)IDが含まれているような場合は、URIの最後の/以下を切り出すことで文字列としてのIDを得ることができますが、このためにはSPARQLで文字列処理を適用するなど余計な手間が発生します。そこで、主なリソースのID文字列については、`dcterms:identifier`で記述することが推奨されます。

```
uniprot:P51028 dcterms:identifier "P51028" .
```

なお、IDで識別されるあるリソースに対して、`dcterms:identifier`プロパティを用いて複数のIDを宣言することはやらないようにして下さい。

```
#以下は非推奨。  
pdb:2RH1 dcterms:identifier "2RH1" .`  
pdb:2RH1 dcterms:identifier "2rh1" .`  
```

IDの識別子の本来的な意味から、異なるIDを与えることはできませんが、なんらかの事情で複数のidentifierを宣言したい場合（例えば、上記の例のように大文字、小文字両方のケースで、問い合わせに対応した場合等）は、別のプロパティを利用して宣言して下さい。

```
#以下は問題無し。  
pdb:2RH1 dcterms:identifier "2RH1" .  
pdb:2RH1 skos:altLabel "2rh1" .  
```

#### 1.2.4 他のデータセットへのリンクをつける

RDFでは、Linked Dataの原則4で示されるように、外部のリソースに対する参照リンクを貼ることで、データのウェブ(Linked Data)が実現されます。クロスリファレンス先がデータベースエントリの場合は、参照するDBエントリーが実際に閲覧できるURLへリンクを張ることが一般的です。これをオリジナルのサイトを尊重するという意味からpolite URLと呼んでいます。しかし、オリジナルのURLを利用することの問題点として、

-   リンクしたページのURLが変更されてしまっても、一度公開してしまったRDFへはその変更が自動的に適用されないこと
-   オリジナルのURLがcool URIではない（CGI引数をとって動的に生成されるページなどIDとしてふさわしくない）場合があること

等があります。また、同じ項目を参照するのに複数のURLが存在する場合があり（例えば[Taxonomy ID](http://info.identifiers.org/taxonomy/9606) などは、NCBI, EBI, UniProt, Bio2RDF等のサイトが異なるURLを提供していて、どれも広く利用されています）

-   個々のデータベースが同じモノを指すのに異なるURIを利用したRDFを提供すると、統合的な検索が出来なくなる

という問題もあります。

こういった問題点を避ける方法として、[<http://identifiers.org/>](http://identifiers.org/)のURIを利用することが考えられます。Identifiers.orgはPURLに似ていますが、データベースのURIに特化したサービスで、転送先が複数ある場合には選択することも可能なほか、データベースのメタデータをRDFで提供してくれるなどの利点があります。Identifiers.orgはライフサイエンスのコミュニティによって維持されており、参照したいデータベースが登録されていない場合もリクエストすることで追加することができます。

このため、クロスリファレンスを記述する場合には、

```
例)  
ex:111 rdfs:seeAlso <http://pfam.xfam.org/family/PF01590> .  
ex:111 rdfs:seeAlso <http://identifiers.org/pfam/PF01590> .  
```

のようにpolite URLとIdentifiers.orgのURIの両方に参照リンクを貼ることで、外部のリソースと繋がりやすいRDFを作成することができます。

NBDC RDF portalでは、RDFが公開されているデータベースについては、そのRDFで利用されているURIと、Identifiers.orgのURI（利用できる場合）の両方に対して、リンクすることを推奨しています。Identfiers.orgのURIについては、そのURIが所属するデータベースを表すクラスのインスタンスとなるように、NBDC RDF portal 側でトリプルを生成する予定です（2017.11現在）。

##### RDFが公開されている主要なデータベースと,外部からリンクする際に利用するリソースのURI prefix

| データベース名 | Class  |  URI Prefix |
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


#### 1.2.5 文献情報へのリンクをつける

記述している内容の根拠となる文献情報が分かっている場合には、積極的に文献情報へのリンクを付けて下さい。文献情報は、可能な限り[PubMed](http://pubmed.org/)または[DOI](http://doi.org/)のIDを使ってリンクしてください。これらのIDをURI化する際は下記のprefixを利用することを推奨します。

-   PubMed IDのURI <http://rdf.ncbi.nlm.nih.gov/pubmed/24495517>
-   DOI IDのURI <http://doi.org/10.1021/jo0349227>

PubMed IDや DOI IDが利用できない場合は、学術文献であれば [Bibliographic Ontology](http://bibliontology.com/) の `bibo:Article` のインスタンスとして記述して下さい。書籍であれば、`bibo:Book`も利用できます。文献に関する詳細な情報（雑誌名、巻数、ページ数、出版社等）についても、[Bibliographic Ontology](http://bibliontology.com/) [[GitHub](https://github.com/structureddynamics/Bibliographic-Ontology-BIBO)\]
の利用を推奨します。（参考） [CINIIでの利用方法](https://support.nii.ac.jp/ja/cia/api/a_rdf)

文献情報へリンクする際には、[`dcterms:references`](http://purl.org/dc/terms/references)
プロパティの利用を推奨します。

```
<a resource> dcterms:references pubmed:24495517 .
```

文献に関する詳細な情報については、[Bibliographic Ontology](http://bibliontology.com/) の利用を推奨します。 （参考）
[CINIIでの利用方法](https://support.nii.ac.jp/ja/cia/api/a_rdf)

```
@prefix dcterms: <http://purl.org/dc/terms/> .  
@prefix bibo: <http://purl.org/ontology/bibo/> .  
@prefix prism: <http://prismstandard.org/namespaces/1.2/basic/> .  

<a resource> dcterms:references [  
  a bibo:Article;  
  prism:publicationName "Nature science cell";  
  prism:volume "10";  
  prism:number "11";  
  prism:startingPage "123";  
  prism:endingPage "456";  
  dcterms:date "2015-12-08" ;  
  seeAlso <http://rdf.ncbi.nlm.nih.gov/pubmed/1234567>  
] .  
```

#### 1.2.6 データにメタ情報をつける

データをセマンティック・ウェブ化する利点の一つは、メタデータを必要なだけ付与できることです。RDFのデータセットにメタデータを付与することで、作成された日、作成した人、データソース、データのカテゴリ、ライセンス等を明示することができます。
RDFデータセットに関するメタデータを記述するための語彙として[VoID](http://www.w3.org/TR/void/)があります。
本語彙を利用してRDFデータセットに対するメタデータを効率的に構築するためのツールとして例えば[VoID Editor](http://voideditor.cs.man.ac.uk/)があります。

また、RDFで記述された1つ1つの文(statement;各トリプルのこと)についても、その文の来歴(provenance)を付与することができます。特に科学のデータの場合、各文がファクトを表明することが多いため、その文を記述するにいたった来歴は重要です。この来歴には、論文や本などに記載された情報、機械的な手法(BLASTによる相同性検索等)で得られた情報から、個人の推測まで、様々な可能性があり得ます。来歴が明示的に記述されていることで、(論文で言及された情報だけ使う等)各トリプルをどのように利用するべきか、ユーザが選択することができるようになります。

RDFで来歴情報を記述するための語彙は、いくつか提案されており（[DC terms](http://purl.org/dc/terms/)、[PROV-O](http://www.w3.org/TR/prov-o/)、[PAV](http://bioportal.bioontology.org/ontologies/PAV)）、また、来歴情報を記述するためのモデルもいくつか提案されています([RDF　Reification](https://goo.gl/RXvpBE)、[NanoPub](http://nanopub.org/wordpress/)、[OvoPub](http://arxiv.org/pdf/1305.6800.pdf)、[VoAG](http://linkedmodel.org/doc/voag/1.0/)等)。BioHackathon 2014 でも議論されました　 [Standardization of RDF data and development of tools/ontologies](http://goo.gl/u32WCo) (8ページ目)。

また、アノテーションなどで、その根拠のレベルを記述するために、Evidence Code Ontology を利用することもできます。

#### 1.2.7 画像へのリンク

-   主語URIが意味する実体を表す画像へのリンクを張る際には、[foaf:depiction](http://xmlns.com/foaf/spec/#term_depiction)
    を利用する。
```
an-assay-db:12345 foaf:depiction an-assay-db-image:12345.jpg .
```

-   逆に、画像ファイルのURIが主語で、その画像に描かれているもののリソースURIが目的語の場合、[foaf:depicts](http://xmlns.com/foaf/spec/#term_depicts)
    を利用する。
```
an-assay-db-image:12345.jpg foaf:depicts an-assay-db:12345 .
```

#### 1.2.8 URI、空白ノード、リテラルの使い分けを適切に行う

RDFはURI、空白ノード、リテラルの組み合わせで構成されますが、これからRDF化を行うデータをどのようにこれらに当てはめると、より有用なRDFになるか検討します。RDFの最小単位は主語、述語、目的語の三つ組み（RDFトリプル）であり、

-   主語にはURIもしくは空白ノード
-   述語にはURI
-   目的語にはURI、空白ノード、もしくはリテラル

が使えます。各トリプルは、主語と目的語が述語で示される関係性で結ばれることを表します。ここで、

-   URIはそれが示すリソース（モノやコト）をウェブの世界でグローバルに一意に識別することが適切である場合
-   空白ノードは単に特定のRDFトリプル同士をグループ化して結びつけるためなど、グローバルに識別することが必要ではない場合

に用います。例として、UniProtではQ6GZX3で識別されるタンパク質はグローバルに識別されることが必要ですから、
<http://purl.uniprot.org/uniprot/Q6GZX3>
というURIが割り当てられています。

一方、文字列や観測値などの数値データは識別子(IDのこと)ではなく値そのものを表しますので、リテラルで表現します。この際、値に対して単位などデータの型を付けることで、より明確に値の意味(データのセマンティクス)を記述することができます。文字列リテラルでは、言語タグを使うことにより“protein”@enや“タンパク質”@jaなど英語や日本語といった言語を明示することができますし、数値リテラルの123が整数であることを示すには`"123"^^xsd:integer`のようにデータ型URIをつけることができます(ここで`xsd:integer`は<http://www.w3.org/2001/XMLSchema#integer>の[QName](https://en.wikipedia.org/wiki/QName)表記です)。さらに、数値の単位などを指定したリテラルの記述方法については、本ガイドラインの「単位のついた値の記述方法」セクションを参考にしてください。

#### 1.2.9 httpsで始まるURIを利用しない。

現在、多くのウェブサイトにおいてHTTPS化が進んでいます。このことは、セキュアなWWWを実現するために必要な取り組みですが、一方で、RDFを記述する際に、httpsで始まるURIを用いることには問題があります。例えば、UniProtのQ6GZX3をさすURIとして、あるRDFデータでは、<http://identifiers.org/uniprot/Q6GZX3>　を用い、他のRDFデータでは、<https://identifiers.org/uniprot/Q6GZX3>　を用いてる場合、両者は同じリソースを記述しているにも関わらず、RDFデータの上では異なるURIであり、このままでは繋がらなくなります。
特定のウェブサイトがhttps化しているかどうか、また、利用するRDFデータにおいて、個々のURIにhttpsまたはhttpどちらのスキームが利用されているかなどを全てを把握しながら、RDFデータを構築したり、SPARQL検索を行ったりすることは、現実的ではありません。したがって、RDFデータで用いるURIは、これまで通り、'http://'で始まるURIを用いることを推奨します。


### 1.3 オントロジーを利用・構築する際のガイドライン

#### 1.3.1 既存のオントロジーを再利用する

情報をRDFとして記述する場合、主語のクラスや述語のプロパティ等に、どのオントロジーを利用すればいいのかということは、しばしば自明ではなく、難しい問題です。少なくとも、次にあげるような広く利用されているオントロジーや語彙に適当なものがある場合はそれを利用することが推奨されます。

##### 一般的な語彙の一覧

| 語彙 | 名前空間 | 参考リンク |
|-----|-----|--------------|
| RDF 1.0 | http://www.w3.org/1999/02/22-rdf-syntax-ns# | [概念および抽象構文 [英語](http://www.w3.org/TR/2004/REC-rdf-concepts-20040210/) [日本語](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/rdf/rdf-concepts.html)], [RDF入門 [英語](http://www.w3.org/TR/2004/REC-rdf-primer-20040210/) [日本語](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/rdf/rdf-primer.html)], [RDF/XML構文仕様 [英語](http://www.w3.org/TR/2004/REC-rdf-syntax-grammar-20040210/) [日本語](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/rdf/rdf-syntax-grammar.html)] |
|RDF 1.1 | http://www.w3.org/1999/02/22-rdf-syntax-ns# | [概念および抽象構文 [英語](http://www.w3.org/TR/2014/REC-rdf11-concepts-20140225/)], [RDF入門 [英語](http://www.w3.org/TR/2014/NOTE-rdf11-primer-20140225/)], [RDF/XML構文仕様 [英語](http://www.w3.org/TR/2014/REC-rdf-syntax-grammar-20140225/)], [Turtle構文仕様 [英語](http://www.w3.org/TR/2014/REC-turtle-20140225/)] |
| RDFS 1.0 | http://www.w3.org/2000/01/rdf-schema# | [仕様 [英語](http://www.w3.org/TR/2004/REC-rdf-schema-20040210/) [日本語](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/rdf/rdf-schema.html)] |
| RDFS 1.1 | http://www.w3.org/2000/01/rdf-schema# | [仕様 [英語](http://www.w3.org/TR/2014/REC-rdf-schema-20140225/)] |
| OWL 1 | http://www.w3.org/2002/07/owl | [概要 [英語](http://www.w3.org/TR/2004/REC-owl-features-20040210/) [日本語](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/rec-owl-features-20040210.html)] |
| OWL 2 | http://www.w3.org/2002/07/owl | [概要 [英語](http://www.w3.org/TR/2012/REC-owl2-overview-20121211/)] |
| DC | http://purl.org/dc/elements/1.1/ | [仕様 [英語](http://dublincore.org/documents/dces/) |
| DC terms | http://purl.org/dc/terms/ | [仕様 [英語](http://dublincore.org/documents/dcmi-terms/)] |
| SKOS | http://www.w3.org/2004/02/skos/core | [概要 [英語](http://www.w3.org/TR/skos-reference/) [日本語](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/skos/REC-skos-reference-20090818.html)], [SKOS入門 [英語](http://www.w3.org/TR/2009/NOTE-skos-primer-20090818/) [日本語](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/skos/note-skos-primer-20090818.html)] |
| FOAF | http://xmlns.com/foaf/0.1/ | [仕様 [英語](http://xmlns.com/foaf/spec/)] |
| VoID | http://rdfs.org/ns/void# |[仕様 [英語](http://www.w3.org/TR/void/)], [概要 [英語](http://semanticweb.org/wiki/VoID)] |
| UO | http://purl.obolibrary.org/obo/ | [Home](http://code.google.com/p/unit-ontology/), [BioPortal](http://bioportal.bioontology.org/ontologies/UO) |
| QUDT 1.1 | http://qudt.org/1.1/vocab/unit/ | [Home](http://www.linkedmodel.org/catalog/qudt/1.1/index.html), [BioPortal](http://bioportal.bioontology.org/ontologies/QUDT) |
| QUDT 2.0 | http://qudt.org/2.0/schema/qudt/ | [Home](http://qudt.org/doc/2017/DOC_SCHEMA-QUDT-v2.0.html) |
| PROV-O | http://www.w3.org/ns/prov# | [仕様 [英語](http://www.w3.org/TR/prov-o/)], [BioPortal](http://bioportal.bioontology.org/ontologies/PROVO) |
| PAV | http://purl.org/pav/ |[仕様 [英語](http://www.essepuntato.it/lode/http://purl.org/pav/2.0/)], [BioPortal](http://bioportal.bioontology.org/ontologies/PAV) |
| XSD | http://www.w3.org/2001/XMLSchema# ||
| DCAT | http://www.w3.org/ns/dcat# | [仕様 [英語](https://www.w3.org/TR/vocab-dcat/), [日本語](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/rdf/REC-vocab-dcat-20140116.html)] |
| BIBO | http://purl.org/ontology/bibo/ | [Home](http://bibliontology.com/), [GitHub](https://github.com/structureddynamics/Bibliographic-Ontology-BIBO) |
| Event |<http://purl.org/NET/c4dm/event.owl#> | [Home](http://motools.sourceforge.net/event/event.html) |
| GEO | http://www.w3.org/2003/01/geo/wgs84_pos# | [Home](https://www.w3.org/2003/01/geo/)|

http://www.linkedmodel.org/catalog/qudt/1.1/index.html

そのほか、[Linked Open Vocabularies
(LOV)](http://lov.okfn.org/dataset/lov/)で広く一般に使われている語彙を見つけることができます。

上記のように一般的な概念を扱ったオントロジーや語彙以外に、生命科学情報を記述する際に、利用しやすいオントロジーもあります。
特に[BioPortal](http://bioportal.bioontology.org/)では、生命科学における適切なオントロジーや語彙を検索することができます。

##### 生命科学情報に関するドメインオントロジー


  | 略称 | オントロジー名 | 名前空間 | 参考リンク |
  |------|-------------|---------|----------|
  |GO | Gene Ontology | http://purl.obolibrary.org/obo/ | [Home](http://geneontology.org/), [BioPortal](http://bioportal.bioontology.org/ontologies/GO) |
  |PRO | Protein Ontology | http://purl.obolibrary.org/obo/ | [Home](http://pir.georgetown.edu/pro/), [BioPortal](http://bioportal.bioontology.org/ontologies/PR) |
  |SO | Sequence Types and Features Ontology | http://purl.obolibrary.org/obo/ |      [Home](http://www.sequenceontology.org/), [BioPortal](http://bioportal.bioontology.org/ontologies/SO) |
  |FALDO | Feature Annotation Location Description Ontology | http://biohackathon.org/resource/faldo# | [GitHub](https://github.com/JervenBolleman/FALDO) |
  |PO | Plant Ontology | http://purl.obolibrary.org/obo/ | [Home](http://plantontology.org/), [BioPortal](http://bioportal.bioontology.org/ontologies/PO) |
  |Taxonomy | Taxnomy Ontology | http://ddbj.nig.ac.jp/ontologies/taxonomy/ | [BioPortal](http://bioportal.bioontology.org/ontologies/NCBITAXON), [DDBJ](http://tga.nig.ac.jp/ontologies/) |
  |Nucleotide | INSDC Nucleotide Sequence Entry Ontology | http://ddbj.nig.ac.jp/ontologies/nucleotide/ | [DDBJ](http://tga.nig.ac.jp/ontologies/) |
  |MeSH | Medical Subject Headings | |           [BioPortal](http://bioportal.bioontology.org/ontologies/MESH) |
  |CL | Cell Ontology | http://purl.obolibrary.org/obo/ |  [BioPortal](http://bioportal.bioontology.org/ontologies/CL) |
  |FMA | Foundational Model of Anatomy | http://purl.org/sig/ont/fma/ |                   [Home](http://sig.biostr.washington.edu/projects/fm/), [BioPortal](http://bioportal.bioontology.org/ontologies/FMA) |
  |UBERON | Uber Anatomy Ontology | http://purl.obolibrary.org/obo/ | [Home](http://uberon.org), [BioPortal](http://bioportal.bioontology.org/ontologies/UBERON) |
  |SNOMED-CT | Systematized Nomenclature of Medicine - Clinical Terms | http://purl.bioontology.org/ontology/SNOMEDCT/ | [BioPortal](http://bioportal.bioontology.org/ontologies/SNOMEDCT) |
  |SIO | Semanticscience Integrated Ontology | http://semanticscience.org/resource/ |      [Home](http://semanticscience.org/), [BioPortal](http://bioportal.bioontology.org/ontologies/SIO) |
  |EFO | Experimental Factor Ontology | http://www.ebi.ac.uk/efo/ | [Home](http://www.ebi.ac.uk/efo), [BioPortal](http://bioportal.bioontology.org/ontologies/EFO) |
  |ECO | Evidence Ontology | http://purl.obolibrary.org/obo |               [Home](http://code.google.com/p/evidenceontology/), [BioPortal](http://bioportal.bioontology.org/ontologies/ECO) |
  |EDAM | EDAM bioinformatics operations, data types, formats, identifiers and topics  | <http://edamontology.org/> | [Home](http://edamontology.org/), [BioPortal](http://bioportal.bioontology.org/ontologies/EDAM) |
  |CMO | Clinical Measurement Ontology | http://purl.obolibrary.org/obo/ |                   [Home](http://phenoonto.sourceforge.net/), [BioPortal](http://bioportal.bioontology.org/ontologies/CMO) |
  | MMO | Measurement Method Ontology | http://purl.obolibrary.org/obo/ | [Home](http://phenoonto.sourceforge.net/), [BioPortal](http://purl.bioontology.org/ontology/MMO) |
  | XCO | Experimental Conditions Ontology | http://purl.obolibrary.org/obo/ |    [Home](http://phenoonto.sourceforge.net/), [BioPortal](http://bioportal.bioontology.org/ontologies/XCO) |
  |OrthO | Ortholog Ontology | http://purl.jp/bio/11/orth# |                      [Home](http://mbgd.genome.ad.jp/ontology/), [BioPortal](http://bioportal.bioontology.org/ontologies/ORTHO) |
  |PIERO | PIERO Enzyme Reaction Ontology　| |[Home](http://reactionontology.org/)|
  |GlycoRDF | Glycan Ontology| http://purl.jp/bio/12/glyco/glycan# |               [Home](https://github.com/ReneRanzinger/GlycoRDF), [BioPortal](http://bioportal.bioontology.org/ontologies/GLYCORDF) |
  |MONDO | Monarch Disease Ontology| http://purl.obolibrary.org/obo/ |               [Home](http://www.obofoundry.org/ontology/mondo.html), [BioPortal](https://bioportal.bioontology.org/ontologies/MONDO) |
  |HPO | The Human Phenotype Ontology| http://purl.obolibrary.org/obo/ | [Home](https://hpo.jax.org/), [BioPortal](https://bioportal.bioontology.org/ontologies/HP) |
  |HCO | The Human Chromosome Ontology| http://identifiers.org/hco/ | [github](https://github.com/med2rdf/hco) |


また、言語名に関する語彙としては、米国国会図書館の提供している[ISO　639-1](http://id.loc.gov/vocabulary/iso639-1.html)や、[ISO
639-2](http://id.loc.gov/vocabulary/iso639-2.html)が利用できます。

なお、SPARQLやTurtleで既存のオントロジーや語彙のURIを利用する際には、しばしば名前空間を短縮形で表記します。例えば、上記の`rdf:type`というURIは`<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>`の短縮表記です。この短縮形はTurtleのprefixed nameという表記方法に従って書かれており、一つのデータセット内において短縮名と実際の表記の対応が矛盾無く宣言されていればどのような短縮名を利用しても問題ないのですが、主要なオントロジーに対しては広く使われている短縮名があり、それを使うことで人間にとっての可読性が上がります。一般的な短縮名を探すためのサービスとして[prefix.cc](http://prefix.cc/)を利用することができます。

#### 1.3.2 新規にオントロジーを構築する

データや知識をRDF化する際に、しばしば、オントロジーが必要になることがあります。典型的な例は、

-   リソースが所属するクラスを明示する場合
-   何らかの情報をリテラルとして記述しているが、概念化できそうな場合
-   主語リソースと目的語リソースの関係を記述する述語が必要な場合

などです。
データ統合の観点からは、既存のオントロジーを利用する方が望ましいですが、既存のものの中に適当なものが見つからない場合は、新規に構築するしかありません。セマンティックウェブでは、一般的にOWLというオントロジー記述言語を利用してオントロジーを定義していきます。OWL自体もRDFで記述します。OWLは（RDFも）テキストエディタで記述できますが、専用のオントロジーエディタを利用した方が効率よく作業することができます。特に、階層構造などを試行錯誤しながらオントロジーを構築する場合には、GUIで作業が行えるオントロジーエディタが不可欠です。OWLオントロジーを記述できるエディタとして次のようなものがあります。

-   [Protege](http://protege.stanford.edu/)
    オープンソースのオントロジーエディタ
-   [WebProtege](http://webprotege.stanford.edu/) クラウド版のProtege
-   [TopBraid](http://www.topquadrant.com/tools/modeling-topbraid-composer-standard-edition/)
    商用のオントロジーエディタ

#### 1.3.3 プロパティにdomainとrangeを定義する

プロパティ(RDFの述語; predicate)には、`rdfs:domain`および`rdfs:range`を定義することができます。これによりプロパティの意味がより明示的になります。

```
例）  
core:classifiedWith rdfs:domain core:Protein .  
core:classifiedWIth rdfs:range core:Concept .  
```

上記の例では、UniProtのタンパクエントリーに、GOなどの機能アノテーション情報をつける`core:classifiedWith`プロパティでの例です。ドメインとして`core:Protein`、レンジとして`core:Concept`が定義されています。classifiedWithという文字列から、人間は「何かで主語を分類するのだな」といったような一定の意味を汲み取ることができますが、機械にはそれができません。

機械には、

```
uniprot:P51028 core:classifiedWith go:2000044 .   
```

という文も、

```
uniprot:P51028 ex:fugahoge go:2000044 .  
```

という文も、`uniprot:P51028`と`go:2000044`の間に関係がある、という以上の意味を持ちません。

オントロジーにドメインとレンジが定義されることで、このプロパティ`core:classifiedWIth`は`core:Protein`タイプ以外のリソースを主語にとらないこと、`core:Concept`以外の値を持つことができないことを、機械が情報処理に利用することができるようになります。

#### 1.3.4 オントロジーのクラスやプロパティの説明を適切に行う

オントロジーのクラスを新たに定義する場合、利用者が、そのクラスが表す概念が何なのか理解できるように、適切な説明をつけることは重要です。
プロパティとしては、`skos:definition`や`rdfs:comment`等を用いて、簡潔でいいので明確に記述して下さい。

```
例)  
core:Active_Site_Annotation rdfs:comment "Amino acid(s) involved in the activity of an enzyme."^^xsd:string;  
```

### 1.4 RDFを提供する際のガイドライン

#### 1.4.1 定期的なリリースとバージョン情報

現状、多くのデータベースは、RDF化する場合、すでに別の形式（リレーショナル・データベース等）で構築・提供しているデータベースから変換していると思います。こういった場合、公開サービスとして提供しているデータと、RDF化したデータとの間で、内容についての同期の問題が発生します。できるだけ、プライマリに提供しているデータと、RDF化したデータとの間との違いが大きくならないように、定期的なRDF化が望まれます。そのために、既存の形式からRDF化への変換はできるだけ自動化するようにし、その過程からは、マニュアル作業が必要な工程をできるだけ排除して下さい。RDF化の過程に大量のマニュアル作業が発生すると、定期的なRDF化は著しく困難になります。マニュアル作業（アノテーション、オントロジーマッピング等）はプライマリのデータベース構築作業の方に含めるようにして下さい。もちろん、プライマリなデータベースのフォーマットがRDFの場合はこの限りではありません。

また、RDF化の際に、適切なバージョン番号を付与することが必要となります。バージョン付けには、`pav:version`
(RDFデータの場合)や、`owl:versionInfo`(オントロジーの場合)が利用できます。

-   [pav:version](http://purl.org/pav/version)
-   [owl:versionInfo](http://www.w3.org/2002/07/owl#versionInfo)

#### 1.4.2 ライセンス情報をつける

ライセンスが明示的に書かれていることで、ユーザが安心してRDFデータを利用することができます。データの利用しやすさの観点からは、[クリエイティブ・コモンズ](http://creativecommons.jp/)の適切な[ライセンス](http://creativecommons.jp/licenses/)を利用すること等が推奨されます。また、オントロジーについてはそもそも著作物であるか否かで議論の余地があるところですが、著作物である場合を想定し、そして、一部を再利用することの容易性から、[CC0](http://sciencecommons.org/resources/readingroom/ontology-copyright-licensing-considerations/)が良いと言われています。

```
例)  
<a RDF data>` dcterms:license <http://creativecommons.org/licenses/by-sa/4.0/> .  
<http://creativecommons.org/licenses/by-sa/4.0/> a <http://purl.org/dc/terms/LicenseDocument> .  
```

To be fixed: 独自ライセンスの場合にどのように記載したら良いかを追記する

#### 1.4.3 スキーマ図を提供する

RDFは機械可読性を高めることを目的とした仕組みであることから、必然的に人間が読むには適していません。しかし、現状では、RDFデータにSPARQLにより問い合わせを行う際などには、そのデータの構造を知っている必要があります。その際に、RDFデータのスキーマ図があることは、データ構造を理解する際に大きな助けになります。

-   [RDFポータルのスキーマ図作成用ライブラリ](https://drive.google.com/file/d/0B5fGKO5915HjdmxrWVJETmJqT0U/view?usp=sharingl)
    （ファイルをダウンロードした後、[draw io](http://draw.io/)
    で「File＞Open from＞Device…」からファイルを選択する）

#### 1.4.4 SPARQLサンプルを提供する

Provide a set of representative queries on how to use your data in
natural language along with the corresponding SPARQL query

#### 1.4.5 SPARQLエンドポイントを公開する際はCORS対策を行う

SPARQLエンドポイントに対する検索をウェブアプリケーションからJavaScriptで行うケースを想定して、SPARQLエンドポイントとなる公開ウェブサーバではどこからでもアクセス可能となるように　、[CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)
(Cross Origin Resource Sharing)対策をすることが強く推奨されます。

具体的にはApacheやNginxなどの設定で`Access-Control-Allow-Origin:　*` ヘッダを付加するようにします。Nginx の場合は

```
server {  
    : (省略)  
  location /sparql {  
    add_header Access-Control-Allow-Origin *;  
    #proxy_pass http://localhost:8890/sparql ;
  }  
}  
```

のような設定を、Apacheの場合は<Location>や<VirtuoalHost>などの該当箇所に

```
<IfModule mod_headers.c>  
  Header set Access-Control-Allow-Origin "*"  
</IfModule>  
```

のような設定を追記します。

なお、トリプルストア自体の提供するウェブサーバを直接SPARQLエンドポイントとして公開する場合は、トリプルストア固有の設定が必要となります。たとえばVirtuosoの場合は、Conductorの管理画面から[設定方法](http://virtuoso.openlinksw.com/dataspace/doc/dav/wiki/Main/VirtTipsAndTricksCORsEnableSPARQLURLs)を参考に設定変更を行う必要があります。

#### 生命科学のLinked Dataを公開する際の10のルール

[BioHackathon 2014](http://2014.biohackathon.org/) でも、Linked Data
公開のルールについて議論されました( [10 simple rules for publishing Linked Data for the Life Sciences](https://github.com/dbcls/bh14/wiki/Ten-simple-rules-for-publishing-Linked-Data-for-the-Life-Sciences))。この議論は論文化にむけて取りまとめが進められていますが、現状ではまだ完成しておらず追記可能です。

### 1.5 生命科学で利用しやすいRDFモデル

以下では、データベースRDF化ガイドラインの補稿として生命科学データベースのRDF化におけるベストプラクティスを蓄積・共有します。

#### 測定項目および測定値の記述方法

##### 単位のついた値の記述方法

-   ブランクノードを利用する方法 （推奨）

W3C のRDF入門でも書かれているように( [構造化された値: rdf:valueに関する詳細](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/rdf/rdf-primer.html#rdfvalue))
、単位のついた値を記述する場合、次に示すように、ブランクノードを利用し値と単位を指定する方法があります。

```
exproduct:item10245 exterms:weight [  
  rdf:value     "2.4"^^xsd:decimal .  
  exterms:units exunits:kilograms .  
] .  
```

上の例は、[構造化された値:　rdf:valueに関する詳細](http://www.asahi-net.or.jp/~ax2s-kmtn/internet/rdf/rdf-primer.html#rdfvalue)
から抜粋したものです。ただ、ここでは、ブランクノードを指すプロパティ（exterms:weight）や 単位を指すプロパティ（exterms:units）および単位を表すクラス（exunits:kilograms）として、具体的に何を使えばよいのか書かれていません。

単位としては、Units of Measurement Ontology (UO:　[BioPortal](http://bioportal.bioontology.org/ontologies/UO),　[Home](https://code.google.com/p/unit-ontology/))　の利用を推奨します。UOに適当な単位が見つからない場合には、Quantities, Units, Dimensions and Data Types Ontologies (QUDT: [BioPortal](http://bioportal.bioontology.org/ontologies/QUDT),
[Home](http://qudt.org/)) の利用を推奨します。

単位を指すプロパティとしては、[`sio:SIO_000221`](http://semanticscience.org/resource/SIO_000221)
(sio:has-unit) の利用を推奨します。また、数値を指すプロパティとしては、[`sio:SIO_000300`](http://semanticscience.org/resource/SIO_000300)
(sio:has-value) の利用を推奨します。

ブランクノードを指すプロパティとしては、[`sio:SIO_000216`](http://semanticscience.org/resource/has-unit.rdf)
(sio:has-measurement-value) の利用を推奨します。複数の単位付き数値を、同じ`sio:has-measurement-value`
プロパティで指すと、プロパティを指定することでは、特定の数値を指定することはできませんが、以下のように、ブランクノードを適切に定義することで（ブランクノードを、適切なオントロジークラスのインスタンスとすることで）解決できます。

```
ex:m1 sio:SIO_000216 [  
  rdf:type       cmo:CMO_0000209 ;  
  sio:SIO_000300 21.5 ;
  sio:SIO_000221 uo:UO_0000309`  
] .  

# CMO_0000209: Blood fibrinogen level defined by [Clinical Measurement Ontology](http://www.ontobee.org/browser/index.php?o=CMO)  
# UO_0000309: Milligram per square meter defined by [Units of Measurement Onotlogy](http://bioportal.bioontology.org/ontologies/UO) .  
# SIO_000216 is has-measurement-value  
# SIO_000221 is has-unit  
```

-   単位の型指定をつかう方法

単位を表すURIを、値の型として指定することもできます。

```
ex:e1 sio:SIO_000216 ex:m1 .  
ex:m1 rdf:type cmo:CMO_0000209 ;  
      sio:SIO_000300 21.5^^obo:UO_0000309 .  
```

#### 1.5.2 遺伝子やタンパク質配列座標情報の記述方法

遺伝子やタンパク質等の配列の座標情報を記述する場合、[FALDO](http://biohackathon.org/resource/faldo): Feature Annotation Location Description Ontology [GitHub](https://github.com/JervenBolleman/FALDO)の利用を推奨します。FALDOは、UniProt、Ensembl、DDBJ等のRDFで利用されています。使い方は、[README](https://github.com/JervenBolleman/FALDO/blob/master/README.md) や、[FALDO論文](https://jbiomedsem.biomedcentral.com/articles/10.1186/s13326-016-0067-z)に記載されていますので、それを参考にして下さい。
また、FALDOで参照配列を記述する際には、HCO（Human Chromosome Ontology）の利用を推奨します。

#### 1.5.3 サンプルを記述する方法

生命科学データベースで、サンプルについて記述することがしばしばあります。そのサンプルが、BioSamples RDFに登録されている場合は、そのURIを参照するようにしてください。参考までに、EBI BioSamplesでは、ユーザがサブミットしたサンプルの特徴を、次のような記述方法で、機械的にRDFへ変換することを計画しています。キーと値のペアがあった場合、

```
key  value
```

次のように、変換されます。

```
ex:sample 
  ex:hasCharacteristics [
     ex:propertyType a:mappledOntology ;
     ex:propertyValue "Original description for a key" ;
  ] , [
  ex:propertyType a:mappledOntology ;
  ex:propertyValue "Original description for a value" ;

  ] .

```

また、サンプルのURIは、sio:SIO_001050（sio:sample）のインスタンスとして定義してください。

```
ex:e1 rdf:type sio:SIO_001050 .  
```


* * *

### 参考URL

#### RDF化を支援するツール

ガイドラインに従って実際にRDF化を進める際に有益なツール群を紹介します。

-   [OpenRefine with RDF extension](http://refine.deri.ie/)
-   [TogoDB](http://togodb.org/)
-   [D2RQ](http://d2rq.org/)
-   [-ontop-](http://ontop.inf.unibz.it/)
-   [Protege](http://protege.stanford.edu/)
-   [WebProtege](http://webprotege.stanford.edu/)
-   [TopBraid Composer Standard
    Edition](http://www.topquadrant.com/tools/modeling-topbraid-composer-standard-edition/)
-   [TopBraid Composer Maestro
    Edition](http://www.topquadrant.com/tools/IDE-topbraid-composer-maestro-edition/)
-   [Silk](http://silk-framework.com/)

* * *

#### RDF化の際に参考になるサイト

-   [HCLS Dataset
    descriptions](http://htmlpreview.github.io/?https://github.com/joejimbo/HCLSDatasetDescriptions/blob/master/Overview.html)
-   [LOD Diary](http://www.infocom.co.jp/das/loddiary/linked_data/)
-   [Linked Data: Evolving the Web into a Global Data
    Space](http://linkeddatabook.com/editions/1.0/)

(邦訳は[Linked data :
webをグローバルなデータ空間にする仕組み](http://ci.nii.ac.jp/ncid/BB11534438))

#### RDF化ガイドラインに対するコメントを自由に書き込むページ

-   [RDF化ガイドラインへのコメント](http://wiki.lifesciencedb.jp/mw/RDFizingDatabaseGuidelineComments)

* * *

#### History

バージョン 1.5.0 as of “2015-12-07”\^\^xsd:date バージョン 1.4.0 as of
“2015-08-11”\^\^xsd:date

-   本ガイドラインのバージョン番号は
    [セマンティック・バージョニング](http://semver.org/lang/ja/) を採用
    (2015-08-10)
    -   メジャーバージョンは、これまで作成した RDF/Ontology
        の大幅な更新が必要な非互換な改定を行った場合にインクリメント
    -   マイナーバージョンは、新しく情報を追加した場合にインクリメント
    -   パッチバージョンは、文言の改善など意味的に変更がない場合にインクリメント
    -   「てにをは」や typo の修正については Wiki
        なのでバージョン変更なしに書き換え

2015-12-07 バージョン 1.5.0 文献情報の記述について提案を追記。（skwsm）

2016-03-28 バージョン 1.6.0 複数のID定義の非推奨について追記。（skwsm）

2016-10-21 バージョン 1.7.0
スキーマ図を作成することについて追記。SPARQLサンプルを付与することを追記。（skwsm）


2016-10-21 バージョン 1.7.1
「クラスの定義を適切に行う」項目を、「オントロジーのクラスやプロパティの説明を適切に行う」に変更（定義を適切に行う、だと、様々なクラス制約等を定義しているように読めるので）。(skwsm)

2017-04-13 バージョン 1.8.0 「2.2.9 URIにはバージョンを含めない」を追記。（skwsm）

2017-05-15 バージョン 1.9.0 「2.2.9 文献情報へのリンク」。改変（プロパティに関するTo do を推奨へ）（skwsm）

2018-03-20 バージョン 1.9.1 「1.2.9 httpsで始まるURIを利用しない。」を追記。（skwsm）

2018-06-28 バージョン 1.9.2  1.2.9 QUDTの情報を更新。MONDOとHPOを追加。（skwsm）

2018-06-28 バージョン 1.9.3  1.2.9 HCOを追加。1.5.2 HCOの利用について追記。（skwsm）
