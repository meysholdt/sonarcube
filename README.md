# SonarQube + Ona Integration

This repository demonstrates four ways to integrate SonarQube into your development workflow using [Ona](https://ona.com) environments:

1. **SonarLint VS Code Extension** — catch issues in real-time as you code
2. **SonarQube MCP Server** — analyze and fix findings interactively before committing
3. **SonarQube MCP Server** — bulk-triage and clean up issue backlogs across a project
4. **Ona Automations** — generate tested pull requests for findings automatically in the background

> **Video walkthrough:** [Watch the demo on Loom](https://www.loom.com/share/2610df96e615488fbb9fe5f05b547a3d)

---

## Prerequisites

Set these as [Ona secrets](https://ona.com/docs/ona/configuration/secrets/overview) before starting the environment:

| Variable | Description |
|----------|-------------|
| `SONARQUBE_TOKEN` | Your SonarQube Cloud [user token](https://sonarcloud.io/account/security) |
| `SONARQUBE_ORG` | Your SonarQube Cloud [organization key](https://sonarcloud.io/account/organizations) |

The MCP server and SonarLint connected mode require both values.

---

## 1. SonarLint VS Code Extension

The devcontainer includes the [SonarLint extension](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarlint-vscode), which highlights issues directly in the editor as you type — before you ever commit.

### Setup

SonarLint connected mode is pre-configured in `.sonarlint/connectedMode.json`. To activate it:

1. Open the Command Palette (`Ctrl+Shift+P`) → **SonarLint: Add SonarQube Cloud Connection**
2. Enter your organization key (`ona-samples`) and token
3. The workspace is already bound to the project — SonarLint will sync rules automatically

Once connected, SonarLint uses the same rule set as your SonarQube Cloud project, so local findings match what CI would report.

### Real-time feedback while coding

SonarLint underlines issues inline and shows details in the Problems panel:

<!-- TODO: Add screenshot of SonarLint highlighting an issue inline in the editor -->
<!-- ![SonarLint inline issue](docs/screenshots/sonarlint-inline.png) -->

The SonarLint panel shows rule descriptions and suggested fixes:

<!-- TODO: Add screenshot of SonarLint rule description panel -->
<!-- ![SonarLint rule panel](docs/screenshots/sonarlint-rule-panel.png) -->

---

## 2. SonarQube MCP Server

The devcontainer includes a [SonarQube MCP server](https://docs.sonarsource.com/sonarqube-mcp-server/) that connects Ona directly to your SonarQube Cloud account. This gives Ona access to project issues, rules, and analysis tools through natural language.

### Setup

The MCP server is configured in [`.ona/mcp-config.json`](.ona/mcp-config.json). It runs the `mcp/sonarqube` Docker image and passes your credentials via environment variables:

```json
{
  "mcpServers": {
    "sonarqube": {
      "command": "docker",
      "args": ["run", "-i", "--init", "--name", "sonarqube-mcp-server", "--rm",
               "-e", "SONARQUBE_TOKEN", "-e", "SONARQUBE_ORG", "mcp/sonarqube"],
      "env": {
        "SONARQUBE_TOKEN": "${exec:printenv SONARQUBE_TOKEN}",
        "SONARQUBE_ORG": "${exec:printenv SONARQUBE_ORG}"
      },
      "timeout": 30
    }
  }
}
```

The server starts automatically when the environment launches. No additional setup is needed — Ona can query SonarQube as soon as the environment is running.

To verify the connection, ask Ona: *"Find my SonarQube projects"*.

### Analyze and fix findings before committing

1. Make changes to your code
2. Ask Ona: *"Verify my uncommitted changes via SonarQube"*
3. Ona runs SonarQube analysis on the changed files and reports issues with severity, rule, and line numbers
4. Ask Ona to fix specific issues: *"Fix all blockers"* or *"Fix all issues in this file"*
5. Ona applies fixes, verifies compilation, and re-analyzes to confirm resolution

This catches security vulnerabilities, bugs, and code smells before they reach your branch — without leaving the editor.

### Bulk issue backlog cleansing

Beyond pre-commit checks, the MCP integration supports triaging and fixing existing issues across the entire project.

1. Ask Ona: *"Find all SonarQube issues on this repo and give me a breakdown"*
2. Ona fetches all open issues from SonarQube Cloud and categorizes them by area, severity, and rule
3. Prioritize by area: *"Fix all SonarQube issues from the owner package"*
4. Ona reads the affected files, applies fixes, runs the formatter, compiles, and runs tests
5. Repeat for other packages or severity levels

This is effective for reducing technical debt across a codebase in a single session — Ona handles the mechanical fixes while you review the results.

---

## 3. Ona Automations

Ona automations can run SonarQube analysis as part of your development lifecycle and produce pull requests with fixes automatically.

### Setup

The automation is defined in [`.ona/fix-sonar-issue.yaml`](.ona/fix-sonar-issue.yaml). It uses a multi-step agent workflow that:

1. Queries SonarQube for the highest-severity open issue
2. Creates a fix branch and applies a minimal code change
3. Runs `./mvnw compile test` to verify the fix
4. Commits and opens a pull request with a structured description

To register the automation with Ona:

```bash
ona ai automation create .ona/fix-sonar-issue.yaml
```

To update the automation after editing the YAML (replace `<automation-id>` with the ID returned by `create`):

```bash
ona ai automation update <automation-id> .ona/fix-sonar-issue.yaml
```

### Generating tested PRs for findings in the background

Once registered, the automation can be triggered manually or on a schedule. Each run picks the highest-severity issue, fixes it, verifies tests pass, and opens a PR — you review and merge.

This turns SonarQube findings into a continuous improvement loop — issues get fixed in the background without interrupting your feature work.
