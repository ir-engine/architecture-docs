# iR Engine documentation

<style>
:root {
  /* Dark mode colors (default/preferred) */
  --bg-primary: #1a1a1a;
  --bg-secondary: #2d2d2d;
  --bg-card: #2a2a2a;
  --bg-card-hover: #333333;
  --text-primary: #ffffff;
  --text-secondary: #b0b0b0;
  --text-muted: #888888;
  --border-color: #404040;
  --shadow-color: rgba(0, 0, 0, 0.3);
  --shadow-hover: rgba(0, 0, 0, 0.5);
  --accent-primary: #4A90E2;
  --accent-hover: #357ABD;
  --gradient-start: #2a2a2a;
  --gradient-end: #1f1f1f;
}

/* Light mode override */
@media (prefers-color-scheme: light) {
  :root {
    --bg-primary: #ffffff;
    --bg-secondary: #f8f9fa;
    --bg-card: #ffffff;
    --bg-card-hover: #f8f9fa;
    --text-primary: #212529;
    --text-secondary: #495057;
    --text-muted: #6c757d;
    --border-color: #dee2e6;
    --shadow-color: rgba(0, 0, 0, 0.1);
    --shadow-hover: rgba(0, 0, 0, 0.15);
    --accent-primary: #007bff;
    --accent-hover: #0056b3;
    --gradient-start: #ffffff;
    --gradient-end: #f8f9fa;
  }
}

.card-container {
  display: flex;
  flex-wrap: wrap;
  gap: 24px;
  justify-content: center;
  margin: 32px 0;
}

.card {
  background: linear-gradient(135deg, var(--gradient-start), var(--gradient-end));
  border: 1px solid var(--border-color);
  border-radius: 12px;
  padding: 28px 24px;
  text-align: center;
  flex: 1;
  min-width: 280px;
  max-width: 320px;
  box-shadow: 0 4px 12px var(--shadow-color);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  overflow: hidden;
}

.card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, #4A90E2, #50E3C2, #F5A623);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px var(--shadow-hover);
  background: var(--bg-card-hover);
  border-color: var(--accent-primary);
}

.card:hover::before {
  opacity: 1;
}

.card h3 {
  margin: 16px 0 12px 0;
  color: var(--text-primary);
  font-weight: 600;
  font-size: 1.25rem;
}

.card p {
  color: var(--text-secondary);
  margin: 12px 0 24px 0;
  line-height: 1.5;
  font-size: 0.95rem;
}

.button {
  display: inline-block;
  background: var(--accent-primary);
  color: var(--text-primary);
  padding: 12px 24px;
  text-decoration: none;
  border-radius: 8px;
  font-weight: 500;
  font-size: 0.9rem;
  transition: all 0.2s ease;
  border: none;
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(74, 144, 226, 0.3);
}

.button:hover {
  background: var(--accent-hover);
  color: var(--text-primary);
  text-decoration: none;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(74, 144, 226, 0.4);
}

.footer {
  text-align: center;
  margin-top: 48px;
  padding-top: 24px;
  border-top: 1px solid var(--border-color);
  color: var(--text-muted);
}

/* Icon styling for better dark mode compatibility */
.card div[style*="font-size: 3em"] {
  filter: drop-shadow(0 2px 4px var(--shadow-color));
  margin-bottom: 16px;
}

/* Ensure table styling works with dark mode */
table {
  color: var(--text-primary);
}

table td {
  border-color: var(--border-color);
}

/* Component grid styling */
.component-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin: 24px 0 40px 0;
  justify-content: center;
}

.component-card {
  flex: 0 1 calc(33.333% - 14px);
  min-width: 280px;
  background: linear-gradient(135deg, var(--gradient-start), var(--gradient-end));
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 20px;
  text-align: center;
  text-decoration: none;
  color: inherit;
  box-shadow: 0 3px 10px var(--shadow-color);
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  overflow: hidden;
}

@media (max-width: 1024px) {
  .component-card {
    flex: 0 1 calc(50% - 10px);
    min-width: 250px;
  }
}

@media (max-width: 640px) {
  .component-card {
    flex: 1 1 100%;
    min-width: auto;
  }
}

.component-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, #4A90E2, #50E3C2, #F5A623);
  opacity: 0;
  transition: opacity 0.25s ease;
}

.component-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 20px var(--shadow-hover);
  background: var(--bg-card-hover);
  border-color: var(--accent-primary);
  text-decoration: none;
  color: inherit;
}

.component-card:hover::before {
  opacity: 1;
}

.component-icon {
  font-size: 3em;
  margin-bottom: 12px;
  filter: drop-shadow(0 2px 4px var(--shadow-color));
}

.component-card h4 {
  margin: 8px 0;
  color: var(--text-primary);
  font-weight: 600;
  font-size: 1.1rem;
}

.component-card p {
  color: var(--text-secondary);
  margin: 8px 0 0 0;
  font-size: 0.9rem;
  line-height: 1.4;
}

/* Section headers */
h3 {
  color: var(--text-primary);
  margin: 32px 0 16px 0;
  font-weight: 600;
}

