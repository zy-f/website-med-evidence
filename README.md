# Deploy Doks to GitHub Pages

Example repo showing how to publish your Doks site to GitHub Pages — automatically

1. Add `.github/workflows/deploy-github.yml`:

```yml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
# Remmeber to enable Github pages!
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.127.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb          
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v4
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
          TZ: America/Los_Angeles
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"          
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

```

2. Click on the __Actions__ tab of your GitHub repo and wait for the action to finish succesfully (after approximately 30 seconds).

3. Go to the __Settings__ tab of your GitHub repo, and next to the __Pages__ section. Select branche `gh-pages` and click __Save__.
4. Copy the __Your site is published at__ URL and paste it as `baseurl` in `./config/production/config.toml`.
5. Set `canonifyURLs = true` in `./config/production/config.toml`.
6. Push the changes to GitHub and wait for the action to finish succesfully (after approximately 30 seconds).
7. That's it. After a minute or so, you site is avaliable at the __Your site is published at__ URL.

Now, after every push to the master branch, your site will be updated — automatically.

Dont forget to update parameters at: 
```uBench-website/config/_default/params.toml```

