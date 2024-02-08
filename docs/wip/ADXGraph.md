# Demonstrating Graph Database Functionality in ADX
...particularly re: Bill-of-Materials use case

## Create Parts Table
```
.create table Parts (Id: int, ParentId: int, Name: string)  
```

## Ingest Sample Data
```
.ingest inline into table Parts <|   
1, null, 'Part 1' |  
2, 1, 'Part 2' |  
3, 1, 'Part 3' |  
4, 2, 'Part 4'   
```

## Query Data as a Graph
```
Parts  
| evaluate graph_traverse('Part 1', 'Part 2', hop(1,2), 'ParentId', 'Id')  
```
