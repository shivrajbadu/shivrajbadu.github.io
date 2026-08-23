---
layout: post
title: "CircleCI Local Execution: A Developer's Guide to Offline Testing"
date: 2026-08-23 16:45:00 +0545
categories: [DevOps, Testing]
tags: [circleci, ci-cd, local-testing, docker, rspec, jest, minitest, debugging, circleci-cli]
---

# CircleCI Local Execution: A Developer's Guide to Offline Testing

## Introduction

Picture this: You've just refactored a critical Rails controller, updated your test suite, and you're ready to push to GitHub. You commit, push, and then... wait. CircleCI spins up a container, installs dependencies, boots PostgreSQL, and finally runs your RSpec suite—only to fail on a syntax error you could have caught in 30 seconds locally. Or worse: CircleCI is experiencing an outage, and you're blocked from merging time-sensitive hotfixes.

What if you could run your entire CircleCI pipeline on your local machine *before* pushing? What if you could debug failed builds with the exact same Docker images, environment variables, and dependency versions that CircleCI uses in production—all while working offline?

The **CircleCI CLI** makes this possible. In this guide, we'll walk through installing the CircleCI CLI, executing jobs locally, running your full test suite (RSpec, Minitest, Jest), understanding limitations, and building a workflow that eliminates the "push-and-pray" development cycle.

## Why Run CircleCI Locally?

### The Cost of Cloud-Only Testing

Every developer has experienced the friction of cloud-only CI/CD:

**Time waste:**
- Push code → Wait 2-3 minutes for container provisioning
- Discover typo in `.circleci/config.yml` → Fix → Wait another 2-3 minutes
- Repeat 3-4 times per feature branch
- **Total wasted time: 15-30 minutes per day**

**Blocked workflows:**
- CircleCI outage or maintenance window
- Rate-limited builds on free tier (depleted CI credits)
- Network connectivity issues in remote locations
- Need to test before committing (breaking CI is embarrassing)

**Debugging difficulty:**
- Can't reproduce failures locally because environment differs
- No SSH access to debug ephemeral containers quickly
- Unclear if failure is code bug or config issue

### What Local Execution Solves

Running CircleCI locally gives you:

✅ **Instant feedback:** Test config changes in seconds, not minutes  
✅ **Offline capability:** Work on planes, trains, or during outages  
✅ **Exact environment parity:** Same Docker images CircleCI uses  
✅ **Pre-commit validation:** Catch issues before they hit CI  
✅ **Cost savings:** Fewer wasted build minutes on paid plans  
✅ **Faster iteration:** Debug and fix without polluting git history

## Prerequisites

Before installing the CircleCI CLI, ensure you have:

| Requirement | Why It's Needed | Check Command |
|-------------|-----------------|---------------|
| **Docker Desktop** | CircleCI CLI uses Docker to simulate cloud execution | `docker --version` |
| **Git** | Required for checking out code in jobs | `git --version` |
| **Active CircleCI account** | Needed for validating configs against your projects | Account at circleci.com |
| **Admin access** | Installing CLI tools requires system permissions | `sudo -v` |

**macOS users:** Ensure Docker Desktop is running before executing CircleCI commands.

## Installing CircleCI CLI

### macOS Installation (Homebrew)

The simplest installation method on macOS uses Homebrew:

```bash
# Update Homebrew to latest
brew update

# Install CircleCI CLI
brew install circleci

# Verify installation
circleci version
```

**Expected output:**
```
CircleCI CLI 0.1.30000+xxxxxx
```

### Linux Installation (Snap)

For Ubuntu and Debian-based systems:

```bash
# Install via snap
sudo snap install circleci

# Verify installation
circleci version
```

### Manual Installation (All Platforms)

If package managers aren't available, download the binary directly:

```bash
# Linux/macOS x86_64
curl -fLSs https://raw.githubusercontent.com/CircleCI-Public/circleci-cli/master/install.sh | bash

# Verify installation
circleci version
```

The installer places the binary in `/usr/local/bin/circleci` by default.

### Authenticating with CircleCI

To validate configurations against your actual CircleCI projects, authenticate the CLI:

```bash
# Setup authentication (opens browser)
circleci setup

# Alternative: Use personal API token
circleci setup --token YOUR_CIRCLECI_API_TOKEN
```

**To get your API token:**
1. Visit https://app.circleci.com/settings/user/tokens
2. Click "Create New Token"
3. Name it `Local CLI Access`
4. Copy the generated token
5. Use it in the `circleci setup --token` command

