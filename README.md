# Automatic GitHub Releases of packages built with Vercel `pkg` using GitHub Actions

## Introduction

Vercel's `pkg` allows packaging a Node.js project into an executable, which can run on devices which don't have Node.js installed.

GitHub Actions can run code on a schedule, allowing for automatically building your `pkg` and releasing to GitHub Releases.

This is a guide to show this. I tried contributing this to the official docs, but unfortunately this was declined by Vercel: https://github.com/vercel/pkg/pull/1246

## Guide

[GitHub Actions](https://docs.github.com/en/actions) can be used to automatically run [`pkg`](https://github.com/vercel/pkg) when a specific event has occurred.

For example, to automatically run `pkg` to create executables and upload them as assets to a new release on GitHub every time a new Git tag is pushed, try setting up [`softprops/action-gh-release`](https://github.com/softprops/action-gh-release) as follows:

1. Add a new file in your project called `.github/workflows/release.yml` (create the directories `.github/workflow` in case they don't exist in your project) and add this file content:

```yaml
name: Build and release
on:
  push:
    tags:
      - 'v*.*.*'
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Use Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      - run: yarn --frozen-lockfile
      - run: npm install --global pkg
      - name: Build
        run: pkg index.js --output your-program-name-here --targets linux,macos,win
      - name: Release
        uses: softprops/action-gh-release@v1
        with:
          files: |
            your-program-name-here-linux
            your-program-name-here-macos
            your-program-name-here-win.exe
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

2. Create a new Git tag with the version that you want:

```bash
git tag -a v1.0.0
```

3. Push the new tag to GitHub:

```bash
git push --tags
```

This will create a new release under Releases on your GitHub repo that will contain the built executables as assets:

![Releases page on GitHub repo showing release with assets](https://user-images.githubusercontent.com/1935696/124388004-fe2bcb00-dce0-11eb-9701-78f06b31b23f.png)

Every time that you want to create a new release with assets, create a new tag and push it (steps 2 and 3 above).
