# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Mantis Fork

This is the **metis-mantis fork** of [Arize-ai/openinference](https://github.com/Arize-ai/openinference). It is included as a git submodule in the `mantis` repo at `mantisdk/openinference`.

### Remotes

- `origin` = `git@github.com:metis-mantis/openinference.git` (our fork -- push here)
- `upstream` = `https://github.com/Arize-ai/openinference.git` (Arize upstream -- pull updates)

### Mantis-Specific Changes

All Mantis modifications live on the `mantis/insight-trace-io` branch. Current changes:

**`openinference-instrumentation-openai-agents` (`_processor.py`)**:
- The `OpenInferenceTracingProcessor` now propagates child span input/output to the root "Agent workflow" span using `insight.trace.input` / `insight.trace.output` attributes.
- This is needed because the OpenAI Agents SDK's `Trace` object doesn't carry input/output data, leaving root spans empty in Mantis Insight.
- Handles both `ResponseSpanData` (OpenAI Responses API) and `GenerationSpanData` (LiteLLM / chat completions).
- First child input = user prompt. Last child output = agent's final response (last writer wins).

### Pulling Upstream Updates

```bash
cd mantisdk/openinference
git fetch upstream
git merge upstream/main
# Resolve any conflicts in _processor.py, then push to origin
git push origin mantis/insight-trace-io
```

Then update the submodule pointer in the parent `mantis` repo:
```bash
cd ../..
git add mantisdk/openinference
git commit -m "chore: update openinference submodule"
```

## Repository Overview

OpenInference is a multi-language monorepo providing OpenTelemetry-based instrumentation for AI/ML applications:

- **Python**: Instrumentation for OpenAI, LangChain, LlamaIndex, DSPy, etc.
- **JavaScript/TypeScript**: Node.js instrumentations with pnpm workspaces
- **Java**: Instrumentation for LangChain4j and Spring AI
- **Specification**: OpenInference semantic conventions in `spec/`

## Repository Structure

```
.
├── python/                    # Python packages and instrumentations
│   ├── instrumentation/       # Individual instrumentor packages
│   ├── openinference-instrumentation/
│   └── openinference-semantic-conventions/
├── js/                        # JavaScript/TypeScript workspace
│   ├── packages/              # pnpm workspace packages
│   └── examples/
├── java/                      # Java packages
│   ├── instrumentation/
│   ├── openinference-instrumentation/
│   └── openinference-semantic-conventions/
├── spec/                      # OpenInference specification
└── scripts/                   # Repository automation scripts
```

## Essential Commands by Language

### Python

**Package Management:**
```bash
# Install development dependencies
pip install -r python/dev-requirements.txt

# Install tox-uv for testing automation
pip install tox-uv==1.11.2

# Compose namespace package
tox run -e add_symlinks

# Install in editable mode
pip install -e python/openinference-instrumentation
```

**Testing and Quality:**
```bash
# Run all CI checks in parallel
tox run-parallel

# Run tests for specific package (e.g., openai)
tox run -e test-openai

# Run linting and formatting
tox run -e ruff-openai

# Run type checking
tox run -e mypy-openai

# Run all checks for a package
tox run -e ruff-mypy-test-openai
```

**Key Tools:**
- tox for automation
- ruff for formatting and linting
- mypy for type checking

### JavaScript/TypeScript

**See `js/CLAUDE.md` for detailed JavaScript-specific guidance.**

**Package Management:**
```bash
# MUST use pnpm, not npm
pnpm install --frozen-lockfile -r

# Build all packages (includes prebuild)
pnpm run -r build

# Run prebuild only
pnpm run -r prebuild
```

**Testing and Quality:**
```bash
# Run tests
pnpm run -r test

# Type checking
pnpm run type:check

# Linting
pnpm run lint

# Formatting
pnpm run prettier:check
pnpm run prettier:write
```

**Version Management:**
```bash
# Create changeset for version bumping
pnpm changeset
```

### Java

**Build and Test:**
```bash
# Build all Java packages
cd java && ./gradlew build

# Run tests
./gradlew test

# Publish to Maven Local
./gradlew publishToMavenLocal
```

## Development Workflow

1. Explore the codebase to understand existing patterns
2. Check language-specific guides (`js/CLAUDE.md`, `python/DEVELOPMENT.md`)
3. Run tests using the appropriate command for your language
4. Ensure code passes all quality checks before committing

### Creating New Instrumentors

See language-specific documentation:
- **Python**: `python/DEVELOPMENT.md`
- **JavaScript**: `js/CLAUDE.md`
- **Java**: Existing patterns in `java/instrumentation/`

## Release Management

This repository uses release-please for automated version management.

**Configuration:** `.release-please-manifest.json` and `release-please-config.json`

**Process:**
1. Release-please creates PRs with version bumps and changelogs
2. Merge the PR to trigger releases to PyPI, npm, and Maven Central

## Contributing

See `CONTRIBUTING` for detailed guidelines. Key points:
- Keep PRs small and focused
- Follow conventional commit format for PR titles
- Include tests for new features and bug fixes
- Run all quality checks before submitting

## Requirements

### All Instrumentors Must:
1. Support suppressing tracing
2. Propagate context attributes (session ID, user ID, metadata, tags)
3. Respect trace configuration for masking sensitive data
4. Follow OpenInference semantic conventions
5. Include comprehensive tests

### Security
- Never commit sensitive data (.env files, credentials)
- Use trace configuration for masking PII

## Additional Documentation

- **JavaScript/TypeScript**: `js/CLAUDE.md`
- **Python**: `python/DEVELOPMENT.md`
- **Java**: `java/README.md`
