# YAML Inheritance Rules

YAML prototype inheritance rules are confusing, so here is a guide.

## Inheritance

There are different ways datafields get merged or overwritten when inheriting from another yaml prototype and they can be a little confusing.
Let's look at a few examples:
```yml
# Define a few tags for testing purposes.
# Normally these should be in tags.yml.
- type: Tag
  id: TagA1

- type: Tag
  id: TagA2

- type: Tag
  id: TagB

- type: entity
  abstract: true
  id: ParentA
  components:
  - type: Tag
    tags:
    - TagA1
    - TagA2
  - type: MeleeWeapon # turn the entity into a weapon
    damage:
      types:
        Heat: 10

- type: entity
  abstract: true
  id: ParentB
  components:
  - type: Tag
    tags:
    - TagB
  - type: PointLight # make it glow
    color: green

- type: entity
  abstract: true
  id: ParentC
  components:
  - type: Sprite
    sprite: Objects/Fun/Plushies/hampter.rsi # change the sprite folder only, but not the state

# A simple item with a lizard sprite.
# A sprite needs both the 'sprite' datafield, which contains the path to the rsi folder the image file is in,
# and the 'state' datafield which is the name of the .png file itself.
- type: entity
  parent: BaseItem
  id: TestItem1
  name: test lizard 1
  components:
  - type: Sprite
    sprite: Objects/Fun/Plushies/lizard.rsi
    state: icon # this state name is the same for all basic plushie sprites

# The lizard sprite rsi is inherited first from TestItem1.
# The rsi is not overwritten by the hampter because it already exists in the first parent.
# So the resulting entity has a lizard sprite.
- type: entity
  parent: [ TestItem1, ParentC ]
  id: TestItem2
  name: test lizard 2

# If we inherit in this order the state will be taken from ParentC.
# TestItem1 will then add the state datafield, but not overwrite the rsi.
# The result will be a hampter.
- type: entity
  parent: [ ParentC, TestItem1]
  id: TestItem3
  name: test hampter 3

# This time we manually overwrite the inherited rsi.
# The inherited state remains unchanged.
# The result will be another hampter.
- type: entity
  parent: TestItem1
  id: TestItem4
  name: test hampter 4
  components:
  - type: Sprite
    sprite: Objects/Fun/Plushies/hampter.rsi

# This item will inherit the tags from ParentA, but not from ParentB.
# So it will have TagA1 and TagA2.
# The item will have both the PointLightComponent and the MeleeWeaponComponent and the corresponding datafields set in the parents.
- type: entity
  parent: [ TestItem1, ParentA, ParentB ]
  id: TestItem5
  name: test lizard 5

# To fix this and make the entity have all 3 tags we have to redefine the list manually.
- type: entity
  parent: [ TestItem1, ParentA, ParentB ]
  id: TestItem6
  name: test lizard 6
  components:
  - type: Tag
    tags: # this overwrites the inherited list
    - TagA1
    - TagA2
    - TagB
```
  

To summarize the inheritance rules:
  1. A datafield that is not set in yaml gets its default value from its C# definition. In C# all variables have a default value if not specified otherwise, for example `public bool SomeVariable;` will always be `false` (in other programming languages you may get random bits).
  2. If you inherit from multiple parents then components and their datafields are merged, but not overwritten. The order of inheritance matters.
  3. You can overwrite datafields by reassigning a new value in the child.
  4. If a datafield is overwritten, then the whole instance of the variable is reassigned. This means datatypes like lists (for example for tags) won't get merged, but replaced.
  5. You cannot remove components that are inherited from a parent. You will have to make another abstract parent instead to avoid copy pasting everything. 

```admonish warning
Common Mistake:
Rule 4 often gets overlooked for tags. If you add a new tag to an EntityPrototype make sure that
- the new list of tags for that entity contains all tags that were previously inherited.
- any child prototypes that have their own tags explicitly list the new tag as well.
```

## Abstract prototypes
If you got a base prototype that should not a fully functioning prototype on its own, but used for inheritance, then make sure to mark it as `abstract`. This will make sure it cannot be spawned by any means and it will be hidden from the F5 spawn menu.
 Example:
  ```yml
  # A simplified plushie that inherits from BaseItem and has incomplete SpriteComponent datafields.
  - type: entity
    abstract: true # should not be spawned since this prototype is incomplete
    parent: BaseItem
    id: BasePlushie
    components:
    - type: Sprite
      sprite: Objects/Fun/toys.rsi # we have a sprite, but not the state
  - type: EmitSoundOnUse # make it squeak when used
    sound:
      collection: ToySqueak

  # A plushie that can be spawned in the game.
  - type: entity
    parent: BasePlushie
    id: PlushieBee
    name: bee plushie
    components:
    - type: Sprite
      state: plushie_h # add a state (the sprite path is inherited from BasePlushie)

  # And another one.
  - type: entity
    parent: BasePlushie
    id: PlushieLizard # Weh!
    name: lizard plushie
    components:
    - type: Sprite
      state: plushie_lizard
  ```

If a prototype that is a fully functioning entity on its own should only be hidden from the F5 spawn menu, but can still be spawned through other means in-game, then you should be using The `HideSpawnMenu` category.
Examples are mind entities, objectives, or visual effects like this one used for flashbangs:
  ```
  - type: entity
    id: GrenadeFlashEffect
    categories: [ HideSpawnMenu ]
    components:
      - type: PointLight
        enabled: true
        radius: 5
        energy: 8
        netsync: false
      - type: LightFade
        duration: 0.5
      - type: TimedDespawn
        lifetime: 0.5
  ```
  This is a simple pointlight that quickly disappears and deletes itself. We use `categories: [HideSpawnMenu]` to make sure it does not show up in the spawn menu, but it can still be spawned using the `Spawn(protoId, coords)` method.
  This will also exclude it from several integration tests, which may otherwise fail for such an entity.

## AlwaysPushInheritance, NeverPushInheritance

## Making a prototype inheritable
The above examples were all for `EntityPrototype`s, but they work the same for any other type of prototype. You can define your own prototype in C# and make it store datafields. To make it inheritable in YAML you will have to implement the `` interface.
Example:
