---
name: ECS Commit
description: Create well-formatted commits for Infrastructure as Code projects with conventional commit messages and emoji, tailored for Azure and AWS environments and use as a slash command in RooCode.
category: Development
author: Adam Niewiarowski
version: 1.0.0
tags:
  - git
  - commit
  - iac
  - azure
  - aws
  - conventional-commits
dependencies:
  - git
---

# RooCode Command: ECS Commit

This command helps you create well-formatted commits for Infrastructure as Code (IaC) projects with conventional commit messages and emoji, tailored for Azure and AWS environments.

## Usage

To create a commit, just type:
```
/ecs-commit
```

Or with options:
```
/ecs-commit --no-verify
```

## What This Command Does

1. Extracts JIRA ticket number from the current branch name (format: LSMECSSD-xxxx)
   - If no JIRA ticket is found in the branch name, prompts the user for the ticket number
   - If user responds with 'none', 'n/a', or similar, proceeds without the prefix
2. Unless specified with `--no-verify`, automatically runs pre-commit checks:
   - Validates YAML and JSON syntax
   - Checks ARM templates for Azure
   - Validates CloudFormation templates for AWS
   - Performs CDK synthesis checks
   - Runs PowerShell syntax validation
   - Executes security scanning for IaC configurations
3. Checks which files are staged with `git status`
4. If 0 files are staged, automatically adds all modified and new files with `git add`
5. Performs a `git diff` to understand what changes are being committed
6. Analyzes the diff to determine if multiple distinct logical changes are present
7. If multiple distinct changes are detected, suggests breaking the commit into multiple smaller commits
8. For each commit (or the single commit if not split), creates a commit message using emoji conventional commit format with JIRA prefix

## JIRA Ticket Integration

This command automatically integrates with your JIRA workflow by:

1. **Branch Name Detection**: Extracts the JIRA ticket number from the current branch name using the format `LSMECSSD-xxxx`
   - Example branch: `LSMECSSD-12345 my branch name`
   - Extracted ticket: `LSMECSSD-12345`

2. **User Prompting**: If no JIRA ticket is found in the branch name, the command will prompt you to enter the ticket number

3. **Skip Option**: If there's no associated JIRA ticket, you can respond with:
   - `none`
   - `n/a`
   - `no ticket`
   - Or any similar response indicating no ticket is needed

4. **Commit Message Format**: The final commit message will be formatted as:
   - With ticket: `LSMECSSD-12345 🛡️ fix: Fixed security configuration bug`
   - Without ticket: `🛡️ fix: Fixed security configuration bug`

## Best Practices for IaC Commits

- **Verify before committing**: Ensure IaC templates are valid, security-compliant, and follow best practices
- **Atomic commits**: Each commit should contain related changes that serve a single purpose
- **Split large changes**: If changes touch multiple concerns, split them into separate commits
- **Conventional commit format**: Use the format `<type>: <description>` where type is one of:
  - `feat`: A new feature or resource
  - `fix`: A bug fix or configuration correction
  - `docs`: Documentation changes
  - `style`: Code style changes (formatting, etc.)
  - `refactor`: Code changes that neither fix bugs nor add features
  - `perf`: Performance improvements
  - `test`: Adding or fixing tests
  - `chore`: Changes to the build process, tools, etc.
- **Present tense, imperative mood**: Write commit messages as commands (e.g., "add resource" not "added resource")
- **Concise first line**: Keep the first line under 72 characters
- **Emoji**: Each commit type is paired with an appropriate emoji:
  - ✨ `feat`: New feature or resource
  - 🐛 `fix`: Bug fix or configuration correction
  - 📝 `docs`: Documentation
  - 💄 `style`: Formatting/style
  - ♻️ `refactor`: Code refactoring
  - ⚡️ `perf`: Performance improvements
  - ✅ `test`: Tests
  - 🔧 `chore`: Tooling, configuration
  - 🚀 `ci`: CI/CD improvements
  - 🗑️ `revert`: Reverting changes
  - 🔒️ `fix`: Fix security issues
  - 🏗️ `refactor`: Make architectural changes
  - 🌐 `feat`: Internationalization and localization
  - 🚸 `feat`: Improve user experience / usability
  - 🩹 `fix`: Simple fix for a non-critical issue
  - 🔥 `fix`: Remove code or files
  - 🎨 `style`: Improve structure/format of the code
  - 🚑️ `fix`: Critical hotfix
  - 🎉 `chore`: Begin a project
  - 🔖 `chore`: Release/Version tags
  - 🚧 `wip`: Work in progress
  - 💚 `fix`: Fix CI build
  - 📌 `chore`: Pin dependencies to specific versions
  - 👷 `ci`: Add or update CI build system
  - ✏️ `fix`: Fix typos
  - ⏪️ `revert`: Revert changes
  - 📄 `chore`: Add or update license
  - 💥 `feat`: Introduce breaking changes
  - 🍱 `assets`: Add or update assets
  - ♿️ `feat`: Improve accessibility
  - 💡 `docs`: Add or update comments in source code
  - 🔊 `feat`: Add or update logs
  - 🔇 `fix`: Remove logs
  - 🙈 `chore`: Add or update .gitignore file
  - 📸 `test`: Add or update snapshots
  - ⚗️ `experiment`: Perform experiments
  - 🚩 `feat`: Add, update, or remove feature flags
  - ⚰️ `refactor`: Remove dead code
  - 🦺 `feat`: Add or update code related to validation
  - ☁️ `feat`: Add cloud resources or services
  - 🔧 `chore`: Update infrastructure tools
  - 🏷️ `feat`: Add or update resource tags
  - 📦️ `chore`: Update deployment packages
  - 🌐 `feat`: Add multi-region support
  - 🔄 `refactor`: Update resource dependencies
  - 🛡️ `fix`: Fix security configurations
  - 📊 `feat`: Add monitoring and logging
  - 🔑 `feat`: Update authentication and authorization
  - 🌍 `feat`: Add cross-cloud compatibility

