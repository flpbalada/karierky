# Karierky - Prague Tech Companies

## Overview

A curated directory of technology companies in Prague with links to their career pages.

## Main Components

### Company Directory (`companies.md`)

Simple markdown file with one company per line using markdown links:

```markdown
[Company Name](https://company.com/careers) - Brief description of what they do
[Another Company](https://another.com/jobs) - E-commerce platform, React/Node.js
```

## Project Structure

**File descriptions:**

- `companies.md` - Curated list of Prague tech companies with career URLs and descriptions
- `.claude/commands/add-company.md` - Claude Code command for adding new companies

## Usage

### Adding Companies

Use Claude Code's `/add-company` command to add new companies to the directory:

```bash
/add-company https://company.com/careers
```

This command is executed by Claude Code and handles the entire process automatically.
