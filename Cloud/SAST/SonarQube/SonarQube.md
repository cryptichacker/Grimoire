---
tags: [cloud, sast, sonarqube]
type: cheatsheet
source: SonarQube product family — compiled reference
last-verified: 2026-08-25
---

# SonarQube

## Up
- [[SAST]]

SonarQube is a code-quality and SAST platform that analyses source for bugs, vulnerabilities, security hotspots, and code smells across 30+ languages, and enforces a **Quality Gate** in CI. The analysis is run by a **SonarScanner**, which uploads results to a **SonarQube Server** or **SonarQube Cloud** instance.

## Product family
| Product | Formerly | What it is |
|---|---|---|
| **SonarQube Server** | SonarQube | Self-managed server you host (Docker/zip/K8s) |
| **SonarQube Cloud** | SonarCloud | SaaS at `sonarcloud.io` |
| **SonarQube for IDE** | SonarLint | IDE plugin (VS Code, IntelliJ, Eclipse, Visual Studio) |
| **SonarScanner CLI** | | Generic command-line scanner (`sonar-scanner`) |
| **SonarScanner for Maven / Gradle / .NET / Ant** | | Build-tool-integrated scanners |

**Server editions:** Community (free, OSS) · Developer · Enterprise · Data Center.

## Run SonarQube Server (Docker)
```bash
# Start the Community edition
docker run -d --name sonarqube -p 9000:9000 sonarqube:community

# Open http://localhost:9000  — default login: admin / admin (you'll be forced to change it)
```
For production, use an external database (PostgreSQL) and raise the host's `vm.max_map_count`:
```bash
sudo sysctl -w vm.max_map_count=524288
```

## Tokens & authentication
Generate a token in the UI: **My Account → Security → Generate Token** (user, project, or global-analysis token). Pass it to any scanner via `sonar.token` (or the `SONAR_TOKEN` env var). `sonar.login` is the deprecated older name.

```bash
export SONAR_TOKEN="sqp_xxxxxxxx"       # server token prefix: squ_/sqp_/sqa_
export SONAR_HOST_URL="http://localhost:9000"
```

## SonarScanner CLI
For any project with no specific build tool. Configure via `sonar-project.properties` (below) or `-D` flags:
```bash
sonar-scanner \
  -Dsonar.projectKey=my_project \
  -Dsonar.sources=. \
  -Dsonar.host.url=$SONAR_HOST_URL \
  -Dsonar.token=$SONAR_TOKEN
```
Run it without installing anything, via Docker:
```bash
docker run --rm \
  -e SONAR_HOST_URL="$SONAR_HOST_URL" \
  -e SONAR_TOKEN="$SONAR_TOKEN" \
  -v "$(pwd):/usr/src" \
  sonarsource/sonar-scanner-cli
```

## Scanners per build tool
| Build tool | Command |
|---|---|
| **Maven** | `mvn clean verify sonar:sonar -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=$SONAR_TOKEN` |
| **Gradle** | add the `org.sonarqube` plugin, then `./gradlew sonar -Dsonar.token=$SONAR_TOKEN` |
| **.NET** | 3-step: `begin` → build → `end` (see below) |
| **Ant** | use the SonarScanner for Ant task, then `ant sonar` |
| **npm / JS / TS** | use the SonarScanner CLI (no dedicated npm scanner needed) |

**.NET flow:**
```bash
dotnet sonarscanner begin /k:"my_project" \
  /d:sonar.host.url="$SONAR_HOST_URL" /d:sonar.token="$SONAR_TOKEN"
dotnet build
dotnet sonarscanner end /d:sonar.token="$SONAR_TOKEN"
```

## sonar-project.properties (key parameters)
Placed in the project root; read by the CLI/Ant scanners (Maven/Gradle read from the build file instead).

| Property | Description |
|---|---|
| `sonar.projectKey` | **Required.** Unique project identifier |
| `sonar.projectName` | Display name |
| `sonar.projectVersion` | Version string |
| `sonar.sources` | Comma-separated source dirs (e.g. `src`) |
| `sonar.tests` | Test source dirs |
| `sonar.host.url` | Server URL (`http://localhost:9000`) |
| `sonar.token` | Auth token (prefer `SONAR_TOKEN` env) |
| `sonar.sourceEncoding` | e.g. `UTF-8` |
| `sonar.exclusions` | Glob(s) to exclude from analysis |
| `sonar.inclusions` | Restrict analysis to matching files |
| `sonar.coverage.exclusions` | Files to exclude from coverage metrics |
| `sonar.qualitygate.wait` | `true` → scanner polls and fails the build if the gate fails |

