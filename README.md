# 📝 Deployment Workflow

This repository contains the source code for our documentation, which is
automatically synchronized and deployed to **GitHub Pages** via a cross-platform
GitLab CI/CD pipeline.

## 🏗 System Architecture

We utilize a "Double-Repo" strategy to leverage GitLab's CI/CD power while
hosting on GitHub's global CDN:

1.  **Source Repository (GitLab):** The "Source of Truth." All development, code
    reviews, and CI testing happen here.
2.  **Deployment Repository (GitHub):** The "Production Host." This repository
    only contains the compiled static assets (`build/` folder) on the `gh-pages`
    branch.
3.  **Live Site:** Published at `https://<your-github-handle>.github.io/<repo-name>/`

---

## 🚀 The Publishing Process

The deployment is fully automated. You do not need to push to GitHub manually.

### 1\. Quality & Test (GitLab)

On every push to `main`, GitLab CI runs:

-   **Security Audit:** Checks `npm` dependencies for vulnerabilities.
-   **Linting:** Validates code style and Docusaurus heading IDs.
-   **Tests:** Runs the test suite to ensure site logic is sound.

### 2\. Build Stage

If tests pass, the `build-docusaurus` job executes:

This generates a production-ready `build/` directory containing static HTML, JS,
and CSS.

### 3\. Cross-Platform Deploy (GitLab ➡️ GitHub)

The `deploy-to-github` job performs the following:

-   Initializes a temporary Git instance inside the `build/` folder.
-   Connects to GitHub using a **Personal Access Token (PAT)**.
-   **Force-pushes** the content to the `gh-pages` branch on GitHub.

---

## 🛠 Technical Requirements

GitLab CI Variables

The following variables must be configured in **GitLab > Settings > CI/CD > Variables**:

-   `GITHUB_TOKEN`: A GitHub PAT with `repo` scope.
-   `GITHUB_REPO_URL`: The destination host (e.g., `github.com`).

**Note concerning these variables:**

- The `GITHUB_REPO_URL` is only required if the `GITHUB_TOKEN` is NOT
  repository-related and used for several repositories.
- If the `GITHUB_TOKEN` is for the repository only it's enough to use
  `@github.com` at the end of the command, which could be directly written
  in the workflow file.

## ⚙️ Docusaurus Configuration

Ensure your `website/docusaurus.config.js` matches the GitHub destination:

```
module.exports = {
  url: 'https://<username>.github.io',
  baseUrl: '/<repo-name>/',
  organizationName: '<username>',
  projectName: '<repo-name>',
  trailingSlash: false,
};
```

Especially `url` and `baseUrl` are important for Docusaurus to create all links
correctly, including the links to stylesheets and scripts.

---

## 🧹 Maintenance & Squashing

To keep the history professional for external stakeholders:

-   **Squashing:** Periodically squash development commits into a single
    "Feature" or "Release" commit before pushing to `main`.
-   **Cleaning:** If the pipeline history becomes cluttered, the pipeline logs
    can be purged via the GitLab API to maintain a clean "Production-Ready"
    appearance.

---

> **Note for Contributors:** Always branch from `main` on GitLab. Never push
    directly to the GitHub repository, as it will be overwritten by the next
    GitLab CI run.
