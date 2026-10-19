# Evidence — listing request

Nothing was executed to file this request, so there is no test transcript to show. What the request
rests on is the plugin's own evidence, kept in
`../../hermes-agent/2026-10-19-plugin-catalog-degeneracy-guard/evidence.md`:

- `hermes plugins validate --install-deps` passed at the pinned commit, against both current `main`
  and the declared floor `v0.22.2`, with `security scan — safe`
- 37 tests pass in three environments (no Hermes importable, Hermes `main`, Hermes `v0.22.2`)
- the approval gate was driven end to end: deny blocks, "always" stops the prompt for that tool,
  and an unattended process is refused with the gate's own message
- two hold-out sets were scored once each: 40 legitimate payloads asked zero times in the first,
  12 legitimate payloads asked six times in the second — the number that made the request honest
  rather than promotional

The listing body says what the plugin does not do as precisely as what it does: no network, no
shell, no subprocesses, no own threads, no telemetry, no credentials. That claim is verifiable in
the plugin source, and it is the claim a reader of an "awesome" list actually needs.
