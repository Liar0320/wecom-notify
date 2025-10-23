# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a GitHub Action that automatically installs and configures the Aliyun CLI for Alibaba Cloud services in GitHub workflows. The action is implemented as a composite action using bash scripts with multi-architecture support (ARM64/AMD64).

## Key Files

- `action.yaml`: Main action definition with inputs, outputs, and execution steps
- `my.secrets`: Template for required secrets (Access Key ID and Secret)
- `my.variables`: Template for environment variables (region, bucket destination)
- `test-resource/sample.html`: Sample file for testing OSS operations

## Action Configuration

The action supports two authentication modes:
- `AK` (Access Key): Default mode using access-key-id and access-key-secret
- `StsToken`: Uses temporary credentials with sts-token

### Required Inputs
- `region`: Alibaba Cloud region (e.g., cn-shanghai)
- `access-key-id`: Access key ID from secrets
- `access-key-secret`: Access key secret from secrets

### Optional Inputs
- `version`: Aliyun CLI version (defaults to "3.0.161", supports "latest")
- `mode`: Authentication mode (defaults to "AK")
- `sts-token`: Required only when mode is "StsToken"

## Architecture

The action works by:
1. Detecting system architecture (ARM64 vs AMD64)
2. Downloading the appropriate Aliyun CLI binary from GitHub releases
3. Installing it to `/usr/local/bin/aliyun`
4. Configuring authentication using provided credentials
5. Making the CLI available for subsequent workflow steps

## Development Commands

### Local Testing with act
```bash
# Install act
brew install act

# Pull required Docker image
docker pull catthehacker/ubuntu:act-latest --platform linux/amd64

# Test the action locally using act (GitHub Actions local runner)
act --container-architecture linux/amd64 --input trigger=manually --secret-file my.secrets --var-file my.variables -W .github/workflows/test_with_local_actions.yaml
```

### Testing Setup
- Copy `my.secrets` and fill in actual credentials
- Modify `my.variables` with your specific region and bucket settings
- Test with sample files in `test-resource/` directory

### Workflow Testing
- `.github/workflows/test_with_local_actions.yaml`: Tests the local action (uses `./`)
- `.github/workflows/test_with_marketplace_actions.yaml`: Tests the published marketplace version

## Common Usage Pattern

```yaml
- name: Install Aliyun CLI
  uses: Liar0320/aliyun-cli-action@v1.0.2
  with:
    access-key-id: ${{ secrets.ALIYUN_ACCESS_KEY_ID }}
    access-key-secret: ${{ secrets.ALIYUN_ACCESS_KEY_SECRET }}
    region: ${{ vars.ALIYUN_REGION }}

- name: Use Aliyun CLI
  run: |
    aliyun oss ls ${{ vars.BUCKET_DESTINATION }}
    aliyun oss cp file.txt ${{ vars.BUCKET_DESTINATION }}
    aliyun oss cp ./dist/ ${{ vars.BUCKET_DESTINATION }} -r
```