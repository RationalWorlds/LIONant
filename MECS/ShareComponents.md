# Share Components

## Core Concepts
This is a work in progress document and on the way of conceptualizing rather than these concepts been implemented.

### Types of Share Components
System will seach for their share components in the following order:
1. Entity Share Components -> 
2. Pool Share Components -> 
3. Archetypes Share Components -> 
4. World Share Components

### World Share Componets
- A set of components that can be access by any system in a particular world.
- There can be a maximun of 1 instance/value per type for each World.
- System can request them based on scoping issues
- System can only access these types of components as a read only mix with other types of non-share-components or 
as a mutable component when no other types of components been requested.

### Archetype Share Components
- A set of components that are share across all entity pools that belong to the Archetype.
- There can be a maximun of 1 instance/value per type for each Archetype.
- System can request them based on scoping issues
- System can only access these types of components as a read only mix with other types of non-share-components or 
as a mutable component when no other types of components been requested.

### Pool Share Components
- A set of components that belong to a particular entity pool.
- There can be a maximun of 1 instance/value per type for each Pool.
- System can request them based on scoping issues
- System can only access these types of components as a read only mix with other types of non-share-components or 
as a mutable component when no other types of components been requested.

### Entity Share Components
- A set of components that belong to a particular entity. However MECS factor outs share components 
from entities with the same values and groups those entities together into a different pools.
- There can be a maximun of 1 instance/value per type for each Entity.
- System can request them based on scoping issues.
- System can access these types of components mix with other types or alone.

## Possible way to implemented this

### As unique types

```c++
    struct entity_share {};

    struct pool_share {};

    struct archetype_share {};

    struct world_share {};
```
Benefits:
- Compile time type checking

### As a single type with parameter to constraint

```c++
    enum class type_shape : u32
    {
        ENTITY
    ,   POOL
    ,   ARCHETYPE
    ,   WORLD
    };

    struct entity_share 
    {
        static constexpr auto type_shape_v = _type_shape::ENTITY;
    };
```

Benefits:
- Compile time type checking

### As a single type without data/type constraints
The idea for this method is that it will no differenciate between the types and rather the behavior will be determine a run time.
So if you added the share component to an Archetype as apart of an entity descriptor then it will be treated as an entity-share.
If stead you added specifically to a pool then that share component will be a pool-share, same thing for the rest.

```c++
    struct share {};
```

Benefits:
- Same component can be use reuse for different situations.
- Minimizes the use of unique types and there for of bits

### As two different unique types
Here share will be mean entity-share, and global_share is meant to be use as a generic type for the rest.

```c++
    struct share {
        // When accessing as mutable it automatically locks the entire entity against structural changes
    };

    struct global_share {
        // Will accept access as LINEAR or as QUANTUM, it must be specified
    };
```

Benefits:
- Creates a clear line of separation between entity component and the rest. Which is definetly the most different in concept.
- Compile time type checking for the most distance concepts

Notes:
- for global_share access locking rules will apply.
- global_share components will be allocated individually.

#### //// Most likely the winner ////
The method that most clearly separates all the concepts and enables enough type checking at compile time

### As one type and regular components
Here share will be mean entity-share, all other types of data components could be use to represent all other types.
This will be possible because the way to add the non-entity-share components will be through a different pipe.
However this come with the problem that in a query things can get a bit ambiguous.

```c++
    struct share {};
```

Benefits:
- Compile time type checking for the most distance concepts

## Competitive analysis
Unity 3D seems has two type of share-components. entity-share, and pool-share. However it does not have the Archetype-share, and the World-share.
This is ok for them as they can create an entity for each of those types. However is not ideal, and their pool-share are not accessible by systems easily.


## Entity-Share vs Pool-Share vs Archetype-Share
These 3 kinds of components are nearly identical in their storage situation.
All these types are allocated in pools in the Archetype. Entity-Share and the Pool-Share index into the Archetype pools to collect their instances.
You could even claim that the Archetype-Share could be factor out into a World-Share pools. If that is the case begs the question why are all share components
treated as an entity with that component and everyone else just have a guid to that particular instance. This line of thinking help go deeper into this topic.



