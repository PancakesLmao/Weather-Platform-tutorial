# Weather Platform Tutorial

A comprehensive tutorial for building a centralized IoT Weather Platform using AWS services, Next.js, and modern serverless architecture.

## Prerequisites

Before running this tutorial locally, ensure you have:

- **Hugo Extended** (v0.25 or higher) - [Download here](https://gohugo.io/installation/)
- **Git** - For cloning and version control
- **Text Editor** - VS Code, Sublime Text, or your preferred editor

### Install Hugo

**Windows:**

```bash
# Using Chocolatey
choco install hugo-extended

# Using Scoop
scoop install hugo-extended

# Or download from GitHub releases
# https://github.com/gohugoio/hugo/releases
```

**macOS:**

```bash
# Using Homebrew
brew install hugo

# Using MacPorts
sudo port install hugo
```

**Linux:**

```bash
# Ubuntu/Debian
sudo apt install hugo

# Arch Linux
sudo pacman -S hugo

# Or download binary from GitHub releases
```

**Verify Installation:**

```bash
hugo version
# Should show: hugo v0.25+ extended
```

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/PancakesLmao/Weather-Platform-tutorial.git
cd Weather-Platform-tutorial
```

### 2. Install Theme Dependencies

The tutorial uses the Hugo Learn theme, which is included as a git submodule:

```bash
# Initialize and update submodules
git submodule update --init --recursive

# Or if cloning for the first time
git clone --recursive <repository-url>
```

### 3. Run Development Server

```bash
# Start Hugo development server
hugo server

# Or with specific options
hugo server --bind 0.0.0.0 --port 1313 --buildDrafts --buildFuture
```

### 4. Access the Tutorial

Open your browser and navigate to:

- **Local URL**: http://localhost:1313
- **Network URL**: http://[your-ip]:1313 (if using --bind 0.0.0.0)

## Development Commands

### Basic Commands

```bash
# Build the site (generates public/ folder)
hugo

# Start development server
hugo server

# Start server with drafts and future content
hugo server -D -F

# Start server and open browser automatically
hugo server --navigateToChanged

# Build for production
hugo --minify
```

### Advanced Options

```bash
# Serve on different port
hugo server --port 8080

# Serve on all network interfaces
hugo server --bind 0.0.0.0

# Enable live reload
hugo server --liveReloadPort 443

# Verbose output for debugging
hugo server --verbose

# Clean build (remove cache)
hugo --cleanDestinationDir
```

### Content Management

```bash
# Create new content page
hugo new content/section/page.md

# Create new chapter
hugo new --kind chapter content/section/_index.md

# List all content
hugo list all

# Check for broken links
hugo --printPathWarnings
```

## Project Structure

```
Weather-Platform-tutorial/
├── archetypes/          # Content templates
├── content/             # Tutorial content (Markdown files)
│   ├── 1-Introduce/     # Introduction section
│   ├── 2-Prerequiste/   # Prerequisites and setup
│   ├── 3-CreateDataLake/# Data lake configuration
│   ├── 4-ConfigureAWSIoTCore/ # IoT Core setup
│   ├── 5-SetupAmplify/  # Amplify deployment
│   └── 6-cleanup/       # Resource cleanup
├── layouts/             # Custom HTML templates
├── static/              # Static assets (images, CSS, JS)
├── themes/              # Hugo themes
│   └── hugo-theme-learn/# Learn theme
├── config.toml          # Hugo configuration
├── netlify.toml         # Netlify deployment config
└── README.md           # This file
```

## Configuration

### Hugo Configuration (`config.toml`)

Key configuration options:

```toml
baseURL = "https://your-tutorial-site.com"
languageCode = "en-us"
title = "Weather Platform Tutorial"
theme = "hugo-theme-learn"

[params]
  # Edit page button
  editURL = "https://github.com/your-repo/edit/main/content/"

  # Author information
  author = "ITea Lab"
  description = "Complete guide for building IoT Weather Platform"

  # Theme variant (red, blue, green)
  themeVariant = "blue"

  # Disable search if needed
  disableSearch = false

  # Show visited links
  showVisitedLinks = true

# Enable search functionality
[outputs]
home = [ "HTML", "RSS", "JSON"]
```

### Theme Customization

To customize the appearance:

1. **Colors**: Modify `themeVariant` in config.toml
2. **Logo**: Add custom logo in `layouts/partials/logo.html`
3. **CSS**: Add custom styles in `static/css/custom.css`
4. **Favicon**: Place `favicon.png` in `static/images/`

## Content Guidelines

### Writing Content

- Use **Markdown** for all content files
- Follow the **Hugo Learn theme** conventions
- Include **front matter** in each file:

```yaml
---
title: "Page Title"
weight: 10
chapter: false
pre: "<b>1. </b>"
---
```

### Content Structure

- **Chapters**: Use `chapter: true` for section overviews
- **Pages**: Regular content pages with `chapter: false`
- **Weight**: Controls navigation order (lower = first)
- **Pre/Post**: Add prefixes/suffixes to menu items

### Shortcodes

The Learn theme provides useful shortcodes:

```markdown
{{% notice info %}}
Information box
{{% /notice %}}

{{% notice warning %}}
Warning message
{{% /notice %}}

{{% notice tip %}}
Helpful tip
{{% /notice %}}

{{% children %}}
List child pages
{{% /children %}}
```

## Deployment

### Local Build

```bash
# Build for production
hugo --minify

# Output will be in public/ folder
ls public/
```

### GitHub Pages

```bash
# Build and deploy to gh-pages branch
hugo --minify
# Copy public/ contents to gh-pages branch
```

### Manual Deployment

```bash
# Build the site
hugo --minify

# Upload public/ folder to your web server
rsync -avz public/ user@server:/var/www/html/
```

### Performance Issues

```bash
# Clear Hugo cache
hugo mod clean

# Disable live reload for large sites
hugo server --disableLiveReload

# Use fast render mode
hugo server --disableFastRender=false
```

## Resources

### Weather Platform Resources

- [AWS Amplify Gen 2](https://docs.amplify.aws/gen2/)
- [AWS IoT Core](https://docs.aws.amazon.com/iot/)
- [Next.js Documentation](https://nextjs.org/docs)

## Support

For issues with:

- **Tutorial content**: Create an issue in this repository
- **Hugo setup**: Check [Hugo documentation](https://gohugo.io/troubleshooting/)
- **Theme problems**: Visit [Learn theme docs](https://learn.netlify.app/en/)

## License

This tutorial is licensed under [MIT License](LICENSE). The Weather Platform code is available under the same license.