Example:
```properties
sonar.projectKey=my_project
sonar.projectName=My Project
sonar.sources=src
sonar.tests=test
sonar.sourceEncoding=UTF-8
sonar.exclusions=**/vendor/**,**/*.min.js
sonar.qualitygate.wait=true
```

## Coverage & language reports
Coverage isn't computed by Sonar — you feed it a report from your test tooling.
| Language | Parameter |
|---|---|
| JS/TS | `sonar.javascript.lcov.reportPaths=coverage/lcov.info` |
| Java (JaCoCo) | `sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml` |
| Python | `sonar.python.coverage.reportPaths=coverage.xml` |
| .NET | `sonar.cs.opencover.reportsPaths=coverage.opencover.xml` |
| Go | `sonar.go.coverage.reportPaths=coverage.out` |

## Branch & pull-request analysis
| Parameter | Use |
|---|---|
| `sonar.branch.name` | Analyse a long/short-lived branch |
| `sonar.newCode.referenceBranch` | Reference branch for "new code" |
| `sonar.pullrequest.key` | PR/MR number |
| `sonar.pullrequest.branch` | Source branch of the PR |
| `sonar.pullrequest.base` | Target branch of the PR |

> Branch/PR analysis needs Developer edition or above on Server (it's built-in on SonarQube Cloud).

## Quality Gates & Profiles
- **Quality Profile** — the set of active rules applied per language (e.g. "Sonar way"). Customise/clone in **Quality Profiles**.
- **Quality Gate** — pass/fail conditions on the analysis (e.g. *0 new bugs*, *coverage on new code ≥ 80%*, *no new security hotspots*). The default is **"Sonar way"**, focused on **new code** (the *Clean as You Code* model).
- Fail CI on a failing gate with `-Dsonar.qualitygate.wait=true`.

## SonarQube Cloud specifics
Hosted at `https://sonarcloud.io`; add `sonar.organization` and point the host URL at Cloud:
```bash
sonar-scanner \
  -Dsonar.organization=my-org \
  -Dsonar.projectKey=my-org_my_project \
  -Dsonar.host.url=https://sonarcloud.io \
  -Dsonar.token=$SONAR_TOKEN
```
Cloud can run **Automatic Analysis** (no CI needed) or **CI-based analysis** with a scanner. Token prefix for Cloud is typically `sqc_`.

## SonarQube for IDE (SonarLint)
IDE extension that flags issues as you type. Use **Connected Mode** to bind the IDE to a Server/Cloud project so it shares the same Quality Profile and suppresses issues consistently. Install from the IDE marketplace (VS Code, IntelliJ, Eclipse, Visual Studio).

## CI examples
**GitHub Actions:**
```yaml
- uses: actions/checkout@v4
  with: { fetch-depth: 0 }   # full history for accurate blame/new-code
- name: SonarQube Scan
  uses: sonarsource/sonarqube-scan-action@v4
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```
**GitLab CI:**
```yaml
sonarqube-check:
  image: sonarsource/sonar-scanner-cli:latest
  variables:
    SONAR_TOKEN: "$SONAR_TOKEN"
    SONAR_HOST_URL: "$SONAR_HOST_URL"
  script:
    - sonar-scanner -Dsonar.qualitygate.wait=true
  only:
    - merge_requests
    - main
```

## Web API (automation)
Authenticate with the token as the basic-auth username (empty password):
```bash
# Quality Gate status for a project
curl -u "$SONAR_TOKEN:" \
  "$SONAR_HOST_URL/api/qualitygates/project_status?projectKey=my_project"

# List projects / trigger housekeeping / manage users, etc.
curl -u "$SONAR_TOKEN:" "$SONAR_HOST_URL/api/projects/search"
```

## Handy operations
| Task | How |
|---|---|
| Force-fail CI on gate | `-Dsonar.qualitygate.wait=true` |
| Verbose scanner logs | `-Dsonar.verbose=true` or `sonar-scanner -X` |
| Exclude generated code | `sonar.exclusions=**/generated/**` |
| Analyse only new code | default Quality Gate already scopes to new code |
| Reset admin password (Server) | update `users` table in the DB, or re-provision |
| Check server health | `GET $SONAR_HOST_URL/api/system/health` |
