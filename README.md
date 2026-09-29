# Network NOC Monitor

Network NOC Monitor documents a small, self-managed path for securely reaching a Linux device and preparing Grafana Cloud to receive its monitoring metrics. The screenshots in [`docs/images`](docs/images/) walk through the setup shown here.

> **Documentation scope:** The repository currently contains setup screenshots only. It does not include an exporter, Grafana dashboard, alert rules, or application code that sends metrics. Those pieces must be installed and configured separately.

## What you will set up

1. A Tailscale account and a Linux device joined to its tailnet.
2. Tailscale SSH access on that device.
3. A Grafana Cloud stack with a Prometheus metrics destination and an API token.

Tailscale provides private connectivity to the device. Grafana Cloud is the metrics destination. The captured workflow does not show the device exporting or sending metrics to Grafana, so completing these steps alone does not produce monitoring data.

## Prerequisites

- A Linux device you administer, with a working internet connection and `sudo` access. The screenshots use a Raspberry Pi running Raspberry Pi OS, but do not establish compatibility with every distribution.
- A web browser and accounts for GitHub (used for Tailscale sign-in in the screenshots), Tailscale, and Grafana Cloud.
- Permission to authorize the device in your Tailscale network and create Grafana access-policy tokens.

## 1. Create or access a Tailscale account

The screenshots show Tailscale's GitHub authorization and first-run questions. Sign in with an account you control and complete the onboarding prompts. The selected onboarding answers are personal choices and are not required by this guide.

![Tailscale requests read-only organization and email access during GitHub authorization.](docs/images/captura1.png)

![Tailscale asks for a primary use case during onboarding.](docs/images/captura2.png)

![Tailscale asks for a role during onboarding.](docs/images/captura3.png)

## 2. Install Tailscale and join the Linux device

On the Linux device, install Tailscale using the official installation instructions for your distribution. The screenshot shows the official install script being run from a terminal:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Review third-party installation scripts before running them, and use Tailscale's current official instructions if the command or supported operating systems have changed.

After installation, start the login flow:

```bash
sudo tailscale up
```

Open the one-time URL printed by the command, sign in, and approve the device for your tailnet. The URL is unique to the login session; do not share it.

![The Linux terminal shows the Tailscale installation command.](docs/images/captura4.png)

![The installation completes and reports that tailscaled is installed.](docs/images/captura5.png)

![The tailscale up command prints a one-time authentication URL.](docs/images/captura6.png)

![The browser asks to connect the device named delta to the tailnet.](docs/images/captura7.png)

![Expanded device details show the hostname and operating system; the public key is redacted.](docs/images/captura8.png)

In the Tailscale admin console, confirm the device appears as connected. The example device is named `delta`; yours may have a different name and address.

![The Tailscale Machines page lists the connected Linux device.](docs/images/captura9.png)

## 3. Enable Tailscale SSH

On the device, enable Tailscale SSH:

```bash
sudo tailscale set --ssh
```

The screenshot shows the command returning to the shell without an error. Review your tailnet's access controls and Tailscale SSH policy so only intended users can connect.

![The terminal shows Tailscale SSH being enabled.](docs/images/captura10.png)

## 4. Create a Grafana Cloud stack

Create or sign in to a Grafana Cloud account, complete the account setup, and select a deployment region that meets your operational and data-location needs. Then choose the server or VM monitoring path, or skip the recommendations to open the full Grafana interface.

![Grafana Cloud account creation page.](docs/images/captura11.png)

![Grafana Cloud asks for a deployment region when creating a stack.](docs/images/captura12.png)

![Grafana Cloud asks what kind of target you want to monitor.](docs/images/captura13.png)

![Grafana Cloud landing page recommends connecting a data source.](docs/images/captura14.png)

## 5. Prepare a Prometheus metrics destination

From Grafana Cloud, open the Prometheus onboarding flow. The screenshots choose the managed Prometheus stack and show custom setup options, then select **Send Metrics over HTTP** as the connection method.

![Prometheus onboarding offers a managed stack or an existing Prometheus instance.](docs/images/captura15.png)

![The onboarding page shows Linux monitoring and several ways to connect Prometheus data.](docs/images/captura16.png)

## 6. Generate an API token

In the HTTP Metrics configuration, select the metrics format required by your sender (the screenshot has **Otel** selected). Create an access-policy token with the write scope needed for your telemetry. The captured form includes `metrics:write`, `logs:write`, `traces:write`, and `profiles:write`; grant only the scopes your sender actually requires.

The screenshots show the token-creation form and confirmation. A token is a secret: copy it when Grafana displays it, store it in a secrets manager or protected environment variable, and never commit it to this repository, a dashboard, or a screenshot. The images do not show a token value.

![HTTP Metrics configuration with format selection and token creation fields.](docs/images/captura17.png)

![Grafana confirms that an API key was generated.](docs/images/captura18.png)

## Next step: send metrics

Use the endpoint, authentication details, and code or agent configuration displayed by your Grafana Cloud stack to configure a Prometheus-compatible sender on the device. Keep the token private and verify that the sender can reach Grafana Cloud. Then query the expected metrics in Grafana Explore or a dashboard.

The repository screenshots end at token creation and do not provide a sender configuration, endpoint, dashboard, alert rules, or validation procedure. These values depend on your Grafana stack and the exporter or collector you choose, so do not copy credentials or endpoints from someone else's setup.

## Troubleshooting

- **The device does not appear in Tailscale:** Check that `tailscaled` is running and repeat `sudo tailscale up`. Complete authorization in the browser and confirm the device is approved in the admin console.
- **Tailscale SSH is unavailable:** Confirm `sudo tailscale set --ssh` succeeded, then review your tailnet's SSH access rules and user permissions.
- **Grafana shows no data:** Creating a stack and API token does not send metrics. Configure and start a compatible exporter or collector, confirm its destination and token, and inspect its logs for delivery errors.
- **A token may have been exposed:** Revoke it in Grafana Cloud and create a replacement with the minimum required scopes.

## Screenshot index

All screenshots are stored in [`docs/images`](docs/images/). They are referenced inline above in the order of the workflow.

## License

See [LICENSE](LICENSE).