## IaC-Specific Guidelines for Splitting Commits

When analyzing the diff, consider splitting commits based on these criteria:

1. **Different concerns**: Changes to unrelated parts of the infrastructure
2. **Different types of changes**: Mixing features, fixes, refactoring, etc.
3. **File patterns**: Changes to different types of IaC files (ARM vs CloudFormation vs CDK)
4. **Logical grouping**: Changes that would be easier to understand or review separately
5. **Size**: Very large changes that would be clearer if broken down
6. **Environment separation**: Changes affecting different environments (dev, staging, prod)
7. **Service boundaries**: Changes to different cloud services or resources

## Examples

Good IaC commit messages:
- LSMECSSD-12345 ☁️ feat: add Azure storage account with encryption
- LSMECSSD-67890 🐛 fix: correct CloudFormation template parameter references
- LSMECSSD-11111 📝 docs: update README with deployment instructions
- LSMECSSD-22222 ♻️ refactor: simplify CDK construct architecture
- LSMECSSD-33333 🛡️ fix: resolve security group overly permissive rules
- LSMECSSD-44444 🔧 chore: update Terraform provider versions
- LSMECSSD-55555 🏷️ feat: add resource tagging strategy implementation
- LSMECSSD-66666 📊 feat: implement CloudWatch monitoring for EC2 instances
- LSMECSSD-77777 🔑 feat: add Azure RBAC role assignments
- LSMECSSD-88888 🌍 feat: add multi-region deployment configuration
- LSMECSSD-99999 🦺 feat: add input validation for ARM template parameters
- LSMECSSD-10101 💚 fix: resolve failing pipeline validation
- LSMECSSD-20202 🔒️ fix: strengthen network security group rules
- LSMECSSD-30303 ♿️ feat: improve resource accessibility compliance

Example of splitting IaC commits:
- LSMECSSD-11111 First commit: ☁️ feat: add Azure virtual network and subnets
- LSMECSSD-11111 Second commit: 📝 docs: update infrastructure documentation
- LSMECSSD-11111 Third commit: 🔧 chore: update ARM template parameters
- LSMECSSD-11111 Fourth commit: 🏷️ feat: implement resource tagging strategy
- LSMECSSD-11111 Fifth commit: 🛡️ fix: resolve security vulnerabilities in network config
- LSMECSSD-11111 Sixth commit: ✅ test: add validation tests for infrastructure code
- LSMECSSD-11111 Seventh commit: 🔒️ fix: update security policies and compliance rules
- LSMECSSD-11111 Eighth commit: 📊 feat: add monitoring and alerting configuration

## Command Options

- `--no-verify`: Skip running the pre-commit checks (validation, security scanning, etc.)

## Important Notes

- By default, pre-commit checks (YAML/JSON validation, ARM/CloudFormation validation, CDK synthesis, PowerShell syntax, security scanning) will run to ensure IaC quality
- If these checks fail, you'll be asked if you want to proceed with the commit anyway or fix the issues first
- If specific files are already staged, the command will only commit those files
- If no files are staged, it will automatically stage all modified and new files
- The commit message will be constructed based on the changes detected
- Before committing, the command will review the diff to identify if multiple commits would be more appropriate
- If suggesting multiple commits, it will help you stage and commit the changes separately
- Always reviews the commit diff to ensure the message matches the changes
- This command is optimized for IaC projects and excludes package manager operations