# Weather Platform Tutorial

A step-by-step guide for building an IoT Weather Platform using AWS services.

## Prerequisites

You need:

- **Hugo Extended** (v0.25+) - [Download here](https://gohugo.io/installation/)
- **Git** - For version control

### Install Hugo

**Windows:**

```bash
choco install hugo-extended
```

**macOS:**

```bash
brew install hugo
```

**Linux:**

```bash
sudo apt install hugo
```

**Verify:**

```bash
hugo version
```

## Quick Start

### 1. Clone and Setup

```bash
git clone https://github.com/PancakesLmao/Weather-Platform-tutorial.git
cd Weather-Platform-tutorial
git submodule update --init --recursive
```

### 2. Run Server

```bash
hugo server
```

### 3. View Tutorial

Open: http://localhost:1313

## Common Commands

```bash
# Development server
hugo server

# Build for production
hugo --minify

# Different port
hugo server --port 8080

# Clean build
hugo --cleanDestinationDir
```

## Project Structure

```
Weather-Platform-tutorial/
├── content/             # Tutorial content
│   ├── 1-Introduce/    # Introduction
│   ├── 2-Prerequiste/  # Prerequisites & Setup
│   │   ├── 2.1-setupEdge/    # Edge device setup
│   │   └── 2.2-setupCloud/   # Cloud environment
│   ├── 3-CreateDataLake/     # S3 Data Lake
│   ├── 4-SetupIoTCore/       # AWS IoT Core
│   │   ├── 4.1-ThingGroups/
│   │   ├── 4.2-SecurityPolicies/
│   │   └── 4.3-RuleEngine/
│   ├── 5-AmplifyConfiguration/ # Amplify Setup
│   │   ├── 5.1-ProjectSetup/
│   │   ├── 5.2-BackendConfiguration/
│   │   ├── 5.3-CustomCDKResources/
│   │   ├── 5.4-LambdaFunctions/
│   │   ├── 5.5-DataProcessing/
│   │   ├── 5.6-CloudFrontSetup/
│   │   ├── 5.7-Authentication/
│   │   └── 5.8-ProductionDeployment/
│   ├── 6-ResourceCleanup/    # Cleanup
│   └── 7-PricingPlan/        # Cost analysis
├── static/              # Images and assets
├── themes/              # Hugo themes
├── config.toml          # Configuration
└── README.md           # Project overview
```

## Configuration

Main settings in `config.toml`:

```toml
baseURL = "https://your-site.com"
title = "Weather Platform Tutorial"
theme = "hugo-theme-learn"

[params]
  themeVariant = "blue"
  author = "ITea Lab"
```

## Content Format

Each page needs front matter:

```yaml
---
title: "Page Title"
weight: 10
chapter: false
pre: "<b>1. </b>"
---
```

### Useful Shortcodes

```markdown
{{% notice info %}}
Information box
{{% /notice %}}

{{% notice warning %}}
Warning message
{{% /notice %}}
```

## Build & Deploy

```bash
# Build site
hugo --minify

# Output in public/ folder
```

The site automatically deploys via GitHub Pages when you push to main branch.

## Troubleshooting

```bash
# Clear cache
hugo mod clean

# Check version
hugo version
```
