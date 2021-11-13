# Registries [:fontawesome-brands-github:](https://github.com/GuusLieben/Hartshorn/tree/develop/hartshorn-persistence/src/main/java/org/dockbox/hartshorn/persistence/registry){:target="_blank"}

In some cases, particularly with hierarchical data, it can be useful to store this information in a key-based column data structure. The `Registry<V>` data structure provides an easy and efficient way this can be done by building on the advantages of a `HashMap<K,V>`.

A `Registry<V>` stores information in columns, which consist of a key (`RegistryIdentifier`) and a list of the values (`RegistryColumn<V>`). `RegistryColumn<V>` extends an `ArrayList<V>`, however, it provides additional methods which make accessing the information in a Registry far easier.

### Registry Identifiers

Each column within a Registry is stored against a `RegistryIdentifier`. You can define your own Registry keys by extending the `RegistryIdentifier` interface. You can create your own `RegistryIdentifier` simply as follows:

??? info "View imports"

    ```java
    import org.dockbox.hartshorn.persistence.registry.RegistryIdentifier;
    ```
```java
public enum ExampleIdentifier implements RegistryIdentifier {
   ExampleKey1, ExampleKey2
   //...            
}
```

You can use anything that implements the `RegistryIdentifier` interface as a key for a registry.

### Registry Usage - 1 Dimensional Data

An example of where you may use a Registry is to store the variants for a block of Cobblestone in Minecraft. First, we'll need to create an `enum` for the different variants.

??? info "View imports"

    ```java
    import org.dockbox.hartshorn.persistence.registry.RegistryIdentifier;
    ```
```java
public enum VariantIdentifier implements RegistryIdentifier {
   FULLBLOCK, STAIR, SLAB, WALL   
}
```

From there, we need to create a Registry. For this example, we'll be using a `String` to represent the different blocks, however, any type can be used.

You can add information to a Registry using primarily #addColumn and #addData. There is an important difference between the two methods, #addData will add the data to an existing column or create one if it doesn't exist, whereas #addColumn will replace an existing column if it has the same `RegistryIdentifier`. Both methods, though, can take in as many values as you want the column to contain and returns the instance of the Registry so that it can be easily chained.

??? info "View imports"

    ```java
    import org.dockbox.hartshorn.persistence.registry.Registry;
    ```
```java
Registry<String> cobblestoneVariants = new Registry<String>()
   .addColumn(VariantIdentifier.FULLBLOCK, "Cobblestone Fullblock", "Mossy Cobblestone Fullblock")
   .addColumn(VariantIdentifier.STAIR, "Cobblestone Stair")
   .addColumn(VariantIdentifier.SLAB, "Cobblestone Slab");

//Add data to an existing column.
cobblestoneVariants.addData(VariantIdentifier.STAIR, "Mossy Cobblestone Stair");
```

Columns in a registry can be retrieved using #getMatchingColumns, which takes in as many `RegistryIdentifiers` as you want and returns all the values of those columns as a single `RegistryColumn<V>`. You can also use all the existing methods of a `HashMap`, such as #get.

```java
List<String> variants = cobblestoneVariants.getMatchingColumns(VariantIdentifier.FULLBLOCK, VariantIdentifier.STAIR);
//"Cobblestone Fullblock", "Mossy Cobblestone Fullblock", "Cobblestone Stair", "Mossy Cobblestone Stair".

List<String> fullblockColumn = cobblestoneVariants.get(VariantIdentifier.FULLBLOCK);
//"Cobblestone Fullblock", "Mossy Cobblestone Fullblock"
```

You can also retrieve all values in a registry using #getAllData. 

```java
List<String> allVariants = cobblestoneVariants.getAllData();
```

It's important to remember that #getMatchingColumns and #getAllData returns a new `RegistryColumn<V>` and so filtering, or removing/adding elements doesn't impact the original Registry, whereas #get returns the `RegistryColumn<V>` used within the Registry and so these changes will be reflected there too.

You can also remove columns using #removeColumns, along with the existing HashMap methods like #remove.

```java
cobblestoneVariants.removeColumns(VariantIdentifier.STAIR, VariantIdentifier.SLAB);
cobblestoneVariants.remove(VariantIdentifier.FULLBLOCK);
```

That being said, none of this so far has been particularly profound and that's because the Registry thrives when we start storing hierarchical information. For example, simply knowing the variants of the cobblestone block isn't particularly useful, but it is far more useful to know the variants of many _different_ blocks.

### Registry Usage - 2 Dimensional Data

Multidimensional data - such as the variants of many different blocks - can be stored in a Registry by nesting Registries within each other, which is where the `RegistryColumn<V>` truly shines. Building on the previous example, Let's start by creating another `enum` for the block identifiers.

