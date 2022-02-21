# Compass aligned graph embeddings
Testing graph embedding extracted from two different knowledge graphs and aligend with CADE as a framwork to:
- discover new entity link between graphs;
- merge knowledge graphs;  
  
Project for AI course 2021/22, master degree in computer science, [UNIMIB](https://www.unimib.it/).
## Phase 1: Walk extraction
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/AlbezJelt/pyrdf2vec-for-graph-embeddings/blob/main/notebooks/Walks_extraction.ipynb)  
In the first phase random walks are extracted from [DBpedia]('https://www.dbpedia.org/') and [Wikidata]('https://www.wikidata.org/wiki/Wikidata:Main_Page') starting from a fixed set of entities, using a modified version of [pyRDF2Vec]('https://github.com/AlbezJelt/pyRDF2Vec').  
This library uses the sparql endpoint to directly querying the graph with two specific queries, one for each endpoint:
- **DBpedia**  
    Extract all (predicate, object) starting from *\*subject\** where the object have a *owl:sameAs* link to Wikidata.
    ```
    SELECT ?p ?o WHERE {
        *subject* ?p ?o.
        FILTER(EXISTS {
            ?o owl:sameAs ?WikidataEntity.
            FILTER(CONTAINS(STR(?WikidataEntity), "wikidata.org/entity"))
        })
    }
    ```
- **Wikidata**  
    Extract all (predicate, object) starting from *\*subject\** where the object is a Wikidata entity.
    ```
    SELECT ?p ?o WHERE {
        *subject* ?p ?o.
        FILTER(CONTAINS(STR(?o), "wikidata.org/entity/Q"))
    }
    ```
**By keeping only the entities and excluding the predicates** two corpora ([1]('https://github.com/AlbezJelt/compass-aligned-graph-embeddings/raw/main/data/dbpedia_walks_final.txt'), [2]('https://github.com/AlbezJelt/compass-aligned-graph-embeddings/raw/main/data/dbpedia_walks_final.txt')) are constructed using the extracted walks.

## Phase 2: Preprocessing
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/AlbezJelt/compass-aligned-graph-embeddings/blob/main/notebooks/Neighborhood_based_sameAs.ipynb)  
In the second phase all the DBpedia walks entities are converted to Wikidata entities following the owl:sameAs links ([converted walks]('https://github.com/AlbezJelt/compass-aligned-graph-embeddings/raw/main/data/dbpedia_walks_final.txt')). This ensure that the two corpora share the same vocabulary, which is a necessary condition to use Word2Vec technique.  
Each DBpedia resource can have multiple link to different Wikidata entites, so a neighborhood based filtering is applied to extract the best match.  
Finally, a [dictionary]('https://github.com/AlbezJelt/compass-aligned-graph-embeddings/raw/main/data/wikidata_label_dictionary.json') is built that maps each Wikidata entity to its label (usefull to translate each entity in natural language in the next step).

## Phase 3: CADE embeddings and evaluation
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/AlbezJelt/pyrdf2vec-for-graph-embeddings/blob/main/notebooks/Cade.ipynb)  
In the third phase two Word2Vec models are trained and aligned using [CADE]('https://github.com/vinid/cade'), a Python package that generate compass aligned distributional embeddings. With this method it's possible to compare the output vectors of the two models using different metrics (ie. cosine similarity).

## Phase 4: Graph merging
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/AlbezJelt/compass-aligned-graph-embeddings/blob/main/notebooks/Graph.ipynb)  
In this final phase a new graph is created using a recursive alghoritm that exploit similarity between word vectors to create new edges between entities. Also this algorithm 
provide a **link discovery method** to link DBpedia entites without a *owl:sameAs* link to Wikidata entities.
