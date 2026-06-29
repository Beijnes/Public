# Roel Beijnes Technical Blog

A modern, Jekyll-powered blog built with the [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) theme and hosted on GitHub Pages.

## Topics

- End User Computing strategy and operations
- Application packaging and delivery (MSI, App-V, ThinApp, VMware App Volumes)
- PowerShell automation
- Recast Application Workspace
- REST API integration
- AI-assisted engineering with GitHub Copilot

## Setup & Development

### Prerequisites

- Ruby 3.0 or later
- Bundler

### Local Development

1. Clone the repository:
```bash
git clone https://github.com/roelbeijnes/public.git
cd public
```

2. Install dependencies:
```bash
bundle install
```

3. Run the development server:
```bash
bundle exec jekyll serve
```

4. Open [http://localhost:4000](http://localhost:4000) in your browser

### Creating a New Post

Blog posts go in the `_posts` directory with the naming convention: `YYYY-MM-DD-title-of-post.md`

Example front matter:
```markdown
---
title: Your Post Title
description: A brief description of your post
date: YYYY-MM-DD
categories: [Category1, Category2]
tags: [tag1, tag2, tag3]
image:
  path: /assets/img/blog/post-name/image.png
---
```

## Deployment

This blog is automatically deployed to GitHub Pages via GitHub Actions when you push to the `main` branch.

### Custom Domain

To use a custom domain:

1. Add your domain to the `url` field in `_config.yml`
2. Create a `CNAME` file in the root with your domain name
3. Update your domain's DNS settings to point to GitHub Pages

## Theme

Built with [Chirpy Jekyll Theme](https://github.com/cotes2020/jekyll-theme-chirpy) - a beautiful, feature-rich Jekyll theme perfect for blogs.

## License

The content of this blog is my own. The Chirpy theme is licensed under the MIT License.

---

*Questions or suggestions? Feel free to open an issue or reach out on LinkedIn.*
