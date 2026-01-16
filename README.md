# shxdow's notebook

Personal blog built with [Hugo](https://gohugo.io/), a static site generator.

## Development

### Prerequisites

- [Hugo](https://gohugo.io/installation/) (extended version recommended for SCSS support)

### Local Development

```bash
# Install Hugo (if not already installed)
# macOS: brew install hugo
# Linux: see https://gohugo.io/installation/

# Run development server
hugo server

# Build site
hugo

# Build with clean destination
hugo --cleanDestinationDir
```

The site will be available at `http://localhost:1313` by default.

## Project Structure

```
.
├── assets/          # Source files (SCSS, processed by Hugo)
│   ├── css/        # Main stylesheet entry point
│   └── sass/       # SCSS partials
├── content/         # Content files (Markdown)
│   └── blog/       # Blog posts
├── layouts/         # HTML templates
│   ├── _default/   # Default page templates
│   ├── blog/       # Blog-specific templates
│   └── partials/   # Reusable template components
├── static/          # Static files (images, copied as-is)
│   └── assets/     # Images and other static assets
└── config.toml     # Hugo configuration
```

## Features

- Minimal, readable design inspired by Practical Typography
- System theme support (dark/light mode)
- Accessible (screen reader friendly, keyboard navigation)
- MathJax support for mathematical notation
- RSS feed
- Responsive design

## Deployment

The `public/` directory contains the generated static site. Deploy this directory to any static hosting service (GitHub Pages, Netlify, Vercel, etc.).

## License

Content is copyright. Code is available for reference.
