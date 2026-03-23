# Install & Recovery

> **Prerequisite:** Read the [main skill](../SKILL.md) for authentication, global flags, and key concepts.

This reference covers CLI installation, Sim API authentication failure recovery, and version compatibility checks. These steps are only needed when a `dune sim` command fails.

---

## CLI Not Found Recovery

If the command fails because `dune` is not found on PATH (e.g. "command not found"), follow the installation steps in the [shared CLI install reference](../../shared/cli-install.md#cli-not-found-recovery).

---

## Sim Authentication Failure Recovery

If a `dune sim` command fails with an authentication error (e.g. 401, "authentication failed: check your Sim API key", "missing Sim API key"), the user needs to authenticate.

**Important:** The Sim API key (`sim_` prefix) is separate from the Dune API key used by `dune query` commands. Having a valid Dune API key does NOT authenticate Sim API requests.

**Stop and ask the user** to authenticate in a separate terminal:

1. Go to [https://sim.dune.com/](https://sim.dune.com/), click **Keys**, then click **New** to create a new API key (if they don't have one)
2. Run `dune sim auth` in their terminal and enter the key when prompted
3. Come back to the conversation once done

Then verify the connection by running:

```bash
dune sim evm token-info native --chain-ids 1 -o json
```

If this returns token metadata for ETH, auth is working. Retry the original command. If it still fails, inform the user their key appears invalid and ask them to retry.

Do **not** attempt to handle the API key yourself -- the user must authenticate outside of this session.

### Common Errors

| Error | Cause | Recovery |
|-------|-------|----------|
| `missing Sim API key: set DUNE_SIM_API_KEY, pass --sim-api-key, or run 'dune sim auth'` | No key found in flag, env, or config | Create a key at [sim.dune.com](https://sim.dune.com/) and run `dune sim auth` |
| `authentication failed: check your Sim API key` | Key is invalid, expired, or revoked (HTTP 401) | Create a new key at [sim.dune.com](https://sim.dune.com/) and re-authenticate |
| `Sim API error (HTTP 403): API key does not have permissions to access this endpoint` | Key lacks the required permission scope | Create a new key with the correct permissions at [sim.dune.com](https://sim.dune.com/) |
| `Sim API error (HTTP 402): This API request would exceed your quota limit...` | Compute unit quota exhausted | Contact sales@dune.com to upgrade the plan |
| `rate limit exceeded: try again later` | Too many requests per time window (HTTP 429) | Wait and retry; reduce request frequency |
| `bad request: <message>` | Invalid parameters (e.g. malformed address, invalid chain ID) (HTTP 400) | Check the flag values and address format |
| `not found: <message>` | Unsupported chain or unknown endpoint (HTTP 404) | Run `dune sim evm supported-chains` to check available chains and endpoints |
| `Sim API server error (HTTP <code>): try again later` | Upstream service error (HTTP 5xx) | Wait and retry; if persistent, check Dune status page |

---

## Version Compatibility

See the [shared CLI version compatibility reference](../../shared/cli-install.md#version-compatibility).

---

## See Also

- [Main skill](../SKILL.md) -- Command overview and key concepts
