# GitHub Actions Workflows

This directory contains GitHub Actions workflows for the Aether OnRamp project.

## Quickstart Workflow

The `quickstart.yml` workflow replicates the functionality of the Jenkins groovy workflow found at [opennetworkinglab/aether-jenkins](https://github.com/opennetworkinglab/aether-jenkins/blob/master/quickstart.groovy).

### Key Features

- **Automated Environment Setup**: Installs kubectl, helm, and Python dependencies at runtime
- **Dynamic Configuration**: Automatically detects IP address and network interface
- **Complete Test Cycle**: Configures, installs, tests, validates, and cleans up Aether OnRamp
- **Artifact Collection**: Saves logs from all components for debugging

### Required Secrets

The workflow requires the following GitHub secret to be configured in your repository:

- `AETHER_QA_PEM`: SSH private key for authenticating to the target nodes
  - This replaces the PEM file that was previously stored on the Jenkins builder at `/home/ubuntu/aether-qa.pem`
  - To add this secret:
    1. Go to your repository's Settings → Secrets and variables → Actions
    2. Click "New repository secret"
    3. Name: `AETHER_QA_PEM`
    4. Value: Paste the entire contents of your SSH private key

### Workflow Stages

1. **Checkout repository**: Clones the repo with all submodules
2. **Set up Python**: Installs Python and creates a virtual environment with Ansible
3. **Install kubectl**: Uses Azure's setup-kubectl action (default/latest version)
4. **Install Helm**: Uses Azure's setup-helm action (default/latest version)
5. **Configure OnRamp**: Sets up SSH keys, generates hosts.ini, configures vars/main.yml
6. **Install Aether**: Installs Kubernetes, 5GC core, and gNBsim
7. **Run gNBsim**: Executes the gNBsim test
8. **Validate Results**: Checks that tests passed
9. **Retrieve Logs**: Collects logs from all components
10. **Archive Artifacts**: Uploads logs as workflow artifacts
11. **Cleanup**: Uninstalls all components (always runs)
12. **Notify on Failure**: Logs failure information (can be extended with Slack notifications)

### Triggering the Workflow

The workflow can be triggered in three ways:

1. **Manual Dispatch**: Go to Actions → Aether OnRamp Quickstart → Run workflow
2. **Push to main**: Automatically runs on pushes to the main branch
3. **Pull Request**: Automatically runs on PRs to the main branch

### Differences from Jenkins Workflow

The GitHub Actions workflow includes the following changes from the original Jenkins groovy script:

1. **Runtime Installation**: kubectl, helm, and Python virtualenv are installed at runtime instead of relying on pre-installed tools
2. **Secret Management**: SSH private key is stored as a GitHub secret (`AETHER_QA_PEM`) instead of a file on the builder
3. **Self-Contained**: All dependencies are installed fresh for each run
4. **Artifact Handling**: Uses GitHub Actions native artifact upload instead of Jenkins' archive system
5. **Error Handling**: Enhanced with conditional steps and proper cleanup

### Runner Requirements

For **ubuntu-latest** runners (GitHub-hosted):
- The workflow installs all required dependencies automatically
- Requires sufficient disk space and memory for Kubernetes and 5GC components

For **self-hosted** runners:
- Ensure Docker is installed and the runner user has access to Docker
- Ensure the runner has sudo access (passwordless sudo recommended)
- Configure the runner with appropriate labels
- Use the `agent_label` input when manually dispatching the workflow

### Customization

To customize the workflow:

1. **Change branches**: Modify the `on.push.branches` and `on.pull_request.branches` sections
2. **Adjust timeout**: Modify `timeout-minutes` under the job definition
3. **Add Slack notifications**: Uncomment and configure the Slack notification step with `secrets.SLACK_WEBHOOK_URL`
4. **Modify test parameters**: Edit the Make commands in the "Install Aether" and "Run gNBsim" steps

### Troubleshooting

If the workflow fails:

1. Check the workflow run logs in the Actions tab
2. Download the archived logs artifacts for detailed component logs
3. Verify that the `AETHER_QA_PEM` secret is correctly configured
4. Ensure the runner has sufficient resources (disk space, memory)
5. Check that network configuration is correct for your environment