**Note:** Authentication is optional for basic local execution but required for:
- Validating configs against your organization's Orbs
- Using private Orbs
- Running `circleci follow` or other API-dependent commands

{% include inarticle-adsense.html %}

## Understanding CircleCI Local Execution Architecture

Before running jobs locally, it's important to understand how the CLI simulates the cloud environment.

### What Gets Executed Locally

```
┌─────────────────────────────────────────┐
│    Your Development Machine             │
│                                         │
│  ┌───────────────────────────────────┐ │
│  │   CircleCI CLI Process            │ │
│  │   - Parses .circleci/config.yml   │ │
│  │   - Validates syntax              │ │
│  │   - Orchestrates Docker           │ │
│  └───────────────┬───────────────────┘ │
│                  │                      │
│                  ▼                      │
│  ┌───────────────────────────────────┐ │
│  │   Docker Engine                   │ │
│  │   ┌─────────────────────────────┐ │ │
│  │   │  Primary Container          │ │ │
│  │   │  (e.g., cimg/ruby:3.2-node) │ │ │
│  │   │  - Runs job steps           │ │ │
│  │   │  - Executes RSpec, Jest     │ │ │
│  │   └─────────────────────────────┘ │ │
│  │   ┌─────────────────────────────┐ │ │
│  │   │  Service Container          │ │ │
│  │   │  (e.g., postgres:15)        │ │ │
│  │   └─────────────────────────────┘ │ │
│  │   ┌─────────────────────────────┐ │ │
│  │   │  Service Container          │ │ │
│  │   │  (e.g., redis:7)            │ │ │
│  │   └─────────────────────────────┘ │ │
│  └───────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

**Key points:**
- CircleCI CLI reads your `.circleci/config.yml`
- Each job runs inside a Docker container (the **executor**)
- Service containers (databases, Redis) run as separate linked containers
- Your project code is mounted into the primary container
- Network connectivity between containers mirrors CircleCI cloud behavior

### What Doesn't Work Locally

Some CircleCI features depend on cloud infrastructure and won't work locally:

| Feature | Works Locally? | Workaround |
|---------|----------------|------------|
| Basic job execution | ✅ Yes | N/A |
| Docker executors | ✅ Yes | Requires Docker Engine |
| Service containers (Postgres, Redis) | ✅ Yes | Defined in `docker:` section |
| `machine` executor | ❌ No | Cannot simulate full VM locally |
| Workflows | ❌ No | Run individual jobs only |
| Context secrets | ⚠️ Limited | Must pass as environment variables manually |
| Remote Docker (`setup_remote_docker`) | ❌ No | Use local Docker daemon instead |
| Orbs | ⚠️ Limited | Public Orbs work; private require authentication |
| `persist_to_workspace` / `attach_workspace` | ❌ No | Workspace persistence unavailable |

## Running Your First Local Job

Let's start with a simple Rails application `.circleci/config.yml`:

```yaml
version: 2.1

jobs:
  test:
    docker:
      - image: cimg/ruby:3.2-node
        environment:
          RAILS_ENV: test
          DATABASE_URL: postgresql://postgres@localhost:5432/app_test
      - image: cimg/postgres:15
        environment:
          POSTGRES_DB: app_test
          POSTGRES_HOST_AUTH_METHOD: trust
      - image: cimg/redis:7

    steps:
      - checkout
      - run:
          name: Install Dependencies
          command: |
            bundle config set path vendor/bundle
            bundle install
      - run:
          name: Setup Database
          command: bundle exec rails db:schema:load
      - run:
          name: Run RSpec
          command: bundle exec rspec

workflows:
  version: 2
  test_workflow:
    jobs:
      - test
```

### Basic Execution Command

To run the `test` job locally:

```bash
# Navigate to your project root (where .circleci/config.yml exists)
cd ~/projects/my-rails-app

# Execute the 'test' job
circleci local execute --job test
```

**What happens:**
1. CLI validates your config syntax
2. Pulls Docker images (`cimg/ruby:3.2-node`, `cimg/postgres:15`, `cimg/redis:7`)
3. Starts containers with proper networking
4. Mounts your project directory into the container
5. Executes each step sequentially
6. Outputs results to your terminal

**Expected output:**
```
====>> Spin up environment
Build-agent version  ()
Docker Engine Version: 24.0.6
Kernel Version: Linux 5.15.0
Starting container cimg/ruby:3.2-node
Starting container cimg/postgres:15
Starting container cimg/redis:7

