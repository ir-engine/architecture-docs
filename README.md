# iR Engine architecture documentation

This repository contains the official architecture documentation for the Infinite Reality Engine (iR Engine), along with the tools used to generate and maintain it. The documentation provides comprehensive coverage of the engine's architecture, core concepts, and component relationships.

## Documentation overview

The iR Engine architecture documentation is located in the [docs](./docs/) directory and includes:

- Core engine components and foundational systems
- Entity component system architecture
- Networking and multiplayer infrastructure
- Client and server implementations
- Specialized components including physics, input, UI, visual scripting, and more

[📖 Explore the iR Engine architecture documentation →](./docs/index.md)

## Documentation structure

The documentation is organized into two main categories:

1. **Core components** - Fundamental architecture and systems of the iR Engine
2. **Specialized components** - Deeper technical insights into specific subsystems

Each section contains:
- An overview of the component
- A diagram showing relationships between key abstractions
- Detailed chapters explaining each core concept
- Code examples and explanations

## How to use the documentation

### Option 1: View on Archbee (recommended)

The documentation is professionally hosted on Archbee with enhanced navigation, search, and visual features:

**🌐 [View iR Engine architecture documentation on Archbee →](https://docs.ir.world/architecture_docs)**

This hosted version includes:
- Beautiful visual navigation with color-coded component cards
- Full-text search across all documentation
- Responsive design for desktop and mobile
- Real-time updates when documentation changes
- Professional presentation optimized for teams

### Option 2: View markdown files directly

You can also browse the documentation directly by opening the Markdown files in any Markdown viewer:

1. Start with `docs/index.md` for a visual overview and navigation
2. Explore `docs/about.md` for information about iR Engine features
3. Check `docs/learning-paths.md` for guided learning experiences
4. Navigate to specific components by following the links

## Documentation generator tool

This documentation was created using PocketFlow, a codebase analysis tool that:

1. Analyzes codebases to identify key architectural concepts
2. Determines relationships between components
3. Creates comprehensive tutorials explaining how the code works

## Getting started with the documentation generator

If you want to generate documentation for additional iR Engine components or update existing documentation:

1. Clone this repository
   ```bash
   git clone https://github.com/ir-engine/architecture-docs.git
   cd architecture-docs
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Set up your API key in the `.env` file:
   ```bash
   # Copy the sample environment file
   cp .env.sample .env

   # Edit the .env file and add your API key
   # Replace "<GEMINI_API_KEY>" with your actual API key
   ```

   You can get an API key from [Google AI Studio](https://aistudio.google.com/app/apikey).

   The `.env` file is included in `.gitignore` so your API key won't be uploaded to GitHub.

4. Generate documentation for an iR Engine component:
   ```bash
   # Example: Analyze a specific iR Engine package
   python main.py --dir /path/to/ir-engine/packages/engine --include "*.ts" "*.tsx" "*.js" "*.jsx" --exclude "node_modules/*" "tests/*" "dist/*" --name "iR Engine - Engine Core" --output output --max-abstractions 8
   ```

5. Move the generated documentation to the docs directory:
   ```bash
   # Copy the generated documentation to the appropriate numbered directory
   cp -r output/* docs/
   ```

6. Update the navigation files to include links to the new documentation:
   - `docs/index.md` - Add a visual card for the new component
   - `docs/about.md` - Add the component to the architecture overview
   - `docs/learning-paths.md` - Add the component to relevant learning paths

## Command line options

The documentation generator supports the following command line options:

- `--repo` or `--dir` - Specify either a GitHub repo URL or a local directory path (required, mutually exclusive)
- `-n, --name` - Project name (optional, derived from URL/directory if omitted)
- `-t, --token` - GitHub token (or set GITHUB_TOKEN environment variable)
- `-o, --output` - Output directory (default: ./output)
- `-i, --include` - Files to include (e.g., "`*.py`" "`*.js`")
- `-e, --exclude` - Files to exclude (e.g., "`tests/*`" "`docs/*`")
- `-s, --max-size` - Maximum file size in bytes (default: 100KB)
- `--language` - Language for the generated documentation (default: "english")
- `--max-abstractions` - Maximum number of abstractions to identify (default: 10)
- `--no-cache` - Disable LLM response caching (default: caching enabled)

For more detailed information about using the documentation generator, including advanced configuration examples, customization options, and troubleshooting tips, please see the [documentation generator guide](./documentation-generator-guide.md).

## Running with Docker

To run the documentation generator in a Docker container:

1. Build the Docker image
   ```bash
   docker build -t ir-engine-architecture-docs .
   ```

2. Run the container to generate documentation for iR Engine components
   ```bash
   # Load environment variables from .env file
   docker run -it --rm \
     --env-file .env \
     -v "/path/to/ir-engine/packages/component":/app/code_to_analyze \
     -v "$(pwd)/output":/app/output \
     ir-engine-architecture-docs --dir /app/code_to_analyze
   ```

## Acknowledgments

This documentation and generation tooling is based on [PocketFlow Tutorial Codebase Knowledge](https://github.com/The-Pocket/PocketFlow-Tutorial-Codebase-Knowledge), a tool for analyzing codebases and generating comprehensive documentation.

## Contributing

To contribute to the iR Engine architecture documentation:

1. Fork this repository
2. Create a feature branch for your documentation updates
3. Generate or update documentation using the included tools
4. Submit a pull request with your changes

For questions about the iR Engine architecture or to request documentation for specific components, please open an issue.

## License

This documentation inherits the license of the iR Engine project (CPAL). The documentation generation tools maintain their original licensing from the PocketFlow project.
