# Github Action Example with CodeQL

This repository demonstrates the integration of GitHub Actions with CodeQL for automated security analysis of your codebase.

## Overview

This project showcases the implementation of GitHub Actions using CodeQL, enabling automated code scanning and vulnerability detection in your software projects. CodeQL is a powerful static analysis tool used to identify security vulnerabilities, potential bugs, and code quality issues.

## Repository Structure

- **`.github/workflows/codeql.yaml`:** Contains the configuration for the GitHub Actions workflow responsible for triggering CodeQL analysis.
- **`.github/codeql-config.yaml`:** Configuration for CodeQL selection of query suites and filtering settings.
- **`vulnerable-source-code/`:** Directory housing your project's source code files.

## CodeQL Configuration

```yaml
name: "CodeQL config"

disable-default-queries: false

queries:
  - uses: security-and-quality
  - name: Enable all security queries
    uses: security-extended

query-filters:
  - exclude:
      problem.severity:
        - warning
        - recommendation
```

## Github Action file

```yaml
# For most projects, this workflow file will not need changing; you simply need
# to commit it to your repository.
#
# You may wish to alter this file to override the set of languages analyzed,
# or to provide custom queries or build logic.
#
# ******** NOTE ********
# We have attempted to detect the languages in your repository. Please check
# the `language` matrix defined below to confirm you have the correct set of
# supported CodeQL languages.
#
name: "CodeQL"

on:
  push:
    branches: [main]
  # # Scan changed files in PRs (diff-aware scanning):
  # pull_request: {}
  pull_request:
    branches: [main]
    # Run checks on reopened draft PRs to support triggering PR checks on draft PRs that were opened
    # by other workflows.
    types: [opened, synchronize, reopened, ready_for_review]
  # schedule:
  #   # Weekly on Sunday.
  #   - cron: '30 1 * * 0'

jobs:
  analyze:
    name: Analyze
    # Runner size impacts CodeQL analysis time. To learn more, please see:
    #   - https://gh.io/recommended-hardware-resources-for-running-codeql
    #   - https://gh.io/supported-runners-and-hardware-resources
    #   - https://gh.io/using-larger-runners
    # Consider using larger runners for possible analysis time improvements.
    runs-on: ${{ (matrix.language == 'swift' && 'macos-latest') || 'ubuntu-latest' }}
    timeout-minutes: ${{ (matrix.language == 'swift' && 120) || 360 }}
    permissions:
      actions: read
      contents: read
      security-events: write

    strategy:
      fail-fast: false
      matrix:
        language: [ 'python', 'go', 'javascript' ] # @NOTE: You may need to change depend on your repo

        # CodeQL supports [ 'cpp', 'csharp', 'go', 'java', 'javascript', 'python', 'ruby', 'swift' ]
        # Use only 'java' to analyze code written in Java, Kotlin or both
        # Use only 'javascript' to analyze code written in JavaScript, TypeScript or both
        # Learn more about CodeQL language support at https://aka.ms/codeql-docs/language-support

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    # Initializes the CodeQL tools for scanning.
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v2
      with:
        languages: ${{ matrix.language }}
        config-file: ./.github/codeql-config.yaml  # @NOTE: You may need to change depend on your repo

        # If you wish to specify custom queries, you can do so here or in a config file.
        # By default, queries listed here will override any specified in a config file.
        # Prefix the list here with "+" to use these queries and those in the config file.

        # For more details on CodeQL's query packs, refer to: https://docs.github.com/en/code-security/code-scanning/automatically-scanning-your-code-for-vulnerabilities-and-errors/configuring-code-scanning#using-queries-in-ql-packs
        # queries: security-extended,security-and-quality


    # Autobuild attempts to build any compiled languages (C/C++, C#, Go, Java, or Swift).
    # If this step fails, then you should remove it and run the build manually (see below)
    - name: Autobuild
      uses: github/codeql-action/autobuild@v2

    # ℹ️ Command-line programs to run using the OS shell.
    # 📚 See https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsrun

    #   If the Autobuild fails above, remove it and uncomment the following three lines.
    #   modify them (or add more) to build your code if your project, please refer to the EXAMPLE below for guidance.

    # - run: |
    #     echo "Run, Build Application using script"
    #     ./location_of_script_within_repo/buildscript.sh

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2
      with:
        category: "/language:${{matrix.language}}"
        # default will be output: /home/runner/work/sample-codeql-ci/results/
        output: ../analyze-results

    # Upload SARIF result to the GitHub Security Dashboard
    - uses: actions/upload-artifact@v3
      with:
        name: codeql-output.sarif
        path: /home/runner/work/sample-codeql-ci/analyze-results/
```