====>> Checkout code
Cloning into '.'...

====>> Install Dependencies
Fetching gem metadata from https://rubygems.org/
Bundle complete! 45 Gemfile dependencies, 120 gems now installed.

====>> Setup Database
Created database 'app_test'

====>> Run RSpec
...................................
35 examples, 0 failures

Success!
```

### Running Specific Jobs

If your config has multiple jobs:

```yaml
jobs:
  lint:
    # ... rubocop linting
  
  test_rspec:
    # ... RSpec tests
  
  test_jest:
    # ... Jest tests
```

Run individual jobs:

```bash
# Run only linting
circleci local execute --job lint

# Run only RSpec
circleci local execute --job test_rspec

# Run only Jest
circleci local execute --job test_jest
```

## Testing Multiple Test Frameworks Locally

Your Rails application likely uses multiple testing frameworks. Here's how to execute each locally.

### Running RSpec Tests

**Config snippet:**
```yaml
- run:
    name: Run RSpec
    command: |
      mkdir -p tmp/test-results/rspec
      bundle exec rspec --format progress \
                        --format RspecJunitFormatter \
                        --out tmp/test-results/rspec/results.xml
```

**Local execution:**
```bash
circleci local execute --job test_rspec
```

**Verification:**
Check that all specs pass and results are saved to `tmp/test-results/rspec/`.

### Running Minitest Tests

**Config snippet:**
```yaml
- run:
    name: Run Minitest
    command: |
      bundle exec rails test
```

**Local execution:**
```bash
circleci local execute --job test_minitest
```

### Running Jest Tests

**Config snippet:**
```yaml
- run:
    name: Run Jest
    command: |
      yarn test --ci --coverage --maxWorkers=2
```

**Local execution:**
```bash
circleci local execute --job test_jest
```

**Note:** If Jest is defined in a separate job with a Node.js executor:

```yaml
jobs:
  test_frontend:
    docker:
      - image: cimg/node:20
    steps:
      - checkout
      - run: yarn install --frozen-lockfile
      - run: yarn test --ci
```

Run it with:
```bash
circleci local execute --job test_frontend
```

### Running RSpec Migrations (rspec-rails-migrations)

If you use gems like `immigrant` or custom migration specs:

**Config snippet:**
```yaml
- run:
    name: Check Migration Specs
    command: bundle exec rspec spec/migrations
```

**Local execution:**
```bash
circleci local execute --job test
```

The job will execute all steps including migration-specific specs.

## Advanced Configuration Techniques

### Passing Environment Variables

CircleCI Context variables and project secrets aren't available locally. Pass them manually:

```bash
# Single variable
circleci local execute --job test \
  --env AWS_ACCESS_KEY_ID=your_key_here

# Multiple variables
circleci local execute --job test \
  --env AWS_ACCESS_KEY_ID=your_key \
  --env AWS_SECRET_ACCESS_KEY=your_secret \
  --env DATABASE_URL=postgresql://localhost/test_db
```

**Better approach:** Create a `.env.local` file (add to `.gitignore`):

```bash
# .env.local
AWS_ACCESS_KEY_ID=your_key
AWS_SECRET_ACCESS_KEY=your_secret
STRIPE_API_KEY=sk_test_xxxxx
```

Then load it before execution:

```bash
# Load variables and run job
set -a; source .env.local; set +a
circleci local execute --job test \
  --env AWS_ACCESS_KEY_ID \
  --env AWS_SECRET_ACCESS_KEY \
  --env STRIPE_API_KEY
```

### Overriding the Checkout Step

By default, `checkout` clones your repository. When running locally, CircleCI mounts your current directory instead, preserving uncommitted changes.

**This means:**
- ✅ You can test code changes before committing
- ✅ Works with dirty working directory
- ⚠️ Make sure your code is in the expected state

### Using Build Parameters

If your config accepts pipeline parameters:

```yaml
parameters:
  run_integration_tests:
    type: boolean
    default: false

jobs:
  test:
    steps:
      - when:
          condition: << pipeline.parameters.run_integration_tests >>
          steps:
            - run: bundle exec rspec spec/integration
