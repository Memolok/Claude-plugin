# Connecting to Memolok

## How the connection works

The plugin ships one thing — the server URL:

```json
{
  "mcpServers": {
    "Memolok": {
      "type": "http",
      "url": "https://www.memolok.ai/mcp"
    }
  }
}
```

Connecting registers the client automatically and opens a browser to sign in at `memolok.ai`. There is
no client id, no client secret, and no token to paste.

**Never ask the user for a client id or a client secret.** None exist in this system. If a client's
Advanced settings offer those fields, they must be left empty — a stale value there causes the
connection to fail.

An account is required, and there is no public sign-up. Accounts are provisioned by a Memolok
administrator.

## Clients without a browser sign-in

Some clients cannot run a browser OAuth flow. They use a Personal Access Token instead:

```json
{
  "mcpServers": {
    "Memolok": {
      "url": "https://www.memolok.ai/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_PAT"
      }
    }
  }
}
```

Tokens are issued by a Memolok administrator. You cannot mint one, and the user should not be asked to
paste one into a conversation — it belongs in their client configuration.

## Diagnosing a failure

Start with `ping`. It is the only tool that tells you about the connection rather than the ledger.

| Symptom | Meaning | Fix |
| --- | --- | --- |
| Tools absent entirely | The server was never connected | Connect it in the client's settings |
| Every call returns unauthorized | No valid credential | Re-run the browser sign-in, or check the token |
| `Authenticated token has no linked Memolok user.` | The credential is valid but not linked to an account | Reconnect to sign in again, or the token needs an administrator |
| `The access token carries a malformed Memolok user id.` | The credential is valid but its identity claim is unusable | Reconnect to sign in again; if it recurs, report it |
| `This PAT's client id is malformed.` | The Personal Access Token is damaged or mistyped | Check the token, or an administrator issues a new one |
| `Memolok Decision Ledger not found.` on a read | Authenticated, but not a member of that ledger | `get_MDLs` to see what they can actually reach |
| `You are not a member of this Memolok Decision Ledger.` on a write | Reading is fine, writing is not | An administrator adds them |
| Writes refused, reads fine | `visitor` role | An administrator changes the role |
| "Automatic client registration isn't supported" | Server-side registration issue | Not fixable by the user; report it |

The key distinction: **not authenticated** is a connection problem, while **not a member** is a
permissions problem. `ping` succeeding tells you which one you are looking at.

## An error carrying a reference

Two replies quote a `req_…` reference. They mean different things and only one is retryable.

| Reply | What to do |
| --- | --- |
| `Invalid arguments for this tool: …` followed by field names | Your call was malformed. Fix the named fields and call again |
| `Memolok hit an internal error and could not complete this request.` | Something failed on the server. **Do not retry, and do not diagnose** |

The internal-error reply is deliberately empty of detail — the server keeps the cause in its own logs
and hands out only the reference. There is nothing in the message to reason from, so an explanation
built on it would be invented. Say plainly that the operation failed, give the user the reference
verbatim, and offer to continue with something else. An administrator can resolve a reference to the
exact failure.

Retrying is worse than useless here. These are overwhelmingly deterministic faults — a rejected write
shape, a misconfiguration — and an identical call fails identically while burning the user's time. If
the same reference class appears twice in a session, stop and report rather than working around it.

Do not let a failure here become a reason to write the user's decision somewhere else. A record that
did not reach the ledger was not recorded.

Reads on a ledger the user cannot access return `Memolok Decision Ledger not found.` — identical to a
ledger that does not exist. That is deliberate, so existence is not leaked. Do not read it as proof
the ledger is gone; run `get_MDLs` and see what is actually reachable.

## Connections made elsewhere

A connector added in one Claude surface generally syncs to the others, so there is usually no need to
add it twice. If the tools are present, it is connected.

## What to tell the user

- To connect: add Memolok in their client's connector settings using **the URL alone**, and leave any
  advanced credential fields empty.
- Signing in happens in a browser, against their Memolok account.
- No account: an administrator creates one; there is no self-service sign-up.
- On a client with no browser flow: they need a Personal Access Token from an administrator, set as an
  `Authorization: Bearer` header in their own configuration.

## Never

- Ask for, accept, or handle a password, a token, or a client secret in conversation
- Suggest creating an account on the user's behalf
- Retry a failing call repeatedly hoping authentication resolves itself
- Fall back to local files when the server is unreachable — they are not the ledger, and a decision
  written to a local file is a decision that was not recorded
