# Playwright CLI Workflows

Use the wrapper script and snapshot often.
Assume `PWCLI` is set and `pwcli` is an alias for `"$PWCLI"`.
In this repo, run commands from `output/playwright/<label>/` to keep artifacts contained.

## Standard interaction loop

```bash
pwcli open https://example.com
pwcli snapshot
pwcli click e3
pwcli snapshot
```

## Form submission

```bash
pwcli open https://example.com/form --headed
pwcli snapshot
pwcli fill e1 "user@example.com"
pwcli fill e2 "password123"
pwcli click e3
pwcli snapshot
pwcli screenshot
```

## Data extraction

```bash
pwcli open https://example.com
pwcli snapshot
pwcli eval "document.title"
pwcli eval "el => el.textContent" e12
```

## Debugging and inspection

Capture console messages and network activity after reproducing an issue:

```bash
pwcli console warning
pwcli network
```

Record a trace around a suspicious flow:

```bash
pwcli tracing-start
# reproduce the issue
pwcli tracing-stop
pwcli screenshot
```

## Sessions

Use sessions to isolate work across projects:

```bash
pwcli --session marketing open https://example.com
pwcli --session marketing snapshot
pwcli --session checkout open https://example.com/checkout
```

Or set the session once:

```bash
export PLAYWRIGHT_CLI_SESSION=checkout
pwcli open https://example.com/checkout
```

## Configuration file

By default, the CLI reads `playwright-cli.json` from the current directory. Use `--config` to point at a specific file.

Minimal example:

```json
{
  "browser": {
    "launchOptions": {
      "headless": false
    },
    "contextOptions": {
      "viewport": { "width": 1280, "height": 720 }
    }
  }
}
```

## Troubleshooting

- If an element ref fails, run `pwcli snapshot` again and retry.
- If the page looks wrong, re-open with `--headed` and resize the window.
- If a flow depends on prior state, use a named `--session`.


## Headless WebRTC audio checks

RTP packets and transcripts do not establish decoded audio. Verify the browser
consumer first with a local peer-to-peer synthetic tone before calling a billed
voice service. In an environment without a normal output device, an explicit
silent sink can make the receive pipeline testable without physical playback:
use an `AudioContext({sinkId: {type: "none"}})` where supported, start a muted
`Audio` element with the received stream, and connect the analyser through a
zero-gain node to that context destination. Check that playback/resume succeeds.

Inspect waveform peaks, recorded bytes and emitted/received sample counts.
`totalAudioEnergy` can remain zero while the decoded waveform and recording are
valid; do not use that field alone as the pass condition. A local Chromium trial
with this setup decoded a synthetic tone and recorded nonempty audio with no
model calls or physical media. Preserve earlier service outcomes and complete
only missing observations or distinct useful follow-ups; do not replay paid
work to repair a verifier.
