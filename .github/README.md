# GitHub Actions Workflows

This directory contains GitHub Actions workflows for the Aether OnRamp project. All workflows replicate the functionality of Jenkins groovy workflows found at [opennetworkinglab/aether-jenkins](https://github.com/opennetworkinglab/aether-jenkins/).

## Overview

The workflows provide automated CI/CD for testing different Aether deployment scenarios:

| Workflow | Description | Jenkins Equivalent |
|----------|-------------|-------------------|
| `quickstart.yml` | Standard Aether deployment with gNBsim | `quickstart.groovy` |
| `quickstart-amp.yml` | Aether deployment with AMP management platform | `quickstart_amp.groovy` |
| `oai.yml` | OpenAirInterface (OAI) based deployment with emulated UE | `oai.groovy` |
| `sdran.yml` | SD-RAN (Software Defined RAN) deployment and validation | `sdran.groovy` |
| `upf-alt.yml` | Multi-UPF deployment for load balancing scenarios | `upf-alt.groovy` |

## Common Workflow Structure

All workflows follow a similar pattern with the following stages:

1. **Checkout repository**: Clones the repo with all submodules
1. **Set up Python**: Installs Python and creates a virtual environment with Ansible
1. **Install kubectl**: Uses Azure's setup-kubectl action (latest version)
1. **Install Helm**: Uses Azure's setup-helm action (latest version)
1. **Configure OnRamp**: Sets up network configuration, generates hosts.ini, configures vars/main.yml
1. **Install Aether**: Installs Kubernetes and various Aether components (varies by workflow)
1. **Run Tests**: Executes specific tests based on the deployment type
1. **Validate Results**: Checks that tests passed using appropriate validation patterns
1. **Retrieve Logs**: Collects logs from all components
1. **Archive Artifacts**: Uploads logs as workflow artifacts
1. **Cleanup**: Uninstalls all components (always runs, even on failure)
1. **Notify on Failure**: Logs failure information (can be extended with Slack notifications)

## Workflow Details

### quickstart.yml - Aether OnRamp Quickstart

**Purpose**: Standard Aether deployment with gNBsim for basic 5G core testing

**Components Installed**:
- Kubernetes (RKE2)
- Aether 5G Core
- gNBsim (gNodeB Simulator)

**Test Execution**:
- Runs `make aether-gnbsim-run`
- Validates that gNBsim tests pass with retry logic (2 attempts)

**Validation**:
- Checks gNBsim summary.log for "Ue's Passed" with non-zero count

### quickstart-amp.yml - Aether OnRamp Quickstart with AMP

**Purpose**: Aether deployment including AMP (Aether Management Platform) for enhanced management capabilities

**Components Installed**:
- Kubernetes (RKE2)
- Aether AMP (Management Platform)
- Aether 5G Core
- gNBsim (gNodeB Simulator)

**Test Execution**:
- Runs `make aether-gnbsim-run`
- Validates that gNBsim tests pass with retry logic (2 attempts)

**Validation**:
- Checks gNBsim summary.log for "Ue's Passed" with non-zero count

### oai.yml - OAI Test

**Purpose**: OpenAirInterface deployment with emulated UE for realistic radio testing

**Components Installed**:
- Kubernetes (RKE2)
- Aether 5G Core
- OAI gNodeB
- OAI UE Simulator

**Test Execution**:
- Runs `make oai-uesim-start`
- Tests connectivity with emulated UE

**Validation**:
- Pings 192.168.250.1 from UE interface (oaitun_ue1)
- Checks for "0% packet loss" in ping results

### sdran.yml - SD-RAN Test

**Purpose**: Software Defined RAN deployment for testing RAN intelligence and control

**Components Installed**:
- Kubernetes (RKE2)
- Aether 5G Core
- SD-RAN components (ONOS, RAN Simulator)

**Test Execution**:
- Deploys SD-RAN infrastructure
- Runs RAN simulator for testing

**Validation**:
- Queries ONOS services for metrics, UE count, and cell information
- Validates that SD-RAN components are responding correctly

### upf-alt.yml - Multi-UPF Test

**Purpose**: Tests multi-UPF (User Plane Function) deployment scenarios for load balancing and redundancy

**Components Installed**:
- Kubernetes (RKE2)
- ROC (Radio Operations Center)
- Aether 5G Core
- Additional UPF instance
- Multiple gNBsim instances

**Test Execution**:
- Configures multiple UPFs using ROC
- Runs multiple gNBsim instances (gnbsim-1 and gnbsim-2)

**Validation**:
- Checks both gNBsim instances for "Profile Status: PASS"
- Validates load balancing across UPFs

## Triggering Workflows

All workflows support two trigger methods:

### Manual Dispatch
Go to Actions → Select workflow → Run workflow
- Allows selection of custom runner label for self-hosted runners
- Default uses `ubuntu-latest` GitHub-hosted runners

### Automatic Trigger
- Automatically runs on pushes to the `main` branch
- Provides continuous integration for main branch changes

## Common Features

### Retry Logic
All workflows include retry logic for test execution (2 attempts) to handle transient failures.

### Comprehensive Logging
- All component logs are collected and archived as workflow artifacts
- Logs include container logs, Kubernetes pod logs, and test output
- Available for download after workflow completion

### Robust Cleanup
- Cleanup steps run regardless of workflow success/failure (`if: always()`)
- Ensures clean environment for subsequent runs
- Prevents resource leaks

### Network Configuration
- Automatically detects runner IP and network interface
- Configures Ansible for local execution (no SSH required)
- Sets up appropriate network settings for each deployment type

## Differences from Jenkins Workflows

The GitHub Actions workflows include several enhancements over the original Jenkins implementations:

1. **Runtime Installation**: kubectl, helm, and Python virtualenv are installed at runtime
1. **Local Execution**: Ansible runs with `ansible_connection=local` instead of SSH
1. **Self-Contained**: All dependencies are installed fresh for each run
1. **Enhanced Error Handling**: Better conditional steps and cleanup procedures
1. **Artifact Management**: Comprehensive log collection and artifact archiving
1. **Timeout Protection**: 60-minute timeout prevents hung workflows

## Troubleshooting

If a workflow fails:

1. **Check Workflow Logs**: Review the workflow run logs in the Actions tab
1. **Download Artifacts**: Download archived logs for detailed component analysis
1. **Resource Requirements**: Ensure runner has sufficient resources (disk space, memory)
1. **Network Configuration**: Verify network connectivity and firewall settings
1. **Component Status**: Check Kubernetes pod status and Docker container health
1. **Retry Mechanism**: Note that test steps include automatic retry logic

### Common Issues

- **Timeout Errors**: Workflows have 60-minute timeout; consider using self-hosted runners for better performance
- **Network Connectivity**: Ensure runner can access required network resources
- **Resource Constraints**: GitHub-hosted runners have limited resources; self-hosted may be needed for complex scenarios
- **DNS Resolution**: Some deployments may require specific DNS configuration

## Self-Hosted Runners

For better performance and resource control, consider using self-hosted runners:

1. Set up self-hosted runners with appropriate labels
1. Use the `agent_label` input parameter when manually triggering workflows
1. Ensure runners have sufficient resources for Kubernetes and container workloads
1. Configure network access for the specific deployment requirements

## Extending Workflows

To extend or modify workflows:

1. **Add Slack Notifications**: Uncomment and configure Slack webhook URLs in failure notification steps
1. **Custom Validation**: Modify validation steps for specific test requirements
1. **Additional Components**: Add make targets for additional Aether components
1. **Environment Variables**: Use workflow inputs or repository secrets for configuration
1. **Matrix Builds**: Consider adding matrix strategies for testing multiple configurations

All workflows are designed to be maintainable and extensible while preserving compatibility with the original Jenkins implementations.