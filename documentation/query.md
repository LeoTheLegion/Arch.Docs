---
description: The Query, a way to select and iterate entities
---

# Query

But how do you actually organize your entities? Well, there are queries for that.

A `Query` is a view of the [World](world.md) and **targets a specific set of** [**entities**](entity.md). It only consists of a `QueryDescription`, which specifies which entities with **which structure are being searched for**. You can specify which components an entity should definitely have, which it should perhaps have and which it should not have at all. It is also possible to search only for the exclusive structure of an entity. Let's take a look at this!

## Iteration

One dwarf has probably become hundreds. And the elves and humans have also joined us. They buzz around, without a goal, without a task. Time to crack the whip, isn't it?

<pre class="language-csharp"><code class="lang-csharp">// Creating an army of dwarfs
for(var index = 0; index &#x3C; 1_000, index++){
    world.Create(new Dwarf(), new Position(0,0), new Velocity(1,1), new Pickaxe());
    world.Create(new Elf(), new Position(0,0), new Velocity(1,1), new Bow());
    world.Create(new Human(), new Position(0,0), new Velocity(1,1), new Pickaxe(), new Sword());
}

// Iterating over all entities that have Position &#x26; Velocity to make them move. 
<a data-footnote-ref href="#user-content-fn-1">var movementQuery = new QueryDescription().WithAll&#x3C;Position, Velocity>();</a>
world.Query(in movementQuery, (Entity entity, ref Position pos, ref Velocity vel) => {
    pos += vel;
    Console.WriteLine($"Moved: {entity}");
});
</code></pre>

{% hint style="info" %}
The methods of the `QueryDescription` can accept up to 25 generic parameters and can be chained together.  Can also be created without generics by passing [`Signature`](utilities/non-generic-api.md). There multiple different [`Query-Variants`](optimizations/query-techniques.md). More on this later.
{% endhint %}

It looks wonderful, all entities are already moving. It doesn't matter whether they are dwarves, elves or humans. As long as they have the **Position & Velocity** components, they move!

## Any and None

See how your creatures are marching now, do you feel the power? But what do we do, for example, if we want all beings with a pickaxe **except** the highborn Elven to work for us in the mines? It's time to make the queries a little more complex.

```csharp
// Targets entities with a pickaxe that are not Elven. So humans and dwarves.
var letDwarfsAndHumansMine = new QueryDescription().WithAll<Pickaxe>().None<Elf>();
world.Query(in movementQuery, (ref Pickaxe pickaxe) => {
    var rock = FindNextRock();
    MineSomeOres(pickaxe, rock);
});
```

{% hint style="info" %}
You do not necessarily have to pass the [entity](entity.md) in a `Query`.
{% endhint %}

Excellent, look how they work. But have you thought about the defense? What if we are attacked? Then we would need everyone with a weapon, **whether** sword or bow, to defend ourselves.

```csharp
// Targets entities with either a bow or sword. So elves and humans. 
var letElvesAndHumansPatrol = new QueryDescription().WithAny<Bow, Sword>();
world.Query(in movementQuery, (Entity entity) => {
    
    ref var bow = entity.TryGet<Bow>(out var hasBow);
    ref var sword = entity.TryGet<Sword>(out var hasSword);

    if(EnemyNearby()){
        if(hasBow) bow.Attack();
        if(hasSword) sword.Attack();
    }
});
```

And we have already created order in our new little kingdom. Everyone does what they can and what they were meant to do.

## Exclusive

Silence falls and you realize that there is a problem. A new breed of dwarf has grown up. They are even smaller and slighter than their ancestors and can no longer even carry a pickaxe.&#x20;

```csharp
// New breed of dwarfs, too weak to wear pickaxes
for(var index = 0; index < 1_000, index++){
    world.Create(new Dwarf(), new Position(0,0), new Velocity(1,1));
}
```

You want to give them an exclusive task. But how do we tell them apart from the rest? With one of those?

```csharp
var firstQueryDesc = new QueryDescription().WithAll<Dwarf, Position, Velocity>();
var secondQueryDesc = new QueryDescription().WithAll<Dwarf, Position, Velocity>().None<Pickaxe>();
```

{% hint style="danger" %}
No. Both `QueryDescriptions` would also target dwarfs that have additional components (e.g. \[`Dwarf, Position, Velocity, Bow`]). We do not want that.
{% endhint %}

Instead, we only want to target this one type of dwarf. No more or less components on them.

```csharp
// Only targeting entities with exact those components, no more or less.
var makeWeakDwarfsExile = new QueryDescription().WithExclusive<Dwarf, Position, Velocity>();
world.Query(in makeWeakDwarfsExile, (Entity entity) => {
    Exile(entity);
});
```

It's as simple as that. And you've already given them the task of disappearing. We don't need dwarves like that, you're so brutal!

[^1]: This is also possible, but it would be better if the `QueryDescription` was not created before each query, but only once.
