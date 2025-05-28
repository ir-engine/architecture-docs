# Documentation generator guide

This guide explains how to use the documentation generator tool to create comprehensive documentation for your codebase. The tool (based on PocketFlow) analyzes your code, identifies key abstractions, and creates beginner-friendly tutorials that explain how your code works.

## Table of contents

1. [Overview](#overview)
2. [Installation](#installation)
3. [Basic usage](#basic-usage)
4. [Configuration options](#configuration-options)
5. [Advanced configuration examples](#advanced-configuration-examples)
6. [How the tool works](#how-the-tool-works)
7. [Customizing the generation process](#customizing-the-generation-process)
8. [Troubleshooting](#troubleshooting)
9. [Example usage](#example-usage)

## Overview

PocketFlow is a tool that uses AI to analyze codebases and generate comprehensive documentation. It:

- Crawls GitHub repositories or local directories
- Identifies key abstractions and their relationships
- Creates structured tutorials with chapters
- Generates visual diagrams of code relationships
- Supports multiple languages

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/your-company/PocketFlow-Tutorial-Codebase-Knowledge
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Set up your API key in the `.env` file or directly in `utils/call_llm.py`:
   ```
   GEMINI_API_KEY=your-api-key-here
   ```

## Basic usage

### Analyzing a GitHub repository

```bash
python main.py --repo https://github.com/username/repo --include "*.py" "*.js" --exclude "tests/*" --name "Project Name"
```

### Analyzing a local directory

```bash
python main.py --dir /path/to/your/codebase --include "*.py" --exclude "*test*" --name "Project Name"
```

### Output

The generated documentation will be saved in the `output` directory (or the directory specified with `--output`). The structure will be:

```
output/
├── index.md             # Main entry point with visual navigation
├── about.md            # Overview of the project
├── learning-paths.md   # Recommended learning sequences
├── README.md           # Usage instructions
└── component-name/     # Directories for each component
    ├── index.md        # Component overview
    └── 01_chapter.md   # Individual chapters
```

## Configuration options

PocketFlow offers several configuration options:

| Option | Description | Default |
|--------|-------------|---------|
| `--repo` | GitHub repository URL | (required if not using `--dir`) |
| `--dir` | Local directory path | (required if not using `--repo`) |
| `--name` | Project name | (derived from repo/directory) |
| `--token` | GitHub token for private repos | (from GITHUB_TOKEN env var) |
| `--output` | Output directory | `./output` |
| `--include` | File patterns to include | Common code files |
| `--exclude` | File patterns to exclude | Test/build directories |
| `--max-size` | Maximum file size in bytes | 100KB |
| `--language` | Language for generated docs | `english` |
| `--no-cache` | Disable LLM response caching | (caching enabled) |
| `--max-abstractions` | Maximum number of abstractions | 10 |

## Advanced configuration examples

Here are some advanced configuration examples for different use cases:

### Analyzing a large codebase

For large codebases, you might want to focus on specific parts and increase file size limits:

```bash
python main.py --dir /path/to/large-project \
  --include "*.py" "*.js" "core/*.ts" "lib/*.ts" \
  --exclude "tests/*" "examples/*" "docs/*" "node_modules/*" \
  --max-size 200000 \
  --max-abstractions 15 \
  --output large-project-docs
```

This configuration:
- Focuses on Python, JavaScript, and TypeScript files in core/ and lib/ directories
- Excludes tests, examples, docs, and node_modules
- Increases the maximum file size to 200KB
- Increases the number of abstractions to 15
- Saves output to a custom directory

### Generating documentation in another language

To generate documentation in a language other than English:

```bash
python main.py --repo https://github.com/username/repo \
  --language "spanish" \
  --output spanish-docs
```

This will generate all documentation content in Spanish. Supported languages include:
- spanish
- french
- german
- chinese
- japanese
- korean
- russian
- portuguese
- italian

### Analyzing a private GitHub repository

For private repositories, you'll need to provide a GitHub token:

```bash
python main.py --repo https://github.com/company/private-repo \
  --token "your-github-token" \
  --include "src/*.js" "src/*.ts" \
  --exclude "src/test/*" \
  --output private-repo-docs
```

Alternatively, you can set the GITHUB_TOKEN environment variable:

```bash
export GITHUB_TOKEN="your-github-token"
python main.py --repo https://github.com/company/private-repo
```

### Optimizing for performance

If you're running multiple analyses or have limited API quota:

```bash
# First run with caching enabled
python main.py --dir /path/to/project --output project-docs

# Subsequent runs will use cached responses
python main.py --dir /path/to/project-v2 --output project-v2-docs

# Force fresh analysis (no caching)
python main.py --dir /path/to/project --output project-docs-fresh --no-cache
```

### Focusing on specific file types

To generate documentation for specific technologies:

```bash
# Frontend web application
python main.py --dir /path/to/frontend \
  --include "*.js" "*.jsx" "*.ts" "*.tsx" "*.css" "*.scss" \
  --exclude "node_modules/*" "dist/*" \
  --name "Frontend App" \
  --output frontend-docs

# Backend API
python main.py --dir /path/to/backend \
  --include "*.py" "*.go" "*.java" \
  --exclude "tests/*" "venv/*" \
  --name "Backend API" \
  --output backend-docs
```

## How PocketFlow works

PocketFlow uses a pipeline of nodes to process your codebase:

1. **FetchRepo**: Crawls the repository or local directory to collect files
2. **IdentifyAbstractions**: Uses AI to identify key abstractions in the code
3. **AnalyzeRelationships**: Determines how abstractions relate to each other
4. **OrderChapters**: Creates a logical sequence for the tutorial
5. **WriteChapters**: Generates detailed content for each chapter
6. **CombineTutorial**: Combines everything into a complete documentation set

Each node performs a specific task and passes its results to the next node in the pipeline.

## Customizing the generation process

While PocketFlow's default settings work well for most projects, you may want to customize how it generates documentation. Here are several ways to modify the generation process:

### Modifying the prompts

The prompts used to generate content are defined in the various node classes in `nodes.py`. You can modify these to change the style, tone, or content of the generated documentation:

1. **IdentifyAbstractions node**: Controls how abstractions are identified and described
   ```python
   # In nodes.py, find the IdentifyAbstractions class
   prompt = f"""
   For the project `{project_name}`:

   Codebase Context:
   {context}

   {language_instruction}Analyze the codebase context.
   Identify the top 5-{max_abstraction_num} core most important abstractions to help those new to the codebase.

   # You can modify this prompt to change how abstractions are identified
   """
   ```

2. **AnalyzeRelationships node**: Controls how relationships between abstractions are determined
   ```python
   # In nodes.py, find the AnalyzeRelationships class
   prompt = f"""
   Based on the following abstractions and relevant code snippets from the project `{project_name}`:

   # You can modify this prompt to change how relationships are analyzed
   """
   ```

3. **WriteChapters node**: Controls how individual chapters are written
   ```python
   # In nodes.py, find the WriteChapters class
   prompt = f"""
   Write a detailed, beginner-friendly tutorial chapter about "{abstraction_name}" for the project {project_name}.

   # You can modify this prompt to change the style and content of chapters
   """
   ```

### Adjusting the documentation structure

The documentation structure is defined in the `CombineTutorial` node in `nodes.py`. You can modify this to change the output format:

```python
# In nodes.py, find the CombineTutorial class
def exec(self, prep_res):
    # ...

    # Create index.md
    index_content = f"""# Tutorial: {project_name}

{relationships['summary']}

**Source Repository:** [{repo_url or "None"}]({repo_url or "None"})

```mermaid
{mermaid_diagram}
```

## Chapters

{chapter_list}

---

Generated by [AI Codebase Knowledge Builder](https://github.com/The-Pocket/Tutorial-Codebase-Knowledge)
"""
    # You can modify this template to change the index.md structure

    # ...
```

### Creating custom templates

You can create custom templates for different types of documentation by modifying the `CombineTutorial` node:

1. **API documentation template**:
   ```python
   # Example modification for API-focused documentation
   index_content = f"""# API documentation: {project_name}

{relationships['summary']}

## API endpoints

{api_endpoints_list}

## Data models

{data_models_list}

## Authentication

{authentication_info}

---

Generated by [AI Codebase Knowledge Builder](https://github.com/The-Pocket/Tutorial-Codebase-Knowledge)
"""
   ```

2. **Developer onboarding template**:
   ```python
   # Example modification for developer onboarding
   index_content = f"""# Developer onboarding: {project_name}

{relationships['summary']}

## Getting started

1. Clone the repository
2. Install dependencies
3. Run the application

## Key components

{chapter_list}

## Development workflow

{workflow_description}

---

Generated by [AI Codebase Knowledge Builder](https://github.com/The-Pocket/Tutorial-Codebase-Knowledge)
"""
   ```

### Modifying the node pipeline

For more advanced customization, you can modify the node pipeline in `flow.py`:

```python
# In flow.py
def create_tutorial_flow():
    """Creates and returns the codebase tutorial generation flow."""

    # Instantiate nodes
    fetch_repo = FetchRepo()
    identify_abstractions = IdentifyAbstractions(max_retries=5, wait=20)
    analyze_relationships = AnalyzeRelationships(max_retries=5, wait=20)
    order_chapters = OrderChapters(max_retries=5, wait=20)
    write_chapters = WriteChapters(max_retries=5, wait=20)
    combine_tutorial = CombineTutorial()

    # Add your custom node
    custom_node = YourCustomNode()

    # Connect nodes in sequence
    fetch_repo >> identify_abstractions
    identify_abstractions >> analyze_relationships
    analyze_relationships >> order_chapters
    order_chapters >> write_chapters
    write_chapters >> custom_node  # Insert your custom node
    custom_node >> combine_tutorial

    # Create the flow starting with FetchRepo
    tutorial_flow = Flow(start=fetch_repo)

    return tutorial_flow
```

### Creating custom nodes

You can create custom nodes to add new functionality to the pipeline:

```python
# Example custom node that adds code metrics to the documentation
class CodeMetricsNode(Node):
    def prep(self, shared):
        files_data = shared["files"]
        return files_data

    def exec(self, prep_res):
        files_data = prep_res
        metrics = {}

        for path, content in files_data:
            # Calculate metrics for each file
            loc = len(content.split('\n'))
            complexity = calculate_complexity(content)  # Your complexity calculation
            metrics[path] = {
                "lines_of_code": loc,
                "complexity": complexity
            }

        return metrics

    def post(self, shared, prep_res, exec_res):
        shared["code_metrics"] = exec_res
```

## Troubleshooting

### API rate limits

If you encounter rate limits with the Gemini API, you can:

1. Use the `--no-cache` flag to disable caching
2. Modify `utils/call_llm.py` to use a different API provider
3. Add delays between API calls

### Large codebases

For large codebases:

1. Use more specific `--include` and `--exclude` patterns
2. Increase `--max-size` if needed
3. Process the codebase in smaller chunks

## Example usage

Here are complete examples of using PocketFlow for different types of projects:

### Example 1: Documenting a web application

```bash
# Generate documentation for a React web application
python main.py \
  --dir /path/to/react-app \
  --include "src/*.js" "src/*.jsx" "src/*.ts" "src/*.tsx" "src/*.css" \
  --exclude "node_modules/*" "build/*" "public/*" "**/*.test.*" \
  --name "Shopping Cart App" \
  --output shopping-cart-docs \
  --max-abstractions 12
```

This will generate documentation that focuses on the source code of a React application, excluding test files and build artifacts. The documentation will identify up to 12 key abstractions in the codebase.

### Example 2: Documenting a Python package

```bash
# Generate documentation for a Python package
python main.py \
  --dir /path/to/python-package \
  --include "*.py" \
  --exclude "tests/*" "examples/*" "docs/*" "*.pyc" "__pycache__/*" \
  --name "Data Processing Library" \
  --output data-lib-docs \
  --max-size 150000
```

This will generate documentation for a Python package, focusing on the Python source files while excluding tests, examples, and documentation. The increased max-size parameter allows for larger Python files to be included in the analysis.

### Example 3: Documenting a microservices architecture

```bash
# Generate documentation for each microservice separately
for service in auth-service user-service payment-service; do
  python main.py \
    --dir /path/to/microservices/$service \
    --include "src/**/*.js" "src/**/*.ts" \
    --exclude "node_modules/*" "dist/*" "**/*.test.*" \
    --name "$service" \
    --output microservices-docs/$service
done

# Create an overview documentation for the entire architecture
python main.py \
  --dir /path/to/microservices \
  --include "docker-compose.yml" "README.md" "architecture.md" \
  --name "Microservices Architecture" \
  --output microservices-docs/overview
```

This example shows how to document a microservices architecture by generating documentation for each service separately and then creating an overview documentation for the entire architecture.

### Example 4: Documenting a mobile app

```bash
# Generate documentation for an iOS app
python main.py \
  --dir /path/to/ios-app \
  --include "*.swift" "*.h" "*.m" \
  --exclude "Pods/*" "build/*" "*.test.swift" \
  --name "iOS Fitness App" \
  --output ios-app-docs

# Generate documentation for an Android app
python main.py \
  --dir /path/to/android-app \
  --include "app/src/main/**/*.java" "app/src/main/**/*.kt" "app/src/main/**/*.xml" \
  --exclude "app/src/test/*" "app/build/*" \
  --name "Android Fitness App" \
  --output android-app-docs
```

This example shows how to document both iOS and Android versions of a mobile app, focusing on the relevant source files for each platform.

### Example 5: Documenting a multi-language project

```bash
# Generate documentation for a project with multiple languages
python main.py \
  --dir /path/to/full-stack-project \
  --include "frontend/src/**/*.ts" "frontend/src/**/*.tsx" "backend/**/*.py" "database/**/*.sql" \
  --exclude "**/node_modules/*" "**/__pycache__/*" "**/*.test.*" \
  --name "E-commerce Platform" \
  --output ecommerce-docs \
  --max-abstractions 15
```

This example shows how to document a full-stack project that includes TypeScript/React frontend, Python backend, and SQL database scripts. The increased max-abstractions parameter allows for more components to be identified across the different parts of the system.

### Example 6: Generating documentation in multiple languages

```bash
# Generate English documentation
python main.py \
  --dir /path/to/project \
  --name "International App" \
  --output docs/english

# Generate Spanish documentation
python main.py \
  --dir /path/to/project \
  --name "International App" \
  --language "spanish" \
  --output docs/spanish

# Generate Chinese documentation
python main.py \
  --dir /path/to/project \
  --name "International App" \
  --language "chinese" \
  --output docs/chinese
```

This example shows how to generate documentation for the same project in multiple languages to support international teams or users.

## Conclusion

PocketFlow is a powerful tool that automates the documentation generation process for your codebase:

1. **Key benefits:**
   - Automatically analyzes your codebase to identify key abstractions
   - Determines relationships between components
   - Generates comprehensive, beginner-friendly documentation
   - Creates visual diagrams to illustrate system architecture
   - Supports multiple programming languages and documentation languages
   - Saves significant time compared to manual documentation

2. **Best practices:**
   - Use specific include/exclude patterns to focus on relevant code
   - Adjust max-abstractions based on project complexity
   - Consider generating separate documentation for different parts of large systems
   - Use the caching feature to optimize performance for multiple runs
   - Customize the generation process for specific documentation needs

3. **Next steps:**
   - Experiment with different configuration options to find what works best for your project
   - Explore customizing the prompts and templates for your specific documentation style
   - Consider integrating PocketFlow into your CI/CD pipeline for automatic documentation updates

By leveraging PocketFlow's capabilities, you can maintain up-to-date, high-quality documentation with minimal effort, allowing your team to focus on development while ensuring that knowledge about your codebase remains accessible to all team members.

---

For more information, see the [PocketFlow GitHub repository](https://github.com/The-Pocket/PocketFlow) or contact your documentation team.
