# GitHub Actions Complete Guide for Beginners

This repository contains comprehensive examples and documentation for learning GitHub Actions from the ground up. All examples are designed to be educational and demonstrate real-world patterns.

## 📁 Repository Structure

```
.github/
├── workflows/
│   ├── demo.yml                      # Main demo workflow showing all concepts
│   └── matrix-strategy-examples.yml  # Matrix strategy patterns and examples
└── actions/
    ├── publish-artifact/             # Custom TypeScript action for publishing
    │   ├── action.yml
    │   ├── package.json
    │   ├── tsconfig.json
    │   ├── src/index.ts
    │   └── dist/index.js
    └── deploy/                       # Custom TypeScript action for deployment
        ├── action.yml
        ├── package.json
        ├── tsconfig.json
        ├── src/index.ts
        └── dist/index.js
```

## 🎯 What You'll Learn

### 1. **Event Triggers** 
Learn when workflows run and how to control execution.

### 2. **Workflow Inputs**
Handle user inputs and parameterize workflows.

### 3. **Jobs and Steps**
Organize work into logical units and sequential operations.

### 4. **Environment Variables**
Manage configuration and sensitive data.

### 5. **Secrets and Variables**
Secure handling of credentials and environment-specific values.

### 6. **Matrix Strategy**
Run jobs across multiple configurations efficiently.

### 7. **Custom Actions**
Create reusable TypeScript actions for common tasks.

### 8. **Artifacts and Outputs**
Share data between jobs and workflow runs.

---

## 🚀 Core Concepts Explained

### Event Triggers

Event triggers determine when your workflow runs. Our demo workflow shows several trigger types:

#### **Push Trigger**
```yaml
on:
  push:
    branches: [ main, develop ]
    paths-ignore:
      - '*.md'
      - 'docs/**'
```
- Runs when code is pushed to specified branches
- Can filter by paths to avoid unnecessary runs
- Supports `paths` and `paths-ignore` for fine-grained control

#### **Pull Request Trigger**
```yaml
on:
  pull_request:
    branches: [ main ]
```
- Runs when PRs are opened, synchronized, or reopened
- Useful for validation before merging
- Can target specific branches

#### **Manual Trigger (workflow_dispatch)**
```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
```
- Allows manual workflow execution from GitHub UI
- Supports various input types: `string`, `boolean`, `choice`, `environment`
- Enables interactive workflows

#### **Schedule Trigger**
```yaml
on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC
```
- Runs workflows on a schedule using cron syntax
- Useful for maintenance tasks, backups, or regular checks
- Always runs on the default branch

### Environment Variables

Environment variables provide configuration and context to your workflows:

#### **Global Variables**
```yaml
env:
  NODE_VERSION: '18'
  REGISTRY_URL: 'https://npm.pkg.github.com'
```

#### **Job-Level Variables**
```yaml
jobs:
  build:
    env:
      BUILD_ENV: 'production'
```

#### **Step-Level Variables**
```yaml
steps:
  - name: Deploy
    env:
      API_KEY: ${{ secrets.API_KEY }}
```

#### **Built-in Variables**
- `${{ github.actor }}` - User who triggered the workflow
- `${{ github.sha }}` - Commit SHA
- `${{ github.ref }}` - Branch or tag reference
- `${{ github.workspace }}` - Workspace directory path
- `${{ runner.os }}` - Operating system (Linux, Windows, macOS)

### Secrets and Variables

#### **Repository Secrets**
Store sensitive data like API keys, tokens, and passwords:

```yaml
steps:
  - name: Deploy to production
    env:
      API_KEY: ${{ secrets.PRODUCTION_API_KEY }}
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

**Setting Secrets:**
1. Go to repository Settings > Secrets and variables > Actions
2. Click "New repository secret"
3. Add name and value
4. Use in workflows with `${{ secrets.SECRET_NAME }}`

#### **Repository Variables**
Store non-sensitive configuration:

```yaml
steps:
  - name: Configure app
    env:
      APP_NAME: ${{ vars.APPLICATION_NAME }}
      REGION: ${{ vars.DEPLOYMENT_REGION }}
```

#### **Environment Secrets and Variables**
Environment-specific values with protection rules:

```yaml
jobs:
  deploy:
    environment: production  # Uses production environment secrets/variables
    steps:
      - name: Deploy
        env:
          API_ENDPOINT: ${{ vars.API_ENDPOINT }}  # Environment variable
          SECRET_KEY: ${{ secrets.SECRET_KEY }}   # Environment secret
```

### Jobs and Dependencies

Jobs define units of work that can run in parallel or sequence:

#### **Parallel Jobs**
```yaml
jobs:
  build:
    # Runs independently
  test:
    # Also runs independently
```

#### **Sequential Jobs**
```yaml
jobs:
  build:
    # Runs first
  
  test:
    needs: build  # Waits for build to complete
  
  deploy:
    needs: [build, test]  # Waits for both build and test
```

#### **Conditional Jobs**
```yaml
jobs:
  deploy:
    if: ${{ github.ref == 'refs/heads/main' }}
    needs: test
