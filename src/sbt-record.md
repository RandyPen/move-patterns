# SBT Record

|||
|-|-|
| **Name** | SBT Record |
| **Origin** | [Pika](https://github.com/RandyPen) |
| **Example** |  |
| **Depends on** | None |
| **Known to work on** | Move |

## Summary

In many fully on-chain applications, it is necessary to record the activities that users have participated in, to help them remember which activities they have engaged in and to allow them to check results or claim rewards.

Soulbound Tokens (SBTs) can be distributed to users, encapsulating a record of their engagement with various activities.

Define SBT.
```move
public struct Ticket has key {
    id: UID,
    record: Table<ID, bool>,
}
```

When users participate in activities.

```move
public fun participant_activity(ticket: &mut Ticket, game: &mut Game, ...) {
    ...
    if (!table::contains(&ticket.record, object::id(&game))) {
        table::add<ID, bool>(&mut ticket.record, object::id(&game), true);
    };
    ...
} 
```

Upon users redeeming their rewards for an activity, the participation record should be archived or removed.

## Examples