??? info "View imports"

    ```java
    import org.dockbox.hartshorn.persistence.registry.RegistryIdentifier;
    ```
```java
public enum BlockIdentifier implements RegistryIdentifier {
   COBBLESTONE, SANDSTONE
}
```

We can then create a nested Registry as shown below:

??? info "View imports"

    ```java
    import org.dockbox.hartshorn.persistence.registry.Registry;
    ```
```java        
Registry<Registry<String>> blockVariants = new Registry<>();
blockVariants.addColumn(BlockIdentifier.COBBLESTONE, cobblestoneVariants)
             .addColumn(BlockIdentifier.SANDSTONE, new Registry<String>()
                .addColumn(VariantIdentifier.FULLBLOCK, "Sandstone Fullblock")
                .addColumn(VariantIdentifier.STAIR, "Sandstone Stair")
                .addColumn(VariantIdentifier.SLAB, "Sandstone Slab"));
```

Alternatively, if the Registry is being created dynamically, it may be useful to leverage #getColumnOrCreate, which takes a `RegistryIdentifier` and an optional number of default values for the column to have if it doesn't exist and needs to be created.

??? info "View imports"

    ```java
    import org.dockbox.hartshorn.persistence.registry.Registry;
    ```
```java
Registry<Registry<String>> blockVariants = new Registry<>();

List<String> blocks = Arrays.asList(
   "Sandstone Fullblock", "Sandstone Stair", "Sandstone Slab", "Wood Fullblock", "Wood Stair", "Wood Slab");

for (String block : blocks) {
   TestIdentifier blockIdentifier = this.identifyBlockIdentifier(block);
   TestIdentifier variantIdentifier = this.identifyVariantIdentifier(block);

   blockVariants.getColumnOrCreate(blockIdentifier, new Registry<>())
      .first()
      .ifPresent(registry -> registry.addData(variantIdentifier, block));
}
```

Accessing information can be done using #getMatchingColumns in conjunction with `RegistryColumn<V>`'s #mapToSingleList. Using #mapToSingleList prevents you from ending up with Lists inside of Lists, instead, the resulting collection of that method is merged into a single `RegistryColumn<V>`. Some useful examples are demonstrated below:

Get all the cobblestone fullblocks in the Registry.
```java
List<String> cobblestoneFullblocks = blockVariants
   .getMatchingColumns(BlockIdentifier.COBBLESTONE)
   .mapToSingleList(r -> r.getMatchingColumns(VariantIdentifier.FULLBLOCK));
```

Get all the cobblestone variants in the Registry.
```java
List<String> cobblestoneVariants = blockVariants
   .getMatchingColumns(BlockIdentifier.COBBLESTONE)
   .mapToSingleList(r -> r.getAllData());
```

Get all the fullblocks in the registry
```java
List<String> fullblocks = blockVariants
   .getAllData()
   .mapToSingleList(r -> r.getMatchingColumns(VariantIdentifier.FULLBLOCK));
```

When adding data to a Registry - especially in a 2 Dimensional Registry - you must remember you are dealing with columns of values. Even though we've only ever added one Registry to each `BlockIdentifier` in the previous Registry, there is the potential for more to be added, which is something we need to take into consideration. What this means is that the result of:

```java
blockVariants.getMatchingColumns(BlockIdentifier.COBBLESTONE);
```

Is in fact a `RegistryColumn<V>`, which only contains the Registry for cobblestone's block variants. As a result, you cannot directly call the nested Registry's methods. Fortunately, `RegistryColumn<V>` adds several methods that enable you to safely and easily access these nested Registries. Let's say we want to add a `VariantIdentifier.WALL` column to the previous Registry; there are several ways this can be achieved depending on what your particular situation.

If you know that you want to add it to the first item in the `BlockIdentifier.COBBLESTONE` column.
```java
blockVariants.getMatchingColumns(BlockIdentifier.COBBLESTONE)
   .first()
   .ifPresent(r -> r.addData(VariantIdentifier.WALL, "Cobblestone Wall"));
```

If you're not sure where it is in the `BlockIdentifier.COBBLESTONE` column, however, you already know it contains the `VariantIdentifier.WALL` column.
```java
blockVariants.getMatchingColumns(BlockIdentifier.COBBLESTONE)
   .firstMatch(r -> r.containsColumns(VariantIdentifier.WALL))
   .ifPresent(r -> r.addData(VariantIdentifier.WALL, "Mossy Cobblestone Wall"));
```

If you know that it is present in the column, or want to add the same value to multiple columns, or don't care about adding the value if the column doesn't exist.
```java
blockVariants.getMatchingColumns(BlockIdentifier.COBBLESTONE)
   .forEach(r -> r.addData(VariantIdentifier.WALL, "Another Cobblestone Wall"));
```