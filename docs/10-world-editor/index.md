# World editor

## Overview

The World Editor is a comprehensive component of the iR Engine that provides a complete environment for creating and modifying 3D virtual worlds. It enables users to import, arrange, and configure various assets within scenes through an intuitive interface. By leveraging the Entity Component System (ECS) architecture and providing visual tools for manipulation and scripting, this component empowers creators to build complex interactive environments without extensive programming knowledge. This documentation explores the architecture, features, and workflows of the World Editor.

## Core components

The World Editor consists of several key components that work together to provide a complete editing experience:

### Editor panels and UI structure

The user interface is organized into specialized panels that:

- Provide focused tools for specific tasks
- Can be arranged and resized according to user preference
- Include viewport, hierarchy, inspector, and other specialized panels
- Support keyboard shortcuts and context menus for efficient workflows
- Integrate with the underlying editor systems

These panels create an intuitive and customizable editing environment.

### Asset handling and pipeline

The asset management system handles:

- Importing and processing various asset types (models, textures, etc.)
- Organizing assets in a browsable library
- Previewing assets before use
- Dragging and dropping assets into scenes
- Managing asset metadata and properties

This system streamlines the use of external resources in world creation.

### Scene operations and GLTF management

The scene management system enables:

- Creating, opening, and saving scene files
- Managing the hierarchy of entities in the scene
- Importing and exporting GLTF models
- Configuring scene-wide settings and properties
- Supporting undo/redo for scene modifications

This system forms the core of the world-building process.

### Entity Component System integration

The ECS integration provides:

- A component-based architecture for entity definition
- Visual editing of component properties
- Runtime behavior definition through components
- Hierarchical organization of entities
- Serialization and deserialization of entity data

This integration connects the editor to the underlying engine architecture.

### Gizmos

The gizmo system offers:

- Visual tools for manipulating objects in 3D space
- Transform controls (move, rotate, scale)
- Camera manipulation tools
- Visual feedback for selections and operations
- Customizable behavior for different editing contexts

These tools enable precise control over objects in the scene.

### Visual scripting system

The visual scripting integration provides:

- Node-based programming interface
- Visual representation of logic and data flow
- Integration with the entity component system
- Custom node creation and management
- Script debugging and testing capabilities

This system enables non-programmers to create complex behaviors.

### Editor control functions

The control functions handle:

- User input processing
- Command execution and routing
- Tool state management
- Selection and manipulation operations
- Keyboard and mouse interaction

These functions create a responsive and intuitive editing experience.

### Modal dialog management

The modal dialog system provides:

- Focused interaction for important decisions
- Standardized interface for user input
- Blocking of background interaction when necessary
- Consistent appearance and behavior across the editor
- Support for various dialog types (confirmation, input, settings)

This system ensures clear communication with users during critical operations.

### Editor state management

The state management system using Hyperflux:

- Maintains the current state of the editor
- Provides reactivity for UI updates
- Manages selection, tool modes, and editor preferences
- Coordinates between different editor systems
- Enables undo/redo functionality

This system creates a cohesive and responsive editing environment.

## Documentation chapters

1. [Editor panels and UI structure](01_editor_panels___ui_structure_.md)
2. [Asset handling and pipeline](02_asset_handling___pipeline_.md)
3. [Scene operations and GLTF management](03_scene_operations___gltf_management_.md)
4. [Entity Component System integration](04_entity_component_system__ecs____editor_integration_.md)
5. [Gizmos](05_gizmos_.md)
6. [Visual scripting system](06_visual_scripting_system_.md)
7. [Editor control functions](07_editor_control_functions_.md)
8. [Modal dialog management](08_modal_dialog_management_.md)
9. [Editor state management](09_editor_state_management__hyperflux__.md)

---


