# README template

Use these sections as appropriate — omit any that do not apply rather than
leaving empty placeholders.

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

## ITK Dev Docker projects

For projects using ITK Dev Docker infrastructure, prefer this Development Setup
section. Command tables for the `itkdev-docker-compose` CLI and Taskfile live in
the `itkdev-docker` and `itkdev-taskfile` skills (in the
`itkdev-scaffolding-and-templates` plugin) — link to them rather than restating
them.

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
`````
