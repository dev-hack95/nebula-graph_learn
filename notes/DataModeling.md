# Data Modeling

* Data modeling for knowledge graph is the process of defining the structure and sematics
  of interconnected data that respresnts real world knowledge, entities and thier relation
  ships so the whole idea is organized data according to their relationship and their weight
  on that relations


## Data Structures (Nebula Graph)

* Nebula Graph models uses six data sturcures

  1) Spaces
  2) Vertices
  3) Edges
  4) Tags
  5) Edge Type
  6) Properties


1) **Graph Spaces**

  * **Description** : A logical container for graph data, used from different teams or
    application. Each graph spcae has its own storage configuration(e.g replication,
    partition), access, privileges and schema

  * **Key Features**

    * Contains vertices and edges.

    * Supports independent settings for storage and security.


```mermaid
classDiagram
    class GraphSpace {
        +name: string
        +partition_num: int
        +replication_factor: int
        +properties: map<string, string>
    }
```

2) **Vertices**

  * **Description** : Represent entities (e.g. users, products). Each vertex has a unique
    identifer(VID) and can belong to mutiple tags

  * **Key Features**

    * VID : Unique Identifer (int64 or fixed_string(N))

    * Tags: Tags are oprional in latest version of nebula graph i.e vertices can have zero tags

```mermaid
classDiagram
    class Vertex {
        +VID: string|long
        +tag: Tag
        +properties: map<string, string>
    }
```

3) **Edges**

  * **Description** : Edges are directed connection b/w verticrs(nodes), representing relation
    ship (e.g "followers", "ratings"). Edges are uniquely identified by

    * Source Vertex

    * Edge type

    * Rank Value (immutable, for odering multiple edges b/w same vertices)

    * Destination vertex


```mermaid
classDiagram
    class Edge {
        +source: Vertex
        +destination: Vertex
        +edge_type: EdgeType
        +rank: int
        +properties: map<string, string>
    }
```

4) **Edge Types**

  * **Description** : Categories for edges, defining their demantic anf properties. Each edge
    must have exactly one edge type.

  * **Key Features**

    * Each edge type defines properties (keys) for edges

    * Used to enforece consistency in edge metadata.

```mermaid
classDiagram
    class EdgeType {
        +name: string
        +properties: map<string, string>
    }

```

5) **Properties**

  * **Description** : Key value pairs that stores metadata for vertices and edges

  * **Key Features**

    * Both vertices and edge can have properties.

    * Property keys are defined by tags or edge types.

```mermaid
classDiagram
    class Property {
        +key: string
        +value: string
    }
```

6) **Tags**
  
  * **Description**: Labels for categorizing vertices (similar to labels in other graph 
    databases). Vertices with the same tag share the same property keys.

  * **Key Features**

    * Each tag defines a set of properties (keys).
    * Properties must be consistent for all vertices using the tag.

```mermaid
classDiagram
    class Tag {
        +name: string
        +properties: map<string, string>
    }
```

### Relationship Diagram

```mermaid

graph TD
    G[Graph Space] --> V[Vertex]
    G --> E[Edge]
    
    V --> VID[VID]
    V --> T[Tag]
    T --> P[Properties Keys]
    
    E --> SRC[Source Vertex]
    E --> DEST[Destination Vertex]
    E --> ET[Edge Type]
    E --> R[Rank]
    
    SRC --> V
    DEST --> V
    ET --> P

````

## Nebula Graph Directed Property graph

  * NebulaGraph stores data in directed property graphs. A directed property graph is a
    graph database structure where verticrs and directed edges are connected and both 
    can have assocaited properties.

  * The graph is represnted as G = <V, E, PV, PE>

    * **V** : A set of vertices (nodes).
    * **E** : A set of directed edges (relationships or connections).
    * **PV** : Property definitions for vertices.
    * **PE** : Property definitions for edges.


---

#### Vertex

  * **Definition**
    
    * A vertex represents an entity in the graph (e.g., a person, an object).
    
    * Each vertex has a unique identifier called VID (Vertex ID).

  * **Key Characteristics**
    
    1) **VID** : Must be unique within a graph space. It can be either int64 or fixed_string(N).
    
    2) **Tags** : Vertices can belong to one or more tags. Tags are used to categorize vertices 
       and define their property schema.

      * Example: In the basketball player dataset, player and team are vertex tags.

    3) Properties: Key-value pairs associated with a vertex. Property keys are defined by tags.
       
      * **Example** : For player tag: name (string) and age (int).

      * For team tag: name (string).

**Example**

|Vertex | Tag	Property Keys |	Description
|-------|-------------------|----------------------------------
|player	| name, age         | Represents players in a team.
|team	  | name	            | Represents teams with their names.

---

###  Edge (Edge)
  * **Definition**
  
    * An edge represents a directed relationship between two vertices.

    * It is uniquely identified by <source vertex, edge type, rank, destination vertex>.

  * **Key Characteristics**
    
    1) **Directed** : Edges are directed (i.e., they point from one vertex to another).
       
      * **Example** : The edge follow points from a follower to a followee.

    2) **Edge Type** : Each edge has one and only one edge type, which defines its property schema.
      
      * **Example** : In the basketball player dataset, serve and follow are edge types.
    
    3) **Rank** : An immutable 64-bit signed integer used to order edges with the same edge type and source-destination pair.
      
      * **Default** : 0. Higher rank values appear first.
      
      * **Example** : The follow edge type includes a property degree (int) to represent follower ratings.

**Example**

|Edge Type	| Source Vertex	| Destination Vertex	| Properties	                |Description
------------|---------------|---------------------|-----------------------------|------------------
|serve	    | Player	      | Team	              | start_year, end_year (int)	| Represents a player serving a team.
|follow	    | Player	      | Player	            | degree (int)	              | Represents a player following another player on Twitter.
