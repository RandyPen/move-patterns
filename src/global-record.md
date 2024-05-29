# Global Record

|||
|-|-|
| **Name** | Global Record |
| **Origin** | [Pika](https://github.com/RandyPen) |
| **Example** | [Movescriptions](https://github.com/movescriptions/movescriptions/blob/main/sui/sources/movescription.move#L233) / [Pika](https://github.com/RandyPen) |
| **Depends on** | None |
| **Known to work on** | Move |

## Summary

In many fully on-chain applications, users continuously creating new activities. The web frontend needs to promptly fetch and present these activities to all users. Relying on Event log searches for this data can be time-intensive and detract from the user experience.  

Implementing a shared Object to track these new activities can significantly expedite the frontend's retrieval speed, enhancing the overall user experience.  

If the activity records are only continually expanding, the **Global Record** structure should be designed to accommodate this growth.  
```move
public struct Deployer has key {
    id: UID,
    record: TableVec<ID>,
}
```

When launching new activities, include the Deployer as a parameter within the function call.  
```move
public fun new_game(deployer: &mut Deployer, ..., ctx: &mut TxContext) {
    let game = { id: object::new(ctx), ... };
    table_vec::push_back<ID>(&mut deployer.record, object::id(&game));
    transfer::share_object(game);
}
```

If activities are both initiated and eventually completed, the **Global Record** structure should be defined to encompass both the initiation and conclusion of these activities.
```move
public struct Deployer has key {
    id: UID,
    open: Table<ID, bool>,
    closed: TableVec<ID>,
}
```

In initiating new activities

```move
public fun new_game(deployer: &mut Deployer, ..., ctx: &mut TxContext) {
    let game = { id: object::new(ctx), ... };
    table::add<ID, bool>(&mut deployer.open, object::id(&game), true);
    transfer::share_object(game);
}
```

When terminating an activity
```move
public fun close_game(deployer: &mut Deployer, game: &mut Game, ..., ctx: &mut TxContext) {
    ...
    let _ = table::remove<ID, bool>(&mut deployer.open, object::id(game));
    table_vec::push_back<ID>(&mut deployer.closed, object::id(game));
    ...
}
```


## Examples
