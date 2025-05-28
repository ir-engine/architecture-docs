# About iR Engine

iR Engine is a powerful platform for creating, hosting, and experiencing 3D websites and immersive experiences.

## Features

iR Engine includes features for:

- Creating and hosting 3D worlds and experiences
- Social features like avatars, chat, groups, and friends
- Complete world editing and administration
- 2D, 3D, and XR avatars with full inverse kinematics and facial expressions
- Fully networked physics using Rapier Physics
- Voice and video over WebRTC
- Reactive state management with Hyperflux
- And much more!

## Architecture overview

The iR Engine architecture is built on two fundamental pillars:

1. **Entity component system (ECS)** - The structural foundation that organizes game objects and their behaviors
2. **Hyperflux** - The reactive state management system that enables data flow throughout the engine

These core technologies work together to create a flexible, modular, and reactive architecture. The documentation is then divided into two main categories:

### Core components

These components form the foundation of the iR Engine:

1. **Core engine** - The central engine that powers the platform, including scene management, rendering, and core systems.

2. **Entity component system** - The foundational architecture that organizes game objects and their behaviors.

3. **Networking** - The systems that enable multiplayer functionality and real-time communication (see [Networking](./03-networking/index.md)).

4. **Client core** - The client-side implementation that handles user interaction, UI, and client-specific logic.

5. **Server core** - The server-side implementation that manages users, worlds, and backend services.

### Specialized components

These components provide specialized functionality built on top of the core components:

1. **Multiplayer infrastructure** - Detailed exploration of the instanceserver, WebRTC networking, and Agones integration.

2. **Matchmaking system** - In-depth look at the Open Match-based matchmaking system.

3. **Physics and spatial systems** - Technical details of the physics and spatial systems.

4. **Input and interaction** - Comprehensive guide to input handling and interactions.

5. **Background processing** - Overview of the taskserver and background processing systems.

6. **Visual scripting** - Detailed exploration of the node-based visual programming system.

7. **UI framework** - In-depth look at the UI components and XRUI system.

8. **World editor** - Comprehensive guide to the 3D world creation and editing tools.

## Documentation structure

Each component's documentation contains:

1. An overview of the component's purpose and functionality
2. A diagram showing the relationships between key abstractions
3. Detailed chapters explaining each core concept
4. Code examples and explanations

## Documentation types

The documentation is divided into two main types:

1. **Core documentation** - Provides an overview of the fundamental architecture and systems
2. **Specialized documentation** - Offers deeper technical insights into specific subsystems

For more information on how to navigate through the documentation based on your experience level, see the [Learning paths](./learning-paths.md).

---

[Back to home](./index.md)
