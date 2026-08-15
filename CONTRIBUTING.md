# Contributing to Disk Cleanup v2

Thank you for your interest in contributing to Disk Cleanup v2! 🚀

We welcome bug reports, feature requests, documentation improvements, and code contributions.

## How to Contribute

### 1. Fork the Repository

Create your own fork of the repository and clone it locally.

```bash
git clone https://github.com/YOUR_USERNAME/disk-cleanup-v2.git
```

### 2. Create a Branch

```bash
git checkout -b feature/my-feature
```

Examples:

```bash
git checkout -b feature/windows-cache-support
git checkout -b feature/docker-analysis
git checkout -b fix/path-validation
```

### 3. Make Your Changes

Implement your changes and ensure:

* Existing functionality continues to work
* Safety checks remain intact
* Documentation is updated when necessary

### 4. Test Thoroughly

Before submitting a pull request:

* Test on your platform
* Verify dry-run mode works
* Verify protected paths cannot be deleted
* Verify cleanup reports remain accurate

### 5. Submit a Pull Request

Provide:

* Clear title
* Description of changes
* Screenshots (if applicable)
* Testing notes

---

## Contribution Ideas

### Cleanup Targets

Add support for:

* Additional package managers
* IDE caches
* Browser caches
* Build systems
* Language-specific caches

### Platform Improvements

* Better Windows support
* Linux distribution optimizations
* macOS-specific cleanup targets

### Reporting

* HTML reports
* Interactive summaries
* Storage visualizations
* Historical cleanup tracking

### Safety Improvements

* Additional path validation
* Better quarantine workflows
* Recovery enhancements

---

## Safety Requirements

Disk Cleanup v2 is a safety-first project.

Contributions must NEVER:

* Delete user documents
* Delete source code
* Delete Git repositories
* Delete SSH keys
* Delete cloud storage folders
* Bypass validation logic

Any feature that removes files must:

1. Support dry-run mode
2. Require explicit confirmation
3. Respect protected paths
4. Provide recovery when possible

---

## Code Style

### Python

* Follow PEP 8
* Use descriptive variable names
* Add comments for complex logic
* Keep functions focused and readable

### Documentation

* Update README.md if behavior changes
* Document new cleanup targets
* Include examples where useful

---

## Reporting Bugs

When opening an issue include:

* Operating System
* Python version
* Claude Code version
* Steps to reproduce
* Expected behavior
* Actual behavior

---

## Feature Requests

Feature requests are encouraged.

Please explain:

* The problem being solved
* Proposed solution
* Potential risks
* Platform compatibility

---

## License

By contributing to this project, you agree that your contributions will be licensed under the MIT License.

Thank you for helping make Disk Cleanup v2 safer and more useful for everyone.

