# YAGO 4.5: A Large and Clean Knowldege Base with a Rich Taxonomy

## 1 INTRODUCTION

## 2 RELATED WORK <recalls related work>
- **Some KBs, what type of KB they are and some issues:**  
    ConceptNet, BabelNet, DBPedia, Freebase and [Wikidata].  
- **Some ontologies:**  
    Cyc, SUMO, DOLCE, BFO, WordNet and [Schema.org]

We'll select Wikidata and Schema to build our KB

## 3 DESIGNING YAGO <discusses design decisions>
### 3.1 DESIGN RATIONALE
We want to have a general, precise and non-redundant upper taxonomy
### 3.2 DESIGN PRINCIPLES
We want an upper taxonomy from Schema and a lower taxonomy from Wikidata, the integration is achieved through these principles:  

**1. PREFER PROPERTIES OVER CLASS MEMBERSHIP**
    We prefer to do (A) -[has property]-> (B)  
    Instead of (A) -[is of type]-> (B)  
        Making A an instance of the B class  

**2. CHOOSE THE PROPERTY WITH FEWER OBJECTS**  
    If we have some property like (C)<-(A)->(B) we prefer to do the inverse, (C)->(A) and (B)->(A)  
    this way we have less objects per property  

**3. THE UPPER TAXONOMY EXISTS TO DEFINE FORMAL PROPERTIES THAT WILL BE POPULATED**  
    All classes of upper taxonomy must define formal properties. Both the domain and range of these properties have to be upper-level classes. This makes the upper taxonomy a self contained schema so you don't need to search for property definitions in the lower classes  

**4. THE LOWER TAXONOMY EXISTS TO CONVEY HUMAN-INTELLIGIBLE INFORMATION ABOUT ITS INSTANCES IN A NON-REDUNDANT FORM**  
    This targets mainly human users. The design principle tells us to remove classes that add no information over upper taxonomy classes, merge classes that are hard to distinguish and remove classes that aren't populated  

### 3.3 UPPER TAXONOMY  
How was the upper taxonomy built?  
#### UPPER-LEVEL CLASSES  
We started with the taxonomy of schema.org, Excluding the classes Action, BioChemEntity and MedicalEntity.  
All top-level classes are disjoint except places/organizations and products/creative works  
#### FICTIONAL ENTITIES  
We add a class yago:FictionalEntity as a sublcass of schema:Thing, by defining properties like "createdBy" and "appearsIn" its existence can be justified under Design Principle 1.  
Fictional entities are not disjoint from any other classes since everything can be fictional (Like a place, a person, an invention, etc.)  
#### INTANGIBLES  
We added the classes Award, Gender and BeliefSystem, all subclasses of schema:Intangible. Other subclasses of schema:Intangible were removed under Design Principle 3  
#### PLACES  
Schema's taxonomy on places is very web page oriented, some classes don't define properties that could be populated so they're removed under Design Priniciple 3.  
Then, a new taxonomy was manually created like yago:HumanMadePlace and yago:AstronomicalObject  
### 3.4 LOWER TAXONOMY
**MAPPING TO WIKIDATA**

The lower levels of YAGO 4.5's taxonomy comes from Wikidata  
This mapping can give rise to the following situations:  

**ONE-TO-ONE MAPPING**  
    (Upper class) -> (Wikidata class)  

**ONE-TO-MANY MAPPING**  
    (Wikidata class) <- (Upper class) -> (Wikidata class)  

**ONE-TO-NONE MAPPING**  
    This didn't happen in YAGO 4, it is now used for classes with a convoluted taxonomy in Wikidata  
    It can be used for one of the following classes:  
- **schema:Thing**  
            In YAGO 4 this used to be mapped to entity, which resulted in 1 million direct instances of schema:Thing. This goes against Design Principle 3. So now we only accept entities that fall into one of the manually approved subclasses of Thing.  

- **schema:Place**  
            Wikidata's taxonomy for places is highly convoluted, is difficult to tell apart terrain, geographical location, geographical region, geographical area and location.  

- **schema:Intangible**  
            Likewise, are highly convoluted: class, process or role. It's hard to establish properties to adhere to Design Principle 3  
### 3.5 INSTANCES
**Identifiers**  
Every instance is automatically asigned a readable name, normally we use the title of the Wikipedia page.  
If there's none, we use the enlish label and concatenate it to the Wikidata Q-id.  
If there's none, we use a label that contains legal characters concatenated with the Wikidata id.  
**Instances vs. Classes**  
Wikidata contains items that are both Instances and Classes. Because of our discussion in 3.4, these items would be considered classes. At the same time, if we wanted to say that Eleanor Roosevelt spoke english, we would make a statement about the class, this statement is allowed thanks to OWL 2 by a mechanism called "punning"
## 4 IMPLEMENTING YAGO <discusses implementation issues>
### Infrastructure
YAGO 4 was written in Rust but was hard to mantain so we moved to python
### Data formats
(Not as interesting as it sounds, just discussing formats of the project like BZ2, GZIP, NT, etc.)
### Data processing:
All of this happens in a Unix machine with 90 CPUs and 800Gb of RAM
Steps:
#### 1.- Create schema | time < 1s  
      Loads the schema