/* Component icon styling */
div[style*="font-size: 4em"] {
  filter: drop-shadow(0 2px 6px var(--shadow-color));
}
</style>

Welcome to the iR Engine documentation. This comprehensive guide covers the architecture, components, and systems that make up the Infinite Reality Engine.

## Documentation components

### Core components
<div class="component-grid">
  <a href="./01-core-engine/index.md" class="component-card">
    <div class="component-icon" style="color: #4A90E2;">⚙️</div>
    <h4>Core Engine</h4>
    <p>The central engine that powers the platform</p>
  </a>

  <a href="./02-entity-component-system/index.md" class="component-card">
    <div class="component-icon" style="color: #50E3C2;">🧩</div>
    <h4>Entity Component System</h4>
    <p>The foundational architecture</p>
  </a>

  <a href="./03-networking/index.md" class="component-card">
    <div class="component-icon" style="color: #F5A623;">🌐</div>
    <h4>Networking</h4>
    <p>Systems for multiplayer functionality</p>
  </a>

  <a href="./04-client-core/index.md" class="component-card">
    <div class="component-icon" style="color: #D0021B;">💻</div>
    <h4>Client Core</h4>
    <p>Client-side implementation</p>
  </a>

  <a href="./05-server-core/index.md" class="component-card">
    <div class="component-icon" style="color: #7ED321;">🖥️</div>
    <h4>Server Core</h4>
    <p>Server-side implementation</p>
  </a>
</div>

### Specialized components
<div class="component-grid">
  <a href="./06-physics-and-spatial-systems/index.md" class="component-card">
    <div class="component-icon" style="color: #F5A623;">🎯</div>
    <h4>Physics and Spatial Systems</h4>
    <p>Physics and transform systems</p>
  </a>

  <a href="./07-input-and-interaction/index.md" class="component-card">
    <div class="component-icon" style="color: #D0021B;">🎮</div>
    <h4>Input and Interaction</h4>
    <p>Input handling and user interactions</p>
  </a>

  <a href="./08-ui-framework/index.md" class="component-card">
    <div class="component-icon" style="color: #BD10E0;">🎨</div>
    <h4>UI Framework</h4>
    <p>UI components and XRUI system</p>
  </a>

  <a href="./09-visual-scripting/index.md" class="component-card">
    <div class="component-icon" style="color: #9013FE;">🔀</div>
    <h4>Visual Scripting</h4>
    <p>Node-based visual programming</p>
  </a>

  <a href="./10-world-editor/index.md" class="component-card">
    <div class="component-icon" style="color: #9013FE;">🌍</div>
    <h4>World Editor</h4>
    <p>3D world creation and editing tools</p>
  </a>

  <a href="./11-multiplayer-infrastructure/index.md" class="component-card">
    <div class="component-icon" style="color: #4A90E2;">🔗</div>
    <h4>Multiplayer Infrastructure</h4>
    <p>Instanceserver, WebRTC, and Agones</p>
  </a>

  <a href="./12-matchmaking-system/index.md" class="component-card">
    <div class="component-icon" style="color: #50E3C2;">🎲</div>
    <h4>Matchmaking System</h4>
    <p>Open match-based matchmaking</p>
  </a>

  <a href="./13-background-processing/index.md" class="component-card">
    <div class="component-icon" style="color: #7ED321;">⚡</div>
    <h4>Background Processing</h4>
    <p>Task server and background jobs</p>
  </a>
</div>

## Getting started

New to iR Engine? Choose your learning path:

<div class="card-container">
  <div class="card">
    <div style="font-size: 3em; margin-bottom: 10px;">🌱</div>
    <h3>For Beginners</h3>
    <p>Start here if you're new to the iR Engine:</p>
    <a href="./learning-paths.md#beginners" class="button">Beginner's Guide</a>
  </div>

  <div class="card">
    <div style="font-size: 3em; margin-bottom: 10px;">🚀</div>
    <h3>For Intermediate Developers</h3>
    <p>Already familiar with the basics?</p>
    <a href="./learning-paths.md#intermediate" class="button">Intermediate Guide</a>
  </div>

  <div class="card">
    <div style="font-size: 3em; margin-bottom: 10px;">🔧</div>
    <h3>For Advanced Developers</h3>
    <p>Looking to extend the engine?</p>
    <a href="./learning-paths.md#advanced" class="button">Advanced Guide</a>
  </div>
</div>

## About iR Engine

iR Engine is a powerful platform for creating immersive 3D experiences built on two core technologies:

- **Entity component system (ECS)** - The structural foundation for organizing game objects
- **Hyperflux** - The reactive state management system that powers the entire engine

[Learn more about iR Engine](./about.md).

## Additional resources

- [Learning paths](./learning-paths.md) - Recommended reading sequences
- [About iR Engine](./about.md) - Overview of features and capabilities
- [Official repository](https://github.com/ir-engine/ir-engine)
- [Developer documentation](https://github.com/ir-engine/developer-docs)

---

<div class="footer">
  <p>iR Engine documentation | <a href="./README.md">Usage guide</a></p>
</div>
