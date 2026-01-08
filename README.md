# LGTM Stack Documentation

This branch (`gh-pages`) contains the automatically generated documentation website for the LGTM Stack project.

## 📖 About This Branch

- **Purpose**: Hosts the live documentation website via GitHub Pages
- **Source**: Built from MkDocs source files in the `main` branch
- **URL**: https://hardik-sankhla.github.io/LGTM_Microservices_Observability_Stack/

## 🏗️ Build Process

This branch is automatically updated when documentation changes are made to the `main` branch:

1. Edit source files in `docs-mkdocs/docs/` (main branch)
2. MkDocs builds the site to `docs-mkdocs/site/`
3. Built files are copied to this branch
4. Changes are committed and pushed to `gh-pages`

## 📁 Branch Contents

- `index.html` - Documentation homepage
- `assets/` - CSS, JavaScript, and image files
- `architecture/`, `components/`, etc. - Documentation sections
- `sitemap.xml` - SEO sitemap
- `404.html` - Custom 404 error page

## 🔄 Development

To modify the documentation:

1. Switch to the `main` branch
2. Edit files in `docs-mkdocs/docs/`
3. Run `mkdocs build --clean` to build locally
4. Commit changes to trigger automatic deployment

## 📋 Documentation Structure

- **Getting Started** - Quick start guides and installation
- **Architecture** - System design and data flow
- **Components** - Detailed component documentation
- **Configuration** - Setup and customization
- **Examples** - Language-specific integration examples
- **Operations** - Troubleshooting and maintenance

## 🤝 Contributing

Documentation contributions are welcome! Please see the [Contributing Guide](https://hardik-sankhla.github.io/LGTM_Microservices_Observability_Stack/contributing/development/) for details.

---

*This branch is automatically maintained. Do not edit files directly here.*