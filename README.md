# blog
A personal blog built with Material for MkDocs

## Getting Started

### Prerequisites

- Python 3.8 or higher
- pip (Python package installer)

### Installation

1. Clone this repository
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

### Usage

#### Serve locally

To preview your blog locally with live reload:

```bash
mkdocs serve
```

Then open your browser to `http://127.0.0.1:8000/`

#### Build the site

To build the static site:

```bash
mkdocs build
```

The generated site will be in the `site/` directory.

### Structure

```
.
├── docs/                   # Documentation source files
│   ├── index.md           # Homepage
│   └── blog/              # Blog section
│       ├── index.md       # Blog index
│       ├── .authors.yml   # Blog authors configuration
│       └── posts/         # Blog posts
├── mkdocs.yml             # MkDocs configuration
└── requirements.txt       # Python dependencies
```

### Creating Blog Posts

To create a new blog post:

1. Create a new Markdown file in `docs/blog/posts/`
2. Add front matter with date, authors, and categories:
   ```yaml
   ---
   date: 2024-01-01
   authors:
     - default
   categories:
     - General
   ---
   ```
3. Write your content using Markdown
4. Use `<!-- more -->` to separate the excerpt from the full content

### Customization

Edit `mkdocs.yml` to customize:
- Site name, description, and author
- Theme colors and features
- Navigation structure
- Plugins and extensions

## Documentation

For more information, see the [Material for MkDocs documentation](https://squidfunk.github.io/mkdocs-material/).