#### 2.- Create taxonomy | time = 4hr  
      Parses wikidata into classes and constructs a loop-free taxonomy
#### 3.- Create facts | time = 4hr  
      Parses wikidata into facts in the form of a YAGO predicate
#### 4.- Type-check facts | time = 1.5hr  
      The previous step outputed a list of facts, this list is now loaded into memory and each one is type-checked
#### 5.- Create ids | time = 1hr  
      All type-checked facts are now mapped to a YAGO name
#### 6.- Create statistics | time = 1hr   
      Counts things like number of instances per class, facts per predicate, etc.

| Total time = 12hr

## 5 RESULT <presents the resulting KB>
TL:DR YAGO 4.5 better than YAGO 4 better than Wikidata 
### 5.1 RESOURCE
**Size**  
    YAGO has less predicates than wikidata deliberately according to Design Principle 3 but it does cover the 100 most frequent predicates
    YAGO 4.5 is smaller than YAGO 4 because of the removal of inverse properties according to Design Principle 2
**Data Format:**  
YAGO is split into these files:  
  - **Schema:** Upper taxonomy, property definitions  
  - **Taxonomy:** Hierarchies between classes as (a)-[is]->(b), better explained in General analysis.txt  
  - **Facts:** All facts about entities that have an english wikipedia page  
  - **Beyond Wikipedia:** Facts about entities that don't have that  
  - **Meta:** Temporal annotations  
### 5.2 EVALUATION

table 2:
| Criterion        | Operationalization | Wikidata | YAGO 4 | YAGO 4.5 |
| :---             |    :---                     | ---: | ---: | ---:|
| Consistency      | Absence of contradictions   | no   | yes  | yes |
| Complexity       | Top-level classes           | 41   | 2714 | 9   |
|                  | Number of paths to root     | 44   | 1.1  | 2.3 |
| Modularity       | Disjointness axioms         | 0    | 18   | 24  |
| Concisness       | Taxonomic loops             | 62   | 0    | 0   |
|                  | Redundant taxonomic loops   | 377k | 1216 | 0   |
|                  | Reduntant relations         | 118  | 6    | 0   |
|                  | Classes without instances   | 2.6M | 73   | 0   |
| Understandabilty | Human-readable names        | 0%   | 89%  | 91% |
| Coverage         | Classes per instance        | 8.4  | 3.6  | 7.8 |
|                  | Facts per instance          | 4.8  | 5.1  | 2.7 |

#### Intrinsic evaluation  
It is measured in this parameters:  
- **Consistency:** Absence of logical contradictions  
- **Complexity:** How complicated the ontology is, it's meassured by the number of top classes (i.e. All direct subclasses of schema:Thing) and the average number of paths from an instance to the taxonomy root  
- **Modularity:** How much is the ontology composed of discrete subsets, in this case, the subsets are disjoint classes (Not sure what a disjoint class is)  
- **Conciseness:** Absence of redundancies  
- **Understandability:** How comprehensible the ontology is, measured by the percentage of identifiers that have a human-raedable name  
- **Coverage:** The degree to which the ontology covers a certain domain of knowledge, measured by the number of classes and facts per instance  
#### Extrinsic evaluation  
In here it mentions some attempts to solve ambiguous concepts.  
It says that for each entity, it collects all candidates that share a label. Then it uses end-to-end entity linking system ExtEnD [6] that was pretrained on LongFormer [8], I conserved two reference indexes so it can be looked upon deeper  
### 5.3 APPLICATIONS
#### - BENCHMARKING
YAGO has been widely used as benchmark datasets in entity type prediction [29] and link prediction [24]

#### - YAGO IN INFORMATION RETRIEVAL

For systems of information retrieval, YAGO has resulted useful in aiding with understanding queries and documents beyond the scope of word tokens and plain texts.  
It can help in two ways: 

  - **Document retrieval:**  
Finding relevant textual resources for a given user query 

  - **Entity retrieval:**  
Retrieves all entities from a document 
#### OTHER APPLICATIONS
YAGO has been cited more tan 10,000 times according to Google Scholar 
#### AVAILABILITY 
Mentions YAGO's website and github page 
## 6 CONCLUSION