# SonarQube

## SonarQube MCP Server

This devcontainer includes a [SonarQube MCP server](https://docs.sonarsource.com/sonarqube-mcp-server/) that connects Ona to your SonarQube Cloud account.

### Required environment variables

Set these before starting the environment:

| Variable | Description |
|----------|-------------|
| `SONARQUBE_TOKEN` | Your SonarQube Cloud [user token](https://sonarcloud.io/account/security) |
| `SONARQUBE_ORG` | Your SonarQube Cloud [organization key](https://sonarcloud.io/account/organizations) |

Configure them as Ona secrets or environment variables so they're available inside the devcontainer. The MCP server will not start without both values set.

## Local SonarQube Scanning

Two options are available for running SonarQube analysis locally. Both sync rules from your SonarQube Cloud project.

### Option 1: Maven Plugin

The `sonar-maven-plugin` is configured in `pom.xml`. This is the recommended approach for Java/Maven projects — it integrates with the build lifecycle and automatically picks up compiled classes and JaCoCo coverage reports.

```bash
# Run analysis and upload results to SonarQube Cloud
./mvnw verify sonar:sonar \
  -Dsonar.host.url=https://sonarcloud.io \
  -Dsonar.organization=$SONARQUBE_ORG \
  -Dsonar.token=$SONARQUBE_TOKEN \
  -Dsonar.projectKey=ona-samples_sonarcube-integration

# Skip tests if already run separately
./mvnw sonar:sonar \
  -Dsonar.host.url=https://sonarcloud.io \
  -Dsonar.organization=$SONARQUBE_ORG \
  -Dsonar.token=$SONARQUBE_TOKEN \
  -Dsonar.projectKey=ona-samples_sonarcube-integration
```

Note: If SonarCloud has Automatic Analysis enabled for this project, disable it in **Administration → Analysis Method** before running manual scans.

The `verify` phase runs tests and generates the JaCoCo coverage report, which SonarQube then picks up automatically.

### Option 2: SonarScanner CLI

The SonarScanner CLI is pre-installed in the devcontainer. It's a standalone scanner useful for quick scans or non-Maven workflows.

```bash
sonar-scanner \
  -Dsonar.host.url=https://sonarcloud.io \
  -Dsonar.organization=$SONARQUBE_ORG \
  -Dsonar.token=$SONARQUBE_TOKEN \
  -Dsonar.projectKey=ona-samples_sonarcube-integration \
  -Dsonar.sources=src/main/java \
  -Dsonar.tests=src/test/java \
  -Dsonar.java.binaries=target/classes
```

Note: The CLI requires compiled classes (`target/classes`). Run `./mvnw compile` first if they don't exist.

### Which to use?

| | Maven Plugin | SonarScanner CLI |
|---|---|---|
| **Best for** | Full analysis with coverage | Quick scans, non-Maven projects |
| **Coverage support** | Automatic (via JaCoCo) | Manual configuration required |
| **Setup** | Zero — configured in `pom.xml` | Requires passing project parameters |
| **Uploads results** | Yes | Yes |

### VS Code: SonarLint Extension

The devcontainer includes the [SonarLint extension](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarlint-vscode) for real-time feedback in the editor. To sync rules from SonarQube Cloud, enable Connected Mode in VS Code settings:

1. Open Command Palette → **SonarLint: Add SonarQube Cloud Connection**
2. Enter your organization key and token
3. Bind the workspace to project `ona-samples_sonarcube-integration`

SonarLint catches issues as you type. Use the Maven plugin or CLI for full project analysis.