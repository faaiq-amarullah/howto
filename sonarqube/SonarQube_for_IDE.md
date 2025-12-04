# Run Code Analysis with SonarQube for IDE

## Avilabale Platforms

- **IntelliJ** (and other JetBrains IDEs)
- **Visual Studio**
- **VS Code**
- **Eclipse**

## Supported IDEs from VS Code forks

| **Supported IDE**                                                                                         | **Profile migration is available**                                                                                                      |
|-----------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| [VS Code](https://code.visualstudio.com/docs/setup/setup-overview)                                        | None required                                                                                                                           |
| [Cursor](https://docs.cursor.com/guides/migration/vscode)                                                 | ✅                                                                                                                                       |
| Gitpod                                                                                                    | [Install your extension](https://www.gitpod.io/docs/classic/user/references/ides-and-editors/vscode-extensions#installing-an-extension) |
| [Kiro](https://kiro.dev/docs/guides/migrating-from-vscode/)                                               | ✅                                                                                                                                       |
| [Trae](https://docs.trae.ai/ide/manage-extensions)                                                        | [Manage your extenstion](https://docs.trae.ai/ide/manage-extensions)                                                                    |
| [VSCodium](https://github.com/VSCodium/vscodium/blob/master/docs/migration.md)                            | ✅                                                                                                                                       |
| [Windsurf](https://docs.windsurf.com/windsurf/getting-started#forgot-to-import-vs-code-configurations%3F) | ✅                                                                                                                                       |

## System Requirements

SonarQube for VS Code needs a **Java Runtime (JRE)** 17+.

Automatic JRE Search (priority order):
  1. `sonarlint.ls.javaHome` setting in VS Code
  2. Embedded JRE for specific platforms
  3. `JDK_HOME` environment variable
  4. `JAVA_HOME` environment variable
  5. Windows Registry (Windows only)
  6. PATH search for `javac`

If JRE not found: SonarQube for IDE will ask permission to download its own version

## Language-Specific Requirements

### JavaScript/TypeScript/CSS Analysis

- **Node.js**: Version 20.12.0+ (v20) or 22.11.0+ (v22)
- **Optional configuration**:

```json
{
  "sonarlint.pathToNodeExecutable": "/path/to/node"
}
```

### C and C++ Analysis

- **CFamily Analyzer**: Automatically downloaded after installation
- **Required**: Compilation database (compile_commands.json)
```json
{
  "sonarlint.pathToCompileCommands": "/path/to/compile_commands.json"
}
```

- **Microsoft Visual C++ Compiler**: Must be ready to build (run VS Code from Visual Studio Command Prompt)

### COBOL (Connected mode only)

- **Availability**: SonarQube for VS Code v3.19+
- **Prerequisites**: Must run with SonarQube Server Enterprise Edition+ or SonarQube Cloud
- **Server configurations synchronized**:
  - `sonar.cobol.dialect`
  - `sonar.cobol.file.suffixes`
  - `sonar.cobol.sourceFormat`
  - `sonar.cobol.copy.suffixes`
  - `sonar.cobol.copy.directories`

- **VS Code Language Mode**: Must be set to COBOL

### C#

- **Built-in analysis**: Available in VS Code v4.0+ (Microsoft IDE)
- **VS Code forks**: Must install manually from .vsix file (OpenVSX marketplace)

### Infrastructure as Code (IaC) (SonarQube for VS Code v3.17+)

Supported languages:
- Azure Resource Manager
- CloudFormation
- Docker
- Kubernetes
- Terraform

### Java

- **Required extension**: [Language support for Java](https://marketplace.visualstudio.com/items?itemName=redhat.java) VS Code extension (v0.56.0+)
- **Mode**: Must be in standard mode

### Apex (Connected mode only)

- **Availability**: SonarQube Server Enterprise Edition or SonarQube Cloud
- **Required extension**: [Salesforce Extension Pack](https://marketplace.visualstudio.com/items?itemName=salesforce.salesforcedx-vscode)

### PL/SQL (Connected mode only)

- **Availability**: Commercial editions of SonarQube Server or SonarQube Cloud
- **Required extension**: [Oracle Developer Tools for VSCode](https://marketplace.visualstudio.com/items?itemName=Oracle.oracledevtools)

### T-SQL (Connected mode only)

- **Availability**: Commercial editions of SonarQube Server or SonarQube Cloud
- **Configuration**: Server-side configuration required

## Installation

SonarQube for VS Code can be installed like any other VS Code extension as explained in the VS Code documentation. The standard installation workflow described below works for VS Code, including the Cursor and Trae editors, as well as VSCodium, GitHub Codespaces, and GitPod, among others:
1. Select **Extensions** in the left sidebar of your VS Code app.
2. Enter `SonarQube for IDE` in the search bar.
3. Select **Install**.

![SonarQube for VS Code](https://docs.sonarsource.com/~gitbook/image?url=https%3A%2F%2F3457378997-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F6LPRABg3ubAJhpfR5K0Y%252Fuploads%252Fgit-blob-f8523c4fc016918b33b52d98202e99e86e3f4a9e%252Fcf7c85916358e5b25041b376d8e00f8765586a37.png%3Falt%3Dmedia&width=768&dpr=2&quality=100&sign=47f700c9&sv=2)


## Connected Mode Setup

### Prerequisites

Before connecting, you need:
1. Access to a running SonarQube Server (local or company) or SonarQube Cloud account
2. Authentication Tokens generated from the SonarQube platform
3. Project Analysis must have been run at least once on the SonarQube server/cloud

### Connection Types

#### SonarQube Server (Self-hosted)
- Requires Server URL (e.g., http://localhost:9000 or company URL)
- May need network access and proper authentication

#### SonarQube Cloud (Hosted service)
- URL is pre-configured
- Uses SonarCloud organization/project structure

### Step 1: Generate Tokens

For SonarQube Server:
1. Generate a User Token for IDE connection (Administration → Security → Users → Tokens)
2. Generate a Project Analysis Token for specific project binding

For SonarQube Cloud:
1. Generate token from SonarCloud → Account → Security
2. Same token often serves both connection and project binding

### Step 2: Establish Connection in VS Code
1. Open Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`)
2. Run: `SonarQube: Connect to SonarQube Server`
3. Enter the **Server URL**, **User Token**, and **Connection Name**
4. Click **Save Connection**

### Step 3: Verify Connection
1. Open Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`)
2. Run: `SonarQube Setup: Focus on Connected Mode View`
3. Check if there was an available SonarQube Connection has been saved

### Step 4: Run Analysis
1. Open file that will be the target for analysis
2. Open Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`)
3. Run: `SonarQube: Analyze Current File with SonarQube`
