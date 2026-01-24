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
3. **Install kubectl**: Uses Azure's setup-kubectl action (default/latest version as specified in requirements)
4. **Install Helm**: Uses Azure's setup-helm action (default/latest version as specified in requirements)
5. **Configure OnRamp**: Sets up SSH keys, generates hosts.ini, configures vars/main.yml
6. **Install Aether**: Installs Kubernetes, 5GC core, and gNBsim
7. **Run gNBsim**: Executes the gNBsim test with retry logic (2 attempts)
8. **Validate Results**: Checks that tests passed using the same validation pattern as Jenkins
9. **Retrieve Logs**: Collects logs from all components
10. **Archive Artifacts**: Uploads logs as workflow artifacts
11. **Cleanup**: Uninstalls all components (always runs, even on failure)
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
5. **Pin tool versions**: If you need stable versions instead of 'latest', modify the `version` parameter for kubectl and helm actions (e.g., `version: 'v1.28.0'` or `version: 'v3.12.0'`)

### Important Notes

- **Tool Versions**: The workflow uses 'latest' versions for kubectl and helm as per the requirements. This ensures the most recent stable versions are used. If your environment requires specific versions for compatibility, you can pin them in the workflow file.
- **Validation Pattern**: The gNBsim validation uses the exact same grep pattern as the Jenkins workflow (`grep "Ue's Passed" | grep -v "Passed: 0"`). This matches the expected output format from gNBsim.

### Troubleshooting

If the workflow fails:

1. Check the workflow run logs in the Actions tab
2. Download the archived logs artifacts for detailed component logs
3. Verify that the `AETHER_QA_PEM` secret is correctly configured
4. Ensure the runner has sufficient resources (disk space, memory)
5. Check that network configuration is correct for your environment
