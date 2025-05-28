# iR Engine documentation generator

This repository contains comprehensive documentation for the Infinite Reality Engine (iR Engine) codebase, along with the tools used to generate it. The documentation is designed to help developers understand the architecture and core concepts of the iR Engine.

## Documentation overview

The iR Engine documentation is located in the [docs](./docs/) directory and includes:

- Core engine components
- Entity component system
- Networking and multiplayer infrastructure
- Client and server implementations
- Specialized components like physics, input, UI, and more

[Explore the iR Engine documentation →](./docs/index.md)

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

### Option 1: View markdown files directly

You can browse the documentation directly by opening the Markdown files in any Markdown viewer:

1. Start with `docs/index.md` for a visual overview and navigation
2. Explore `docs/about.md` for information about iR Engine features
3. Check `docs/learning-paths.md` for guided learning experiences
4. Navigate to specific components by following the links

### Option 2: Host on a web server

For team access, you can host these files on any web server or Static Site Generator (SSG).

## Documentation generator tool

This documentation was created using PocketFlow, a documentation generator tool that:

1. Analyzes codebases to identify key abstractions
2. Determines relationships between components
3. Creates beginner-friendly tutorials explaining how the code works

## Getting started with the documentation generator

If you want to generate documentation for additional components or features:

1. Clone this repository
   ```bash
   git clone https://github.com/your-company/iR-Engine-Documentation
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
   # Replace "<GEMINI_API_KEY>" with your actual Gemini API key
   ```

   You can get a Gemini API key from [Google AI Studio](https://aistudio.google.com/app/apikey).

   The `.env` file is included in `.gitignore` so your API key won't be uploaded to GitHub.

4. Generate documentation for a component:
   ```bash
   # Analyze a local directory
   python main.py --dir /path/to/component --include "*.ts" "*.tsx" "*.js" "*.jsx" "*.go" --exclude "node_modules/*" "tests/*" "dist/*" --name "iR Engine - Component Name" --output output --max-abstractions 8
   ```

5. Move the generated documentation to the docs directory:
   ```bash
   # Copy the generated documentation to the docs directory
   cp -r output/* docs/
   ```

6. Update the following files to include links to the new documentation:
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

To run this project in a Docker container:

1. Build the Docker image
   ```bash
   docker build -t ir-engine-docs-generator .
   ```

2. Run the container
   ```bash
   # Load environment variables from .env file
   docker run -it --rm \
     --env-file .env \
     -v "/path/to/your/component":/app/code_to_analyze \
     -v "$(pwd)/output":/app/output \
     ir-engine-docs-generator --dir /app/code_to_analyze
   ```

## Acknowledgments

This documentation generator is based on [PocketFlow Tutorial Codebase Knowledge](https://github.com/The-Pocket/PocketFlow-Tutorial-Codebase-Knowledge), a tool that uses AI to analyze codebases and generate comprehensive documentation.

## License

This documentation inherits the license of the iR Engine project (CPAL).
