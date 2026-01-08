# LGTM Stack Documentation

Professional documentation website for the LGTM Stack - a comprehensive observability platform built with Grafana, OpenTelemetry, Loki, Tempo, Prometheus, and Pyroscope.

## 🚀 Quick Start

### Prerequisites

- Python 3.8+
- pip

### Local Development

1. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

2. **Serve documentation locally:**
   ```bash
   mkdocs serve
   ```

3. **Open your browser:**
   Visit `http://localhost:4000` to view the documentation.

### Build for Production

```bash
mkdocs build
```

The built site will be in the `site/` directory.

## 📁 Project Structure

```
docs-mkdocs/
├── docs/                    # Documentation source files
│   ├── assets/             # Images, logos, and static assets
│   ├── api/                # API reference documentation
│   ├── components/         # Component documentation
│   ├── configuration/      # Configuration guides
│   ├── contributing/       # Contribution guidelines
│   ├── examples/           # Code examples and tutorials
│   ├── operations/         # Operations and maintenance
│   ├── production/         # Production deployment guides
│   ├── index.md           # Homepage
│   └── *.md               # Other documentation pages
├── mkdocs.yml             # MkDocs configuration
├── requirements.txt       # Python dependencies
└── README.md             # This file
```

## 🎨 Features

- **Material Design**: Modern, responsive design with Material Design principles
- **Dark/Light Mode**: Automatic theme switching based on user preference
- **Search**: Full-text search across all documentation
- **Code Highlighting**: Syntax highlighting for code blocks with copy functionality
- **Navigation**: Intuitive navigation with sections and subsections
- **Mobile Responsive**: Optimized for all device sizes
- **Fast Loading**: Optimized assets and minification

## 📝 Writing Documentation

### File Structure

- Use Markdown (`.md`) files
- Organize content in logical directories under `docs/`
- Update `mkdocs.yml` navigation when adding new pages

### Markdown Extensions

The documentation supports extended Markdown features:

- **Admonitions**: `!!! note`, `!!! warning`, `!!! tip`
- **Code blocks**: With syntax highlighting and copy buttons
- **Tables**: Standard Markdown tables
- **Footnotes**: `[^1]` style footnotes
- **Task lists**: `- [x]` and `- [ ]` checkboxes
- **Emoji**: `:smile:` shortcodes

### Front Matter

Add metadata to your pages:

```yaml
---
title: Page Title
description: Brief description for SEO
---
```

## 🚀 Deployment

### GitHub Pages (Recommended)

1. **Enable GitHub Pages** in your repository settings:
   - Go to Settings → Pages
   - Source: "GitHub Actions"

2. **The GitHub Actions workflow** will automatically:
   - Build the documentation on every push to main
   - Deploy to GitHub Pages
   - Make it available at `https://yourusername.github.io/repository-name/`

### Manual Deployment

1. Build the site:
   ```bash
   mkdocs build
   ```

2. Deploy the `site/` directory to your hosting provider.

## 🔧 Customization

### Theme Configuration

Edit `mkdocs.yml` to customize:

- **Colors**: Primary and accent colors
- **Fonts**: Text and code fonts
- **Features**: Enable/disable navigation features
- **Logo**: Add your custom logo

### Adding Plugins

1. Install the plugin:
   ```bash
   pip install plugin-name
   ```

2. Add to `requirements.txt`

3. Configure in `mkdocs.yml`:
   ```yaml
   plugins:
     - plugin-name:
         option: value
   ```

## 🤝 Contributing

1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/new-section`
3. **Make your changes**
4. **Test locally**: `mkdocs serve`
5. **Commit your changes**: `git commit -am 'Add new section'`
6. **Push to the branch**: `git push origin feature/new-section`
7. **Create a Pull Request**

## 📄 License

This documentation is part of the LGTM Stack project. See the main repository for licensing information.

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/Hardik-Sankhla/LGTM_Microservices_Observability_Stack/issues)
- **Discussions**: [GitHub Discussions](https://github.com/Hardik-Sankhla/LGTM_Microservices_Observability_Stack/discussions)

---

Built with ❤️ using [MkDocs](https://www.mkdocs.org/) and [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)