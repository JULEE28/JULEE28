- 👋 Hi, I’m @JULEE28
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
JULEE28/JULEE28 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
# `dist/index.js` is a special file in Actions.
# When you reference an action with `uses:` in a workflow,
# `index.js` is the code that will run.
# For our project, we generate this file through a build process
# from other source files.
# We need to make sure the checked-in `index.js` actually matches what we expect it to be.
name: Check dist/

on:
  push:
    branches:
      - main
    paths-ignore:
      - '**.md'
  pull_request:
    paths-ignore:
      - '**.md'
  workflow_dispatch:

jobs:
  check-dist:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Set Node.js 12.x
        uses: actions/setup-node@v1
        with:
          node-version: 12.x

      - name: Install dependencies
        run: npm ci

      - name: Rebuild the dist/ directory
        run: npm run build

      - name: Compare the expected and actual dist/ directories
        run: |
          if [ "$(git diff --ignore-space-at-eol dist/ | wc -l)" -gt "0" ]; then
            echo "Detected uncommitted changes after build.  See status below:"
            git diff
            exit 1
          fi
        id: diff

      # If index.js was different than expected, upload the expected version as an artifact
      - uses: actions/upload-artifact@v2
        if: ${{ failure() && steps.diff.conclusion == 'failure' }}
        with:
          name: dist
          path: dist/
