# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Repository Overview

The **CNF Certification Suite Operator Console Plugin** is an [OpenShift Dynamic Console Plugin](https://github.com/openshift/console/tree/master/frontend/packages/console-dynamic-plugin-sdk) that extends the OpenShift web console UI for the CNF Certification Suite Operator.

This plugin enables users to:
- Run CNF certification test suites directly from the OpenShift console
- View and filter certification test results (passed, failed, skipped, errored)
- Manage CnfCertificationSuiteRun custom resources
- Access related Secrets and ConfigMaps for test configuration

The plugin integrates with the [CNF Certification Suite Operator](https://github.com/test-network-function/cnf-certsuite-operator) which must be installed on the cluster.

## Build Commands

### Using Makefile

```bash
make docker-build       # Build Docker image (default: cnf-certsuite-plugin)
make docker-push        # Push Docker image to registry
make lint               # Run all linters (hadolint, shfmt, typos, markdownlint, yamllint)
```

You can override the image name:
```bash
make docker-build IMG=my-registry/my-plugin:tag
```

### Using Yarn/NPM

```bash
yarn install            # Install dependencies
yarn build              # Production build (clean + webpack)
yarn build-dev          # Development build
yarn clean              # Remove dist directory
yarn start              # Start webpack dev server on port 9001
yarn start-console      # Start local OpenShift console with plugin
```

## Test Commands

### Linting

```bash
yarn lint               # Run ESLint and Stylelint with auto-fix
```

### Integration Tests (Cypress)

```bash
yarn test-cypress           # Open Cypress test runner (interactive)
yarn test-cypress-headless  # Run Cypress tests headlessly
yarn cypress-postreport     # Merge and generate test reports
```

### Shell Script Testing

```bash
./test-frontend.sh          # Frontend test script
./test-prow-e2e.sh          # Prow E2E test script
```

## Code Organization

```
src/
  components/
    plugin.ts              # Plugin entry point, exports components
    CnfCertsuiteRunPage.tsx # List page for CnfCertificationSuiteRun CRs
    ResultsPage.tsx        # Results display with filtering
    ProgressBar.tsx        # Visual summary of test results
    ConfigMapList.tsx      # ConfigMap list component
    SecretList.tsx         # Secret list component
    example.css            # Component styles

integration-tests/
  tests/                   # Cypress test specs (*.cy.ts)
  plugins/                 # Cypress plugins
  support/                 # Test support files and login utilities
  fixtures/                # Test fixtures

locales/
  en/                      # English translations

i18n-scripts/              # Internationalization build scripts
.devcontainer/             # VS Code dev container configuration
```

## Key Dependencies

### OpenShift Console SDK
- `@openshift-console/dynamic-plugin-sdk` (1.2.0): Core SDK for building console plugins
- `@openshift-console/dynamic-plugin-sdk-webpack`: Webpack plugin for console plugins

### UI Framework
- `@patternfly/react-core` (5.1.1): PatternFly React components
- `@patternfly/react-icons` (5.1.1): PatternFly icons
- `react` (17.0.1): React library
- `react-router-dom` (5.3.x): Routing

### Build Tools
- `webpack` (5.75.0): Module bundler
- `typescript` (4.7.4): TypeScript compiler
- `ts-loader`: TypeScript loader for webpack

### Testing
- `cypress` (13.11.0): E2E testing framework
- `eslint`: Code linting
- `stylelint`: CSS linting

## Development Guidelines

### Console Extensions

The plugin registers extensions in `console-extensions.json`:
- `console.navigation/section`: Creates "Cnf Certification Suites" navigation section
- `console.navigation/resource-ns`: Adds navigation items for CRs, Secrets, ConfigMaps
- `console.page/resource/list`: Custom list page for CnfCertificationSuiteRun
- `console.tab/horizontalNav`: Results tab on CR detail page

### Kubernetes Resources

The plugin works with these resources:
- **CnfCertificationSuiteRun** (`cnf-certifications.redhat.com/v1alpha1`): Main CR for test runs
- **Secret**: Configuration secrets for test runs
- **ConfigMap**: Configuration data for test runs

### Local Development

1. Ensure you're logged into an OpenShift cluster:
   ```bash
   oc login <cluster-url>
   ```

2. Start the webpack dev server:
   ```bash
   yarn start
   ```

3. In another terminal, start the local console:
   ```bash
   yarn start-console
   ```

4. Access the console at http://localhost:9000

### Dev Container

The repository includes VS Code dev container configuration:
- Requires `OC_URL`, `OC_USER`, `OC_PASS` environment variables
- Forwards ports 9000 (console) and 9001 (plugin)
- Includes Docker, TypeScript, ESLint, and Prettier extensions

### Linting Requirements

Code must pass:
- `hadolint`: Dockerfile linting
- `shfmt`: Shell script formatting
- `typos`: Spell checking
- `markdownlint`: Markdown formatting
- `yamllint`: YAML validation
- `eslint`: TypeScript/React linting
- `stylelint`: CSS linting

### Internationalization

The plugin uses react-i18next for translations:
- Translation keys are prefixed with `plugin__cnf-certsuite-plugin~`
- Translations are in `locales/en/plugin__cnf-certsuite-plugin.json`
- Run `yarn i18n` to extract and update translation files

## Container Image

The Dockerfile uses a multi-stage build:
1. **Build stage**: Uses `ubi8/nodejs-20` to run yarn install and build
2. **Runtime stage**: Uses `ubi8/nginx-120` to serve static files

The built plugin assets are served via nginx from `/usr/share/nginx/html`.

## Enabling the Plugin

After installing the CNF Certification Suite Operator:

**Via OLM Subscription**: Plugin is automatically enabled. If not, navigate to Operators > Installed Operators > cnf-certsuite-operator and enable the Console plugin.

**Manual Deployment**: Navigate to Administration > Cluster Settings > Configuration > Console operator.openshift.io > Console plugins and enable `cnf-certsuite-plugin`.
