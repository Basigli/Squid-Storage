# Contributing

Thank you for your interest in contributing to Squid Storage! Please follow the conventions below to keep the project consistent and maintainable.

## Commit Conventions

Follow the **Conventional Commits** format:

```
(type)scope: message
```

**Example:**

```bash
git commit -m "(feat)server: add round-robin DataNode selection"
git commit -m "(fix)protocol: handle empty file path argument"
git commit -m "(docs)readme: add architecture diagram"
```

### Commit Types

| Type | When to use |
|------|-------------|
| `feat` | New feature |
| `chore` | Maintenance, dependency updates |
| `refactor` | Code restructuring without behaviour change |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `test` | Adding or updating tests |
| `misc` | Anything that does not fit another type |

Keep each commit focused on **one logical change**. Do not mix feature work and refactoring in the same commit.

## Branching Strategy

Create a **feature branch** for every new development phase:

```bash
git checkout -b feature/my-feature
```

When a milestone is reached, create a **release branch**:

```bash
git checkout -b release/v1.2.0
```

## Versioning

Squid Storage follows [Semantic Versioning](https://semver.org/):

```
MAJOR.MINOR.PATCH
```

| Segment | Increment when… |
|---------|----------------|
| `MAJOR` | Breaking changes to the API or protocol |
| `MINOR` | New backwards-compatible features |
| `PATCH` | Backwards-compatible bug fixes |

### Creating a Tag and Release

```bash
# Create an annotated tag
git tag -a v1.2.0 -m "Add replication rebalancing improvements"
git push origin v1.2.0

# Delete a tag (if needed)
git tag -d v1.2.0
git push origin --delete v1.2.0
```

Then go to **GitHub → Releases → New Release** and attach the tag.

## Changelog

Consider writing a `CHANGELOG.md` for every new release. You can auto-generate one from commit messages:

```bash
git log --pretty=format:"- %s" v1.1.0..v1.2.0 >> CHANGELOG.md
```

Or write it manually using this structure:

```markdown
# Changelog

## [1.2.0] - 2025-06-01
### Added
- Replication rebalancing on DataNode reconnect.

### Fixed
- Race condition in file lock expiration check.

## [1.1.0] - 2025-04-15
### Added
- Secondary socket for asynchronous client updates.
```

## Building and Testing

Before submitting a pull request, make sure the project builds cleanly on your platform:

```bash
cmake -B build
cmake --build build
```

Run the integration tests using the scripts in `test/`:

```bash
# Requires tmux
test/testMultiple.sh
```

## Documentation

Documentation is built with [MkDocs](https://www.mkdocs.org/) using the **readthedocs** theme. To preview changes locally:

```bash
pip install mkdocs
mkdocs serve
```

Then open [http://127.0.0.1:8000](http://127.0.0.1:8000) in your browser.

To build the static site:

```bash
mkdocs build
```

The generated site is placed in the `site/` directory.
