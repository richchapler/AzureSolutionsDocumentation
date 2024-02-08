# Demonstrating Graph Database Functionality in ADX
...particularly re: Bill-of-Materials use case

## Prompt 1
```
using this example

let nodes = datatable(name:string, type:string, age:int) 
[ 
  "Alice", "Person", 23,  
  "Bob", "Person", 31,  
  "Eve", "Person", 17,  
  "Mallory", "Person", 29,  
  "Trent", "System", 99 
]; 
let edges = datatable(source:string, destination:string, edge_type:string) 
[ 
  "Alice", "Bob", "communicatesWith",  
  "Alice", "Trent", "trusts",  
  "Bob", "Trent", "hasPermission",  
  "Eve", "Alice", "attacks",  
  "Mallory", "Alice", "attacks",  
  "Mallory", "Bob", "attacks"  
]; 

write kql to create nodes and edges tables that can be used with a Bill-of-Materials where SKU is the identifier which identifies a specific part. Parts can range in complexity from screw to engine, where screw and other parts hierarchically build to create an engine
```

### Sample Tables and Data
```
let parts = datatable(SKU: string, partName: string, complexity: string)  
[   
  "SKU1", "Screw", "Low",  
  "SKU2", "Bolt", "Low",  
  "SKU3", "Gear", "Medium",  
  "SKU4", "Engine Block", "High",  
  "SKU5", "Engine", "Very High"  
];   
  
let relationships = datatable(source: string, destination: string, relationship_type: string)  
[   
  "SKU1", "SKU3", "isPartOf",  
  "SKU2", "SKU3", "isPartOf",  
  "SKU3", "SKU4", "isPartOf",  
  "SKU4", "SKU5", "isPartOf"  
];  
```

## Prompt 2
```
now apply this logic on top of those datatables  
&nbsp;  
edges   
| make-graph source --> destination with nodes on name   
| graph-match (mallory)-[attacks]->(compromised)-[hasPermission]->(trent)   
  where mallory.name == "Mallory" and trent.name == "Trent" and attacks.edge_type == "attacks" and hasPermission.edge_type == "hasPermission"   
  project Attacker = mallory.name, Compromised = compromised.name, System = trent.name 
```

### Sample Usage
```
let parts = datatable(SKU: string, partName: string, complexity: string)  
[   
  "SKU1", "Screw", "Low",  
  "SKU2", "Bolt", "Low",  
  "SKU3", "Gear", "Medium",  
  "SKU4", "Engine Block", "High",  
  "SKU5", "Engine", "Very High"  
];  
let relationships = datatable(source: string, destination: string, relationship_type: string)  
[   
  "SKU1", "SKU3", "isPartOf",  
  "SKU2", "SKU3", "isPartOf",  
  "SKU3", "SKU4", "isPartOf",  
  "SKU4", "SKU5", "isPartOf"  
];
relationships   
| make-graph source --> destination with nodes=parts on SKU   
| graph-match (screwOrBolt)-[isPartOf]->(gear)-[isPartOf]->(engineBlock)-[isPartOf]->(engine)   
  where screwOrBolt.SKU in ("SKU1", "SKU2") and engine.SKU == "SKU5" and isPartOf.relationship_type == "isPartOf"   
  project ScrewOrBolt = screwOrBolt.partName, Gear = gear.partName, EngineBlock = engineBlock.partName, Engine = engine.partName  
```
