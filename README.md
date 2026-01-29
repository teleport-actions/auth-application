<div align="center">
   <img src="https://github.com/gravitational/teleport-actions/raw/main/assets/img/readme-header.png" width=750/>
   <div align="center" style="padding: 25px">
      <a href="https://www.apache.org/licenses/LICENSE-2.0">
      <img src="https://img.shields.io/badge/Apache-2.0-red.svg" />
      </a>
   </div>
</div>
</br>

> Read our Blog: <https://goteleport.com/blog/>

> Read our Documentation: <https://goteleport.com/docs/getting-started/>

# `teleport-actions/auth-application@v2`

`auth-application` uses Teleport Machine ID to generate credentials for
accessing an application protected by Teleport.

Pre-requisites:

- **Teleport 16 or above must be used.** Use
  [`teleport-actions/auth-application@v1`](https://github.com/teleport-actions/auth-application/tree/v1)
  for compatibility with older versions of Teleport.
- Teleport binaries must already be installed in the job environment.
- The Application you wish to access must already be connected to your Teleport
  cluster. See
  <https://goteleport.com/docs/enroll-resources/application-access/getting-started/>
- You must have created a bot with a role with access to your Application and
  created a GitHub join token that allows that bot to join.
- A Linux based runner.

Example usage:

```yaml
on:
  workflow_dispatch: {}
jobs:
  demo-auth-application:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
    steps:
      - name: Install Teleport
        uses: teleport-actions/setup@v1
        with:
          # specify version as "auto" and provide the address of your Teleport
          # proxy using the "proxy" input.
          version: auto
          proxy: tele.example.com:443
      - name: Fetch application credentials
        id: auth
        uses: teleport-actions/auth-application@v2
        with:
          # Specify the publically accessible address of your Teleport proxy.
          proxy: tele.example.com:443
          # Specify the name of the join token for your bot.
          token: my-github-join-token-name
          # Specify the length of time that the generated credentials should be
          # valid for. This is optional and defaults to "1h"
          certificate-ttl: 1h
          # Specify the name of the application you wish to access.
          app: grafana-example
          # Enable submission of anonymous usage telemetry to Teleport.
          # See https://goteleport.com/docs/machine-id/reference/telemetry/ for
          # more information.
          anonymous-telemetry: 1
      - name: Make request
        run: curl --cert ${{ steps.auth.outputs.certificate-file }} --key ${{ steps.auth.outputs.key-file }} https://grafana-example.tele.example.com/api/users
```

## Outputs

This action will output the following values:

- `destination-dir`: the path to the tbot destination folder.
- `identity-file`: the path to the identity file.
- `certificate-file`: the path to the client certificate.
- `key-file`: the path to the private key for the client certificate.
