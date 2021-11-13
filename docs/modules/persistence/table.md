# Tables [:fontawesome-brands-github:](https://github.com/GuusLieben/Hartshorn/tree/develop/hartshorn-persistence/src/main/java/org/dockbox/hartshorn/persistence/table){:target="_blank"}

Sometimes it can be useful to access data in a relational model. For example when transforming database values to Java objects. Often this data will have a relational structure, which can be tedious to perform using DTO's if the dataset isn't large enough to warrant such transactions. The `Table` data structure provides an easy and efficient way to store database tables which can be joined, filtered, sorted, and more.

## Column Identifiers
A `Table` stores data in a similar fashion to regular SQL tables. Each `Table` has a set of columns which are identified by `ColumnIdentifier<T>`s. The `ColumnIdentifier<T>` indicates the identifier, but also the data type of a given column. As `ColumnIdentifier<T>` holds a generic type, it cannot currently be implemented by enums. To resolve this Hartshorn offers a `ColumnIdentifierImpl<T>` type, which can be defined as follows:
??? info "View imports"

    ```java
    import org.dockbox.hartshorn.persistence.table.ColumnIdentifier;
    import org.dockbox.hartshorn.persistence.table.ColumnIdentifierImpl;
    ```
```java
public class Identifiers {
    public static final ColumnIdentifier<Object> IDENTIFIER = new ColumnIdentifierImpl<>("identifier", Object.class);
}
```
You can use anything that implements `ColumnIdentifier` as a identifier.

## Table Usage
An example of where you may use a Table is when loading moderation data of players, looking for a specific player's earlier moderation. First we'll need to create the tables which will be present. This will be a table with the name and UUID of all known players, and a table with the UUID of a player and the moderation actions performed on the player. To create the table, we'll first need to have identifiers for the columns.
```java
public class Identifiers {
    public static final ColumnIdentifier<String> NAME = new ColumnIdentifierImpl<>("name", String.class);
    public static final ColumnIdentifier<String> REASON = new ColumnIdentifierImpl<>("reason", String.class);
    public static final ColumnIdentifier<UUID> UNIQUE_ID = new ColumnIdentifierImpl<>("uniqueId", UUID.class);
    public static final ColumnIdentifier<ModerationType> MODERATION_TYPE = new ColumnIdentifierImpl<>("modType", ModerationType.class);
}
```
From there, we need to create a Table. We'll use the newly created column identifiers for this.
??? info "View imports"

    ```java
    import org.dockbox.hartshorn.persistence.table.Table;
    ```
```java
Table players = new Table(Identifiers.NAME, Identifiers.UNIQUE_ID);
Table actions = new Table(Identifiers.UNIQUE_ID, Identifiers.MODERATION_TYPE, Identifiers.REASON);
```
Before using our tables, we'll first need to populate them with data. You can add data to a Table using `#addRow`. This method has three ways of inserting data:
- Using a pre-made `TableRow`
- Using ordered parameters
- Using custom objects

### Adding Data: TableRow
A `TableRow` represents a row inside a table, but is not bound to a specific `Table` instance. However, a `TableRow` can only be added to a `Table` if it contains the same columnds as the table. To add a value to a row, you use `#addValue`.
??? info "View imports"

    ```java
    import org.dockbox.hartshorn.persistence.table.TableRow;
    ```
```java
TableRow playerOne = new TableRow();
playerOne.addValue(Identifiers.NAME, "PlayerOne");
playerOne.addValue(Identifiers.UNIQUE_ID, UUID.of(...));
```
Once the row contains the correct columns, it can be added to any table with the same columns.
```java
players.addRow(playerOne);
```

### Adding Data: Ordered Parameters
As columns are ordered inside a `Table`, it is also possible to dynamically create a `TableRow` using ordered values. `#addRow` accepts varargs objects and can attempt to construct a `TableRow` using these values. In our 'players' table we first provided the identifier `NAME` and then the identifier `UNIQUE_ID`.
```java
players.addRow("PlayerTwo", UUID.of(...));
```
As long as the parameters are of the correct data types, a row is automatically created and inserted into the table.

### Adding Data: Custom Objects
As rows often represent information which can be represented by a object, it is also possible to provide a custom object. The table then uses the fields inside that type to generate a `TableRow`. Columns are obtained using their identifier, for example `NAME` has a identifier `"name"`.
```java
public class User {
    private String name;
    private UUID uniqueId;
    //...
}
```
In this case our user can be added as follows.
```java
players.addRow(new User("playerThree", UUID.of(...));
```
However, what if we can't change the field names of `User` and it stores the `name` as `playerName`. In that case we can annotate the field using `@Property`.
??? info "View imports"

    ```java
    import org.dockbox.hartshorn.core.annotations.Property;
    ```
```java
public class User {
    @Property("name")
    private String playerName;
    private UUID uniqueId;
    //...
}
```
In this case the column will be looked up using the provided identifier, instead of the field name.

### Advanced Actions: Join
Often table data is normalized, making it hard to quickly combine related data. `Table` makes this easy using `#join`. To join two tables together we'll need to decide which column we use to look for matches. In our moderation tables this would be `UNIQUE_ID`. Additionally we'll want to decide two more things: which data to prioritize if both tables hold the same columns (e.g. `NAME`), and whether or not we want to populate empty rows with `null` if no matches are found.
??? info "View imports"

    ```java
    import org.dockbox.hartshorn.persistence.table.Table;
    ```
```java
Table actionsWithPlayerInformation = players.join(actions, Identifiers.UNIQUE_ID, Merge.PREFER_ORIGINAL, true);
```

### Advanced Actions: Where
Now that we have combined the moderation actions with the player name, you may want to filter out results for a specific user. This can be done using `#where`. The method takes a column identifier, and a value the entry has to match to be returned. This value has to be of the same type as the `ColumnIdentifier` data type. If we wish to get all moderation actions performed on user 'PlayerOne' we can use the following.
??? info "View imports"

    ```java
    import org.dockbox.hartshorn.persistence.table.Table;
    ```
```java
Table playerOneActions = actionsWithPlayerInformation.where(Identifiers.NAME, "PlayerOne");
```

### Advanced Actions: Select
As we already knew the name of the player, and we were only looking for the moderation actions performed on the player, we now want to select only the moderation actions and the reason they were performed from our `Table`. This way we only keep relevant information in our module, and remaining data can be disposed of. This can be done using `#select`, taking the identifiers for each column we want to keep.
??? info "View imports"

    ```java
    import org.dockbox.hartshorn.persistence.table.Table;
    ```
```java
Table playerOneActionReasons = playerOneActions.select(Identifiers.MODERATION_TYPE, Identifiers.REASON);
```