---
title: "Prerequisites & Setup"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## Development Environment Setup

### Required Software

Before starting, ensure you have the following installed on your device (for testing and debugging):

- **[Node.js 18 or higher](https://nodejs.org/)** - Download from nodejs.org
- **pnpm package manager** - Install with `npm install -g pnpm`
- **[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)** - Installation guide
- **[Git](https://git-scm.com/downloads)** - For cloning the repository

### Verify Installation

```bash
# Check Node.js version
node --version  # Should be 18+

# Check pnpm
pnpm --version

# Check AWS CLI
aws --version

# Check Git
git --version
```

## Content

- [Setup Edge Device](2.1-setupedge/)
- [Setup Cloud Environment](2.2-setupcloud/)