```

### Job Outputs

Share data between jobs using outputs:

#### **Setting Outputs**
```yaml
jobs:
  build:
    outputs:
      version: ${{ steps.get-version.outputs.version }}
      artifact-url: ${{ steps.upload.outputs.url }}
    
    steps:
      - id: get-version
        run: echo "version=1.2.3" >> $GITHUB_OUTPUT
```

#### **Using Outputs**
```yaml
jobs:
  deploy:
    needs: build
    steps:
      - name: Deploy version
        run: echo "Deploying ${{ needs.build.outputs.version }}"
```

### Artifacts

Share files between jobs and workflow runs:

#### **Upload Artifacts**
```yaml
steps:
  - name: Upload build artifacts
    uses: actions/upload-artifact@v4
    with:
      name: build-files
      path: |
        dist/
        package.json
      retention-days: 30
```

#### **Download Artifacts**
```yaml
steps:
  - name: Download build artifacts
    uses: actions/download-artifact@v4
    with:
      name: build-files
      path: ./downloaded
```

---

## 🔄 Matrix Strategy Deep Dive

Matrix strategy lets you run the same job with different configurations, creating a matrix of combinations.

### Basic Matrix

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node-version: [16, 18, 20]
    # Creates 9 jobs: 3 OS × 3 Node versions
```

### Advanced Matrix with Include/Exclude

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node-version: [16, 18, 20]
    
    # Remove specific combinations
    exclude:
      - os: windows-latest
        node-version: 16
    
    # Add specific combinations
    include:
      - os: ubuntu-latest
        node-version: 21
        experimental: true
```

### Matrix Options

```yaml
strategy:
  matrix:
    # ... your matrix
  fail-fast: false     # Continue other jobs if one fails
  max-parallel: 4      # Limit concurrent jobs
```

### Dynamic Matrix

Create matrix from JSON:

```yaml
jobs:
  generate-matrix:
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - id: set-matrix
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            matrix='{"os":["ubuntu-latest","windows-latest"],"node":[16,18,20]}'
          else
            matrix='{"os":["ubuntu-latest"],"node":[18]}'
          fi
          echo "matrix=$matrix" >> $GITHUB_OUTPUT

  use-matrix:
    needs: generate-matrix
    strategy:
      matrix: ${{ fromJson(needs.generate-matrix.outputs.matrix) }}
```

---

## 🛠 Custom Actions Development

### TypeScript Action Structure

Our repository includes two custom TypeScript actions demonstrating professional patterns:

#### **Action Metadata (action.yml)**
```yaml
name: 'My Custom Action'
description: 'Description of what the action does'
inputs:
  input-name:
    description: 'Input description'
    required: true
    default: 'default-value'
outputs:
  output-name:
    description: 'Output description'
runs:
  using: 'node20'
  main: 'dist/index.js'
```

#### **TypeScript Implementation**
```typescript
import * as core from '@actions/core';
import * as github from '@actions/github';

async function run(): Promise<void> {
  try {
    // Get inputs
    const inputValue = core.getInput('input-name', { required: true });
    
    // Perform action logic
    const result = await performWork(inputValue);
    
    // Set outputs
    core.setOutput('output-name', result);
    
    core.info('Action completed successfully!');
  } catch (error) {
    core.setFailed(`Action failed: ${error.message}`);
  }
}

run();
```

#### **Building Actions**
```json
{
  "scripts": {
    "build": "ncc build src/index.ts -o dist",
    "package": "npm run build"
  },
  "dependencies": {
    "@actions/core": "^1.10.1",
    "@actions/github": "^6.0.0"
  }
}
```

### Action Best Practices

1. **Error Handling**: Always wrap main logic in try-catch
2. **Input Validation**: Validate all inputs early
3. **Informative Logging**: Use `core.info()`, `core.warning()`, `core.error()`
4. **Outputs**: Always set meaningful outputs
5. **Documentation**: Include clear descriptions in action.yml
6. **Testing**: Test actions thoroughly before publishing

---

## 🏗 Complete CI/CD Pipeline Example

Our demo workflow demonstrates a complete Node.js CI/CD pipeline:

### Pipeline Stages

1. **Setup & Validation**
   - Extract version information
   - Check deployment conditions
   - Generate cache keys

2. **Environment Preparation (Matrix)**
   - Test across multiple OS and Node versions
   - Install dependencies
   - Validate environment

3. **Build**
   - Compile application
   - Generate build artifacts
   - Create build metadata

4. **Testing**
   - Unit tests
   - Integration tests (conditional)
   - Generate test reports

5. **Static Analysis**
   - Linting
   - Security scanning
   - Code quality checks

6. **Packaging**
   - Use custom publish action
   - Create deployment packages
   - Tag versions

7. **Deployment**
   - Deploy to staging (conditional)
   - Deploy to production (with approval)
   - Health checks
   - Notifications

8. **Cleanup**
   - Clean temporary resources
   - Send final notifications

### Pipeline Features

- **Conditional Execution**: Steps run based on branch, inputs, or previous results
- **Parallel Processing**: Independent jobs run simultaneously
- **Artifact Sharing**: Build outputs flow through the pipeline
- **Environment Protection**: Production requires approval
- **Rollback Capability**: Automatic rollback on deployment failure
- **Monitoring**: Health checks and notifications

---

## 📚 Learning Path

### Beginner (Start Here)
1. Review `demo.yml` workflow structure
2. Understand event triggers and basic jobs
3. Learn about environment variables and secrets
4. Practice with simple workflows

### Intermediate
1. Explore matrix strategy examples
2. Create custom composite actions
3. Implement artifact sharing between jobs
4. Set up environment protection rules

### Advanced
1. Build TypeScript actions (see our examples)
2. Implement complex conditional logic
3. Create reusable workflow templates
4. Set up advanced monitoring and notifications

---

## 🔧 Setup Instructions

### Prerequisites
- GitHub repository with Actions enabled
- Basic understanding of YAML syntax
- Node.js knowledge (for custom actions)

### Getting Started

1. **Clone/Fork this repository**
2. **Set up secrets** (in repository settings):
   ```
   STAGING_API_KEY=your-staging-key
   PRODUCTION_API_KEY=your-production-key
   STAGING_DEPLOYMENT_TOKEN=your-staging-token
   PRODUCTION_DEPLOYMENT_TOKEN=your-production-token
   ```

3. **Configure environments** (repository settings > Environments):
   - Create `staging` environment
   - Create `production` environment with protection rules

4. **Test workflows**:
   - Push code to trigger automatic workflows
   - Use "Actions" tab to manually trigger `workflow_dispatch`
   - Experiment with different input combinations

### Custom Actions Setup

To build the TypeScript actions:

```bash
# For publish-artifact action
cd .github/actions/publish-artifact
npm install
npm run build