```

**Local execution doesn't support pipeline parameters directly.** Workaround: Temporarily modify your config or use separate jobs.

## Validating Configuration Without Execution

Before running expensive test suites, validate syntax:

```bash
# Validate config file
circleci config validate .circleci/config.yml
```

**Output if valid:**
```
Config file at .circleci/config.yml is valid.
```

**Output if invalid:**
```
Error: ERROR IN CONFIG FILE:
[#/jobs/test/steps/0] expected type: Mapping, found: String
```

### Processing Orbs and Dynamic Config

If your config uses Orbs or dynamic generation, process it first:

```bash
# Expand Orbs into raw YAML
circleci config process .circleci/config.yml > processed_config.yml

# Review the processed config
cat processed_config.yml

# Execute against processed version
circleci local execute --config processed_config.yml --job test
```

This resolves Orb commands into their underlying implementation, helpful for debugging.

## Common Pitfalls and Solutions

### Pitfall 1: Docker Images Not Pulling

**Problem:** `image not found` or `pull access denied`

**Solution:**
```bash
# Manually pull images first
docker pull cimg/ruby:3.2-node
docker pull cimg/postgres:15
docker pull cimg/redis:7

# Then run job
circleci local execute --job test
```

### Pitfall 2: Port Conflicts

**Problem:** `port is already allocated` when Postgres or Redis containers start

**Solution:** Stop conflicting local services:

```bash
# Stop local PostgreSQL
brew services stop postgresql

# Stop local Redis
brew services stop redis

# Or change ports in your config (not recommended)
```

### Pitfall 3: Missing Dependencies

**Problem:** Job fails because dependencies aren't cached

**Cause:** CircleCI's `save_cache` / `restore_cache` don't work locally

**Solution:** Run bundle install locally first:

```bash
# Install gems locally
bundle install --path vendor/bundle

# Then run job (gems will be reused)
circleci local execute --job test
```

### Pitfall 4: Workspace Persistence Failures

**Problem:** `attach_workspace` step fails

**Cause:** Workspace persistence is cloud-only

**Solution:** Restructure jobs to be self-contained for local testing, or skip workspace-dependent steps:

```yaml
- run:
    name: Skip workspace check locally
    command: |
      if [ -z "$CIRCLECI" ]; then
        echo "Running locally, skipping workspace"
      else
        # workspace logic here
      fi
```

### Pitfall 5: Machine Executor Unsupported

**Problem:** `machine executor is not supported for local builds`

**Cause:** CircleCI CLI can't simulate full VMs

**Solution:** Use Docker executor for local testing:

```yaml
# Instead of:
# executor: machine

# Use:
executor:
  name: docker
  - image: cimg/ruby:3.2-node
```

## Building a Pre-Commit Workflow

Integrate local CircleCI testing into your development workflow for maximum efficiency.

### Git Pre-Commit Hook

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash
# .git/hooks/pre-commit

echo "🔍 Running CircleCI validation..."

# Validate config syntax
if ! circleci config validate .circleci/config.yml; then
  echo "❌ CircleCI config is invalid. Fix errors before committing."
  exit 1
fi

echo "✅ CircleCI config is valid"

# Optionally run quick linting job
# Uncomment if you have a fast lint job:
# circleci local execute --job lint

exit 0
```

Make it executable:

```bash
chmod +x .git/hooks/pre-commit
```

### Makefile for Common Tasks

Create a `Makefile` in your project root:

```makefile
.PHONY: ci-validate ci-test-local ci-lint ci-rspec ci-jest

ci-validate:
	@echo "Validating CircleCI config..."
	@circleci config validate .circleci/config.yml

ci-lint:
	@echo "Running lint job locally..."
	@circleci local execute --job lint

ci-rspec:
	@echo "Running RSpec locally..."
	@circleci local execute --job test_rspec

ci-jest:
	@echo "Running Jest locally..."
	@circleci local execute --job test_jest

ci-test-local: ci-validate ci-lint ci-rspec ci-jest
	@echo "✅ All local CI checks passed!"
```

**Usage:**

```bash
# Validate config only
make ci-validate

# Run all tests locally
make ci-test-local

# Run specific test suite
make ci-rspec
```

### Shell Alias for Quick Testing

Add to your `~/.zshrc` or `~/.bashrc`:

```bash
# Quick CircleCI local execution
alias ci-local='circleci local execute --job test'
alias ci-validate='circleci config validate .circleci/config.yml'
alias ci-lint='circleci local execute --job lint'
```

Reload shell:
```bash
source ~/.zshrc
```

Now run tests with:
```bash
ci-local
```

## When to Use Local Execution vs Cloud

Local execution is powerful but not always the right choice.

### Use Local Execution When:

✅ **Iterating on config changes** — Catch YAML syntax errors instantly  
✅ **Debugging test failures** — Reproduce exact CI environment  
✅ **Working offline** — Planes, remote locations, CircleCI outages  
✅ **Pre-commit validation** — Ensure changes won't break CI  
✅ **Learning CircleCI** — Experiment without burning build minutes  
✅ **Rapid feedback** — Test small changes without push/wait cycle

### Use Cloud Execution When:

☁️ **Testing workflows** — Multi-job orchestration with dependencies  
☁️ **Parallel test splitting** — Using `parallelism: 4` or knapsack_pro  
☁️ **Machine executors** — Full VM required for Docker-in-Docker  
☁️ **Complex Contexts** — Many encrypted secrets from CircleCI Contexts  
☁️ **Resource-intensive builds** — Asset compilation, large test suites  
☁️ **Final validation** — Always run in cloud before merging to main

**Best practice:** Use local execution during development, cloud execution for final validation.

## Performance Comparison

Here's a real-world benchmark for a medium-sized Rails app:

| Test Suite | Local Execution | Cloud Execution | Time Saved |
|------------|-----------------|-----------------|------------|
| Config validation | 2 seconds | 45 seconds* | 95% |
| Linting (RuboCop) | 12 seconds | 90 seconds* | 87% |
| RSpec (100 specs) | 45 seconds | 2m 30s* | 70% |
| Jest (50 tests) | 18 seconds | 1m 45s* | 83% |
| Full test suite | 1m 15s | 4m 50s* | 74% |

*Includes container provisioning, dependency installation, and queue time

**Key insight:** Local execution eliminates the **cold start penalty** of cloud CI, making it 70-95% faster for iterative development.

## Troubleshooting Guide

### Error: "Cannot connect to Docker daemon"

**Cause:** Docker Desktop not running

**Fix:**
```bash
# macOS: Start Docker Desktop from Applications
open -a Docker

# Linux: Start Docker service
sudo systemctl start docker
```

### Error: "Job does not exist in config"

**Cause:** Job name doesn't match config file

**Fix:**
```bash
# List available jobs
circleci config validate .circleci/config.yml
# Look for jobs: section, note exact names

# Use correct job name (case-sensitive)
circleci local execute --job test_rspec
```

### Error: "Volume mount requires absolute path"

**Cause:** Running command from wrong directory

**Fix:**
```bash
# Ensure you're in project root
cd ~/projects/my-rails-app
pwd  # Should show project root
ls .circleci/config.yml  # Should exist

# Then run
circleci local execute --job test
```

### Tests Pass Locally But Fail in Cloud

**Common causes:**
1. **Environment differences:** Cloud has different environment variables
2. **Timing issues:** Cloud containers slower, causing timeouts
3. **Dependency versions:** Local cache vs fresh install
4. **Database state:** Local DB not clean between runs

**Fix:**
```bash
# Clean database before local test
bundle exec rails db:drop db:create db:schema:load RAILS_ENV=test

# Run with clean bundle install
rm -rf vendor/bundle
circleci local execute --job test
```

## Conclusion

Running CircleCI locally transforms your development workflow from reactive to proactive. Instead of discovering config errors or test failures after pushing to GitHub, you catch them instantly on your machine. Instead of waiting 5 minutes for cloud builds during rapid iteration, you get feedback in seconds. And when CircleCI experiences downtime, you stay productive.

The CircleCI CLI bridges the gap between local development and cloud CI/CD, giving you the best of both worlds: the speed and control of local testing with the consistency and scalability of cloud infrastructure. By integrating local execution into your pre-commit hooks, Makefiles, and daily workflow, you build confidence in your changes before they ever touch the remote repository.

Start with simple validation (`circleci config validate`), progress to running individual jobs (`circleci local execute --job test`), and eventually build sophisticated pre-commit workflows that catch issues automatically. Your future self—and your team—will thank you for the time saved and bugs prevented.

Test locally. Push confidently. Deploy safely.

## Suggested Reading

- [CircleCI CLI Documentation](https://circleci.com/docs/local-cli/)
- [CircleCI Configuration Reference](https://circleci.com/docs/configuration-reference/)
- [Docker Documentation: Get Started](https://docs.docker.com/get-started/)
- [CircleCI Orbs Registry](https://circleci.com/developer/orbs)
- [Testing Strategies for Rails Applications](https://guides.rubyonrails.org/testing.html)
- [Git Hooks Documentation](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)

{% include inarticle-adsense.html %}
