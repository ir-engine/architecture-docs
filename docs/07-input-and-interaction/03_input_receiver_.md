# Input receiver

## Overview

The input receiver component enables game entities to listen for and respond to user inputs. It serves as the bridge between the raw input data captured by input sources and the game logic that determines how entities should react to player actions.

## InputComponent

In the iR Engine, input receivers are implemented through the `InputComponent`. When attached to an entity, this component allows it to:

1. **Define semantic actions**: Map physical inputs (like specific keys or buttons) to meaningful actions in the game context
2. **Specify input sources**: Determine which input devices the entity should listen to
3. **Query input states**: Check the current state of defined actions to trigger appropriate responses

### Component structure

The `InputComponent` contains several key properties:

| Property | Description |
|----------|-------------|
| `buttonBindings` | Maps semantic action names to physical button inputs |
| `inputSources` | References to entities with `InputSourceComponent` that this entity listens to |
| `activationDistance` | How close an input pointer needs to be to interact with this entity |
| `autoCapture` | Whether the component automatically captures input focus when receiving input |

### Example configuration

```typescript
// Conceptual structure of an InputComponent on an interactive entity
InputComponent: {
  // Map semantic actions to physical inputs
  buttonBindings: {
    "Interact": [KeyboardButton.KeyE, MouseButton.PrimaryClick],
    "AlternativeAction": [KeyboardButton.KeyQ]
  },
  // Which input source entities to listen to
  inputSources: [keyboardEntityId, mouseEntityId],
  
  // Additional configuration
  activationDistance: 2,
  autoCapture: false
}
```

## Implementation example

The following example demonstrates how to implement an interactive door using the `InputComponent`:

### 1. Adding the component to an entity

```typescript
// Add InputComponent to a door entity
const doorEntity = createEntity();
setComponent(doorEntity, InputComponent, {
  buttonBindings: {
    "Interact": [KeyboardButton.KeyE, MouseButton.PrimaryClick]
  }
  // inputSources typically populated by another system
  // or default to global input sources
});
```

### 2. Checking for input in game logic

```typescript
// In a system or script that updates the door
const doorInputComponent = getComponent(doorEntity, InputComponent);

// Get the current state of all bound buttons for this entity
const buttons = InputComponent.getButtons(doorEntity); 

if (buttons.Interact?.down) { // Check if "Interact" was just pressed this frame
  console.log("Player pressed Interact! Opening door...");
  // Code to open the door
}
```

## Querying axis inputs

In addition to button states, the `InputComponent` can also query continuous axis values:

```typescript
// In InputComponent buttonBindings:
// "AimHorizontal": [MouseAxis.X]

// In game logic:
const axes = InputComponent.getAxes(someEntity);

if (buttons.AimButton?.pressed) { // If an "AimButton" is held
  const lookSpeed = 0.1;
  const horizontalRotation = axes.AimHorizontal * lookSpeed;
  // Apply horizontalRotation to the entity
}
```

## Technical implementation

The `InputComponent.getButtons()` method is the primary interface for querying input states. When called, it performs several operations:

1. **Locate the input entity**: Find the entity that holds the relevant `InputComponent`
2. **Retrieve bindings**: Get the button mappings for the requested semantic actions
3. **Check input sources**: Iterate through the associated input sources
4. **Query raw states**: For each input source, check the state of the mapped buttons
5. **Combine results**: Aggregate the state information to determine the overall state of each semantic action

```mermaid
sequenceDiagram
    participant GameLogic as Game Logic
    participant Helper as InputComponent.getButtons()
    participant EntityIC as InputComponent (on Entity)
    participant KeyboardISC as InputSourceComponent (Keyboard)
    participant MouseISC as InputSourceComponent (Mouse)

    GameLogic->>Helper: Get button states for entity
    Helper->>EntityIC: Get bindings for "Interact"
    EntityIC-->>Helper: "Interact" maps to [KeyE, PrimaryClick]
    Helper->>EntityIC: Get inputSources
    EntityIC-->>Helper: [KeyboardEntity, MouseEntity]
    
    Helper->>KeyboardISC: Is KeyE active?
    KeyboardISC-->>Helper: Yes, KeyE.down is true
    
    Note over Helper: KeyE is active, so "Interact" is active
    Helper-->>GameLogic: Returns proxy; "Interact.down" is true
    GameLogic->>GameLogic: Process "Interact" action
```

### Code implementation

```typescript
// Simplified from: components/InputComponent.ts

export const InputComponent = defineComponent({
  name: 'InputComponent',
  // Schema defines buttonBindings, inputSources, cachedButtons, etc.

  getButtons<BindingsType extends InputButtonBindings = typeof DefaultButtonBindings>(
    entityContext: Entity,
    inputBindings: BindingsType = DefaultButtonBindings as any,
    autoCapture = true
  ) {
    // Find the entity with the InputComponent
    const inputEntity = InputComponent.getInputEntity(entityContext); 
    if (inputEntity === UndefinedEntity) return {} as ButtonStateMap<BindingsType>;
    
    const input = getMutableComponent(inputEntity, InputComponent);

    // Update bindings if new ones are provided
    if (inputBindings) {
      for (const binding of Object.keys(inputBindings)) {
        if (!input.buttonBindings[binding].value) 
          input.buttonBindings[binding].set(inputBindings[binding] as any);
      }
    }
    input.autoCapture.set(autoCapture);
    
    // Return a proxy object that dynamically checks input states
    return input.buttons.get(NO_PROXY_STEALTH) as ButtonStateMap</*...*/>;
  },
  
  // Additional methods omitted for brevity
});
```

The `buttons` field in an `InputComponent` is implemented as a dynamic proxy. When accessing a property like `buttons.Interact`, the proxy:

1. Checks the `buttonBindings` to find the associated physical buttons
2. Iterates through the `inputSources` to check the state of those buttons
3. Considers whether the input has already been consumed by another system
4. Returns the aggregated state for the semantic action

This proxy mechanism provides a convenient interface while handling complex mapping and state checking behind the scenes.

## Benefits of the input receiver approach

The `InputComponent` provides several advantages:

1. **Abstraction**: Game logic can work with semantic actions rather than specific physical inputs
2. **Flexibility**: Input bindings can be changed without modifying the core game logic
3. **Device independence**: The same interaction can work across different input devices
4. **Centralization**: Input handling logic is consolidated in a single component

## Next steps

With an understanding of how entities can listen for and respond to user inputs, the next chapter explores how the input system coordinates the overall input processing pipeline.

Next: [Input system](04_input_system_.md)

---