# For deploy action
cd .github/actions/deploy
npm install
npm run build
```

---

## 🎓 Advanced Patterns

### Reusable Workflows

Create workflow templates:

```yaml
# .github/workflows/reusable-deploy.yml
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
    secrets:
      api-key:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to ${{ inputs.environment }}
        env:
          API_KEY: ${{ secrets.api-key }}
        run: echo "Deploying to ${{ inputs.environment }}"
```

Use reusable workflow:

```yaml
# .github/workflows/main.yml
jobs:
  deploy-staging:
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: staging
    secrets:
      api-key: ${{ secrets.STAGING_API_KEY }}
```

### Composite Actions

Create composite actions for repeated step sequences:

```yaml
# .github/actions/setup-node/action.yml
name: 'Setup Node.js with Cache'
description: 'Setup Node.js with dependency caching'
inputs:
  node-version:
    description: 'Node.js version'
    required: true
runs:
  using: 'composite'
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'
    - run: npm ci
      shell: bash
```

### Advanced Conditionals

Complex conditional logic:

```yaml
jobs:
  deploy:
    if: |
      github.event_name == 'push' && 
      github.ref == 'refs/heads/main' && 
      !contains(github.event.head_commit.message, '[skip deploy]')
```

---

## 🚨 Best Practices & Security

### Security Guidelines

1. **Never log secrets**: Use `::add-mask::` for sensitive outputs
2. **Minimal permissions**: Use `permissions:` to limit token scope
3. **Pin action versions**: Use specific versions like `@v4.1.0`
4. **Validate inputs**: Always validate external inputs
5. **Use environments**: Protect production with approval workflows

### Performance Optimization

1. **Use caching**: Cache dependencies and build artifacts
2. **Parallel execution**: Run independent jobs in parallel
3. **Conditional steps**: Skip unnecessary work with `if:` conditions
4. **Matrix optimization**: Use `fail-fast: false` appropriately
5. **Artifact cleanup**: Set appropriate retention periods

### Debugging Tips

1. **Enable debug logging**: Set `ACTIONS_STEP_DEBUG=true` secret
2. **Use workflow commands**: `::notice::`, `::warning::`, `::error::`
3. **SSH debugging**: Use `tmate` action for interactive debugging
4. **Local testing**: Use `act` to run workflows locally

---

## 📖 Additional Resources

### Official Documentation
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Creating Actions](https://docs.github.com/en/actions/creating-actions)

### Community Resources
- [Awesome Actions](https://github.com/sdras/awesome-actions)
- [GitHub Actions Toolkit](https://github.com/actions/toolkit)
- [Actions Marketplace](https://github.com/marketplace?type=actions)

### Tools
- [act](https://github.com/nektos/act) - Run GitHub Actions locally
- [@vercel/ncc](https://github.com/vercel/ncc) - Compile Node.js projects
- [GitHub CLI](https://cli.github.com/) - Manage workflows from command line

---

## 🤝 Contributing

This repository is designed for learning. Feel free to:

1. **Fork and experiment** with the workflows
2. **Submit improvements** to documentation or examples  
3. **Report issues** if you find problems
4. **Share feedback** on the learning experience

---

## 📝 License

This project is licensed under the MIT License - see the workflow files and actions for educational use.

---

**Happy Learning! 🎉**

Start with the basic `demo.yml` workflow, then explore matrix strategies, and finally dive into custom TypeScript actions. Each component builds upon previous concepts to give you a complete understanding of GitHub Actions capabilities.
