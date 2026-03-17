---
name: itkdev-documentation
description: Technical documentation and README generation for ITK Dev projects. Use when asked to create, update, or improve documentation, README files, deployment guides, architecture docs, or API documentation. Follows ITK Dev documentation standards with clear structure and procedural content.
---

# Documentation Generation Guide

You are assisting with technical documentation for ITK Dev projects. This skill covers README files, deployment guides, and architecture documentation.

## When to Activate

Use this skill when the user asks to:
- Create or update a README file
- Write technical documentation
- Generate deployment or installation guides
- Document architecture or API endpoints

## Project Detection

Before generating documentation, detect the project type by checking for:

| Project Type | Indicators |
|--------------|------------|
| Drupal | `*.info.yml`, `web/modules/`, `drupal/core` in composer.json |
| Symfony | `symfony/framework-bundle` in composer.json, `config/bundles.php` |
| Node.js | `package.json`, `node_modules/` |
| Python | `requirements.txt`, `pyproject.toml`, `setup.py` |
| Docker | `Dockerfile`, `docker-compose.yml` |
| ITK Dev Docker | `itkdev-docker-compose` references, `.docker/` directory |

Tailor documentation structure and content to the detected project type.

## Documentation Style Guide

Follow these conventions based on the [AarhusAI documentation style](https://github.com/AarhusAI/documentation/tree/main/technical):

### Structure

1. **Hierarchical headings** with clear navigation
   - H1 (`#`): Main document title
   - H2 (`##`): Major sections
   - H3 (`###`): Subsections

2. **Logical section flow**
   - Overview/Introduction first
   - Prerequisites and requirements
   - Step-by-step procedures
   - Configuration reference
   - Troubleshooting (if applicable)

### Content Conventions

1. **Variables and placeholders**: Use angle brackets for values that must be replaced
   ```
   <DOMAIN_NAME>
   <API_KEY>
   <DATABASE_PASSWORD>
   ```

2. **Code blocks**: Use fenced code blocks with language identifiers
   ```bash
   # Shell commands
   docker compose up -d
   ```

3. **Callouts**: Use bold for important notes
   ```markdown
   **NOTE:** Critical information here.

   **WARNING:** Potentially destructive operation.
   ```

4. **Cross-references**: Link to related documentation and external resources

5. **Procedural format**: Number steps for sequential operations, use bullets for non-sequential items

## README Template

When generating a README, include these sections as appropriate:

`````markdown
# Project Name

Brief description of what the project does and its purpose.

## Requirements

- Requirement 1
- Requirement 2

## Installation

Step-by-step installation instructions.

### Development Setup

```bash
# Clone the repository
git clone <REPOSITORY_URL>
cd <PROJECT_NAME>

# Install dependencies
<INSTALL_COMMAND>

# Start development server
<START_COMMAND>
```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `VAR_NAME` | What it does | `default_value` |

## Usage

How to use the project with examples.

## Development

### Running Tests

```bash
<TEST_COMMAND>
```

### Code Standards

```bash
<LINT_COMMAND>
```

## Deployment

Brief deployment instructions or link to deployment docs.

## Contributing

Link to contribution guidelines or brief instructions.

## License

License information.
`````

## Technical Documentation Templates

### Deployment Guide Template

`````markdown
# Deployment Guide

## Overview

Brief description of the deployment architecture and process.

## Prerequisites

- Prerequisite 1
- Prerequisite 2

## Variables

| Variable | Description |
|----------|-------------|
| `<VARIABLE>` | What this variable represents |

## Deployment Steps

### 1. Environment Setup

Description of what this step accomplishes.

```bash
# Commands for this step
```

### 2. Configuration

Description of configuration needed.

### 3. Deployment

Actual deployment commands.

### 4. Verification

How to verify the deployment succeeded.

## Troubleshooting

### Common Issue 1

**Symptom:** Description of the problem.

**Solution:** How to fix it.
`````

### Architecture Documentation Template

```markdown
# Architecture Overview

## System Components

Description of major system components and their responsibilities.

## Data Flow

How data moves through the system.

## External Dependencies

Services and APIs the system depends on.

## Diagram

Include diagrams where helpful (Mermaid, ASCII, or image references).
```

### API Documentation Template

`````markdown
# API Reference

## Authentication

How to authenticate with the API.

## Endpoints

### `GET /endpoint`

**Description:** What this endpoint does.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `param` | string | Yes | Description |

**Response:**

```json
{
  "key": "value"
}
```

**Example:**

```bash
curl -X GET https://api.example.com/endpoint
```
`````

## ITK Dev Docker Projects

For projects using ITK Dev Docker infrastructure, include:

### Standard README Sections

`````markdown
## Development Setup

### Prerequisites

- Docker and Docker Compose
- [itkdev-docker-compose](https://github.com/itk-dev/itkdev-docker-compose) CLI

### Getting Started

1. Clone the repository:
   ```bash
   git clone <REPOSITORY_URL>
   cd <PROJECT_NAME>
   ```

2. Start the Docker environment:
   ```bash
   docker compose up -d
   ```

3. Install dependencies:
   ```bash
   itkdev-docker-compose composer install
   ```

4. Access the site:
   ```bash
   itkdev-docker-compose open
   ```

### Common Tasks

| Task | Command |
|------|---------|
| Clear cache | `itkdev-docker-compose drush cr` |
| Export config | `itkdev-docker-compose drush cex -y` |
| Import config | `itkdev-docker-compose drush cim -y` |
| Run tests | `task ci` |
`````

### Taskfile Integration

If a `Taskfile.yml` exists, document available tasks:

```markdown
## Available Tasks

Run `task` to see all available tasks. Common tasks:

| Task | Description |
|------|-------------|
| `task dev:setup` | Initial project setup |
| `task ci` | Run all CI checks |
| `task config:export` | Export Drupal configuration |
```

## Quality Checklist

Before finalizing documentation, verify:

- [ ] All placeholders use angle bracket notation (`<VALUE>`)
- [ ] Code blocks have language identifiers
- [ ] Steps are numbered for sequential operations
- [ ] Prerequisites are listed before instructions
- [ ] Commands are copy-paste ready
- [ ] Links to external resources are included where helpful
- [ ] No sensitive information (passwords, keys) in examples
- [ ] Documentation matches the actual project state
