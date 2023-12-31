# publish-js-to-pages

Workflow any JavaScript/TypeScript application (including React apps) to GitHub Pages

## Usage

First, configure your repository to publish Pages from Actions:

1. Navigate to the "Pages" section of the repository settings
2. Under "Build and deployment" set the "Source" to "GitHub Actions"

Then, create a new workflow file in your repository. Here's an example that will work for many cases:

**/.github/workflows/publish-pages.yml**
```yml
name: Deploy Pages site

on:
  push:
    branches: main

# Important! You must grant the required permission for the action to be able to deploy
permissions:
  contents: read
  pages: write
  id-token: write

# Prevents conflicting deploys when several merges happen quickly
concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - id: deployment # Add an ID to the step so we can get the URL on completion
        uses: iansan5653/publish-js-to-pages@1.0.1 # Note: GitHub recommends tagging this to a full commit hash
```

### Inputs

The action is configured to deploy `create-react-app` applications by default. This means that it assumes:

- You run `npm ci` to install dependencies
- You run `npm run build` to build the app
- When built, the compiled files are located in `/build`

If any of these are not true, it's easy to override them. For example:

```yml
    steps:
      - id: deployment
        uses: iansan5653/publish-js-to-pages
        with:
          build_command: npm run compile
          build_output_directory: output
```

For the full documentation of inputs, see the `inputs` field in [`action.yml`](https://github.com/iansan5653/publish-js-to-pages/blob/main/action.yml).

### Further configuration

This action is a handy starting point but it's not very complex -- you will get the same results if you replace `steps` in the above workflow file with the full list of steps below. This may be useful if you need more control than the action allows.

```yml
    - uses: actions/checkout@v3
    
    - uses: actions/setup-node@v3
      with:
        node-version: 18
    
    - run: |
        npm ci
        npm run build
      shell: bash
    
    - uses: actions/configure-pages@v3
    
    - uses: actions/upload-pages-artifact@v2
      with:
        path: build
    
    - id: deployment
      uses: actions/deploy-pages@v2
```
