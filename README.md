# SnapCD Runner - Docker Deployment

This guide explains how to run the SnapCD Runner using Docker.

## Prerequisites

- Docker and Docker Compose installed
- Network access to [snapcd.io](https://snapcd.io)
- Organization ID and Runner ID, as well as Service Principal credentials (Client ID and Client Secret) from [snapcd.io](https://snapcd.io)
- Terraform and/or OpenTofu binaries installed on the host (the container mounts these from the host)

## Configuration

All configuration files are located in the `config/` directory.

### 1. Application Settings

Copy the example configuration file:

```bash
cp config/appsettings.example.json config/appsettings.json
```

Edit `config/appsettings.json` with your configuration values:

| Setting | Description |
|---------|-------------|
| `Runner.Id` | The unique ID of the runner (GUID format, obtained from SnapCD server) |
| `Runner.Instance` | A friendly name for this runner instance |
| `Runner.Credentials.ClientId` | Service Principal client ID for authentication |
| `Runner.Credentials.ClientSecret` | Service Principal client secret for authentication |
| `Runner.OrganizationId` | The organization ID this runner belongs to |
| `Server.Url` | The URL of your SnapCD server |
| `WorkingDirectory.WorkingDirectory` | Directory where the runner stores working files |
| `WorkingDirectory.TempDirectory` | Directory for temporary files |
| `HooksPreapproval.Enabled` | Enable/disable hooks preapproval feature |
| `HooksPreapproval.PreapprovedHooksDirectory` | Directory containing preapproved hook scripts |

### 2. SSH Keys (Optional)

If your Terraform/OpenTofu modules use private Git repositories over SSH, you need to provide SSH keys.

Place your SSH private key:

```bash
cp ~/.ssh/id_rsa ./config/id_rsa
chmod 700 ./config/id_rsa
```

The `config/known_hosts` file is provided with GitHub and GitLab host keys pre-configured. If you need to add additional hosts:

```bash
ssh-keyscan your-git-server.com >> config/known_hosts
```

### 3. Engine (Terraform/OpenTofu) Binaries

You must provide your own `Engine` binary (either `tofu` or `terraform` or both, depending on your needs), for example by mounting them from your local host. You can find their installation directories with:

> Please note that Snap CD strives to support the latest available version of `tofu`. For `terraform` we design fo binaries up to release [1.5.7](https://github.com/hashicorp/terraform/releases/tag/v1.5.7), which was the final release under the [Mozilla Public License 2.0](https://github.com/hashicorp/terraform/blob/v1.5.7/LICENSE). It was not developed for later versions, such as those published under the [Business Source License 1.1 (BSL 1.1)](https://github.com/hashicorp/terraform/blob/v1.6.0/LICENSE).

```bash
which terraform
which tofu
```

Update the paths in `docker-compose.yml` if your binaries are in different locations.

### 4. Pre-approved Hooks (Optional)

If you want to use the hooks preapproval feature, place your approved hook scripts in the `config/preapproved-hooks` directory:

```bash
cp /path/to/your/hook-script.sh config/preapproved-hooks/
```

Then enable the feature in `config/appsettings.json`:

```json
{
  "HooksPreapproval": {
    "Enabled": true,
    "PreapprovedHooksDirectory": "/app/preapproved-hooks"
  }
}
```

## Running

Start the runner:

```bash
docker compose up
```

View logs:

```bash
docker compose logs -f
```

Stop the runner:

```bash
docker compose down
```

### Azure Authentication

If you wish to use the `azurerm` or `azuread` terraform providers using the Azure CLI, exec into the runner and log in.

```bash
docker compose exec snapcd-runner /bin/bash
az login
```

Follow the browser prompts to complete authentication. You'll need to repeat this when the login expires.

## Docker Image

This deployment uses the `ghcr.io/schrieksoft/snapcd-runner:azure-0.1.4` image, which includes:

- .NET 10 runtime
- Git and SSH client
- Azure CLI (in order to use the `azurerm` and `azuread` terraform providers)

## Network Mode

The container runs with `network_mode: host` for simplified networking. This allows the runner to access services on localhost and avoids port mapping complexity.
