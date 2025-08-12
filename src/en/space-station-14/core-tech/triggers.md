# Triggers

The trigger system is designed to offer small, reusable building blocks which can be used entirely in yaml to create a variety of effects, and allowing basic event subscriptions without having to touch any C# code.
Originally it was only used for grenades and modular payloads, but more effects have been added over time. Later they were also used for subdermal implants.
In the recent [trigger refactor](https://github.com/space-wizards/space-station-14/pull/39034) the system was made more flexible to allow multiple triggers on the same entity, the existing components were standardized and predicted, and conditions were de-hardcoded from them. After that [a lot more trigger components](https://github.com/space-wizards/space-station-14/issues/39354) were added for yaml contributors to work with

## Components
The three basic types of building blocks are:
#### Trigger Components
These cause an entity to be triggered when a certain event occurs. Most of them inherit from the abstract `BaseTriggerOnXComponent`.
Examples:
- `TriggerOnUseComponent`: Equivalent to `UseInHandEvent`. A player holds the entity with this component and pressed `Z` to use it.
- `TriggerOnSpawnComponent`: Equivalent to `MapInitEvent`. The entity has been spawned and all its components have been initialized.
- `TriggerOnCollideComponent`: Equivalent to `StartCollideEvent`. The entity has a physics collision with another object, for example when it hits a wall. This can also be non-hard collisions without momentum changes, meaning two entities are overlapping.
- `TriggerOnExaminedComponent`. Equivalent to `ExaminedEvent`. A player shoft clicked on the entity to examine it.
- `TriggerOnVerbComponent`. A player used a right click verb on the entity. The text to be displaced in the verb can be specified in a datafield.

#### Trigger Effect Components
These do something either to the entity that has been triggered or to the entity that caused the trigger to occurr. Which of these two is the target of the effect can always be selected by setting the `TargetUser` datafield. Most of these components inherit from the abstract `BaseXOnTriggerComponent`.
Examples:
- `SpawnOnTriggerComponent`: Spawns an entity prototype at the target's location.
- `EmitSoundOnTrigger`: Play a sound at the target's location.
- `AlertLevelChangeOnTriggerComponent`: Set the station's alert level when triggered.
- `GibOnTrigger`: Gib the target.
- `PolymorphOnTrigger`: Polymorph the target into something else.
- `SpeakOnTrigger`: Make the target say something.
- `AddComponentsOnTrigger`: Add some components to the target.
- `DnaScrambleOnTriggerComponent`: Apply the effect of the DNA scrambler to the target entity, which gives it a new identity and appearance if it is a humanoid.
and many more!

#### Trigger Conditions
If the entity was triggered these will be checked before any effects are activated, and if not true then then trigger is cancelled. Most of these inherit from the abstract `BaseTriggerConditionComponent`.
Examples:
- `UseDelayTriggerConditionComponent`: Cancels the trigger if the entity has an active UseDelay. This is useful in combination with `UseDelayOnTriggerComponent` and `UseDelayComponent` to prevent the trigger from being spammed.
- `WhitelistTriggerConditionComponent`: Can check the user of the trigger for both a whitelist and a blacklist, making sure that only certain entities can activate the trigger.
- `RandomChanceTriggerConditionComponent`: The trigger will only activate with a random chance.

#### Others
There are a few other useful trigger related components:
- `TimerTriggerComponent`: Starts a timer when activated by a trigger, and will cause another trigger when that timer is up.
- `TwoStageTriggerComponent`: Adds a few components to the entity when triggered, and triggers the entity again after a certain delay. While the same effect can be created by combining other trigger components, this one was kept for ease of use and backwards compability.
- `RepeatedTriggerComponent`: Repeatedly triggers the entity on a certain interval.

## Why use triggers?
TODO: Examples for components to replace.
Examples for actions and implants, link PR.

## Trigger Keys
A huge restriction to triggers used to be that triggering an entity always activated all effects at once. With the trigger refactor this was changed so that you can now have multiple independent trigger and trigger effect pairs on the same entity. This works by giving triggers a key string that is compared when triggering. Triggers always output one single key string, where trigger effects and conditions can have multiple keys that will activate them. This causes the yaml syntax to be a little different for both, since one datafield is of type `string?` and the other is of type `List<string>`. The key `null` always serves as a wildcard and will activate any trigger key.
```yaml
- type: entity
# example: trigger one pair of trigger keys

# The example below is identical to the one above.
# The datafields for the keys always default to "trigger", meaning in most single-trigger cases you can just keep that default value.
# example: same as above, but default values

# If you want to
# for more complicated combinations you need to specify separate keys
# example: multiple independent triggers on the same entity
```

## Examples

## Future plans:
- Action support, convert a lot of actions to use triggers so they are more reusable
- Use for the in-game tutorial
- Broadcast triggers
- Combine triggers with XAE triggers and possibly entity effects
