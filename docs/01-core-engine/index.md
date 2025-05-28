# Core engine

The iR Engine Core Engine is a foundational framework for building 3D interactive experiences. It uses an Entity-component-system (ECS) architecture for managing game objects and their behaviors, coupled with the Hyperflux library for state management. The engine provides specialized systems for handling assets (like 3D models and textures), structuring and rendering complex 3D scenes, managing lifelike avatars with animations and IK, enabling rich interactions (like grabbing or mounting objects), and creating game logic through a visual scripting interface.

## Architecture overview

The Core Engine consists of several interconnected systems that work together to provide a complete framework for interactive applications:

```mermaid
flowchart TD
    A0["Engine module system"]
    A1["Asset management system"]
    A2["Scene graph & rendering abstraction"]
    A3["Avatar system"]
    A4["ECS (Entity-component-system) & state management (Hyperflux)"]
    A5["Interaction system"]
    A6["Visual scripting system"]
    A0 -- "Initializes core architecture" --> A4
    A0 -- "Integrates" --> A1
    A0 -- "Integrates" --> A2
    A0 -- "Integrates" --> A3
    A0 -- "Integrates" --> A5
    A0 -- "Integrates" --> A6
    A1 -- "Uses for state" --> A4
    A2 -- "Loads scene assets via" --> A1
    A2 -- "Represents scene using" --> A4
    A3 -- "Loads avatar assets via" --> A1
    A3 -- "Updates avatar presentation in" --> A2
    A3 -- "Built upon" --> A4
    A5 -- "Operates on objects in" --> A2
    A5 -- "Enables actions for" --> A3
    A5 -- "Defines interactions using" --> A4
    A6 -- "Scripts avatar logic" --> A3
    A6 -- "Manipulates" --> A4
```

## Documentation chapters

1. [ECS (Entity-component-system) & state management (Hyperflux)](01_ecs__entity_component_system____state_management__hyperflux__.md)
2. [Asset management system](02_asset_management_system_.md)
3. [Scene graph & rendering abstraction](03_scene_graph___rendering_abstraction_.md)
4. [Avatar system](04_avatar_system_.md)
5. [Interaction system](05_interaction_system_.md)
6. [Visual scripting system](06_visual_scripting_system_.md)
7. [Engine module system](07_engine_module_system_.md)

---


