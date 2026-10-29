# @pipeworx/nist-fluid-properties

Density, viscosity, thermal conductivity, speed of sound, enthalpy, entropy,
Cv/Cp and phase for ~76 reference fluids — water, nitrogen, oxygen, argon,
carbon dioxide, light hydrocarbons, and common refrigerants (R11-RC318) — at a
caller-given temperature and pressure, or along the liquid/vapor saturation
curve. Every value carries its unit.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `nist_fluid_properties(fluid, temperature, temperature_unit?, pressure, pressure_unit?)`
  — density, specific volume, internal energy, enthalpy, entropy, Cv, Cp,
  speed of sound, Joule-Thomson coefficient, viscosity, thermal conductivity
  and phase (liquid/vapor/supercritical) at one temperature/pressure point.
- `nist_fluid_saturation(fluid, temperature? OR pressure?, temperature_unit?, pressure_unit?)`
  — give a saturation temperature (returns the matching saturation pressure)
  or a saturation pressure (returns the matching saturation temperature),
  plus liquid- and vapor-side density, enthalpy, entropy, viscosity, thermal
  conductivity and surface tension on the phase boundary.
- `nist_fluid_list(query?)` — the ~76 fluids this pack covers, optionally
  filtered by a substring of the name or refrigerant code. An unrecognized
  `fluid` argument to either of the tools above also returns this same list.

## Auth

Keyless.

## Data sources

- NIST, Chemistry WebBook SRD 69 — Thermophysical Properties of Fluid
  Systems. A public, keyless web form; every call proxies it live. The only
  thing this pack bakes in is a ~76-row fluid-name-to-NIST-id lookup table
  (not the measurements) so a caller can say "water" or "R134a" instead of
  NIST's internal identifier.

## Notes

- **"Air" is not in this dataset as a single fluid.** NIST's own fluid-
  property form does not carry a pseudo-pure "air" entry — query nitrogen,
  oxygen and argon separately if that's what you need.
- Fluid names, formulas and refrigerant codes are all accepted and matched
  loosely (case/space/hyphen-insensitive): `"water"`, `"H2O"`, `"co2"`,
  `"Carbon Dioxide"`, `"R134a"`, `"r-134a"` all resolve.
- Requesting a condition outside a fluid's valid range (below its triple
  point, past its upper temperature/pressure limit, or an otherwise unstable
  point) comes back from NIST as a range error — surfaced as a `user_error:`
  with a suggestion to try a condition nearer standard temperature/pressure.
- Viscosity is returned in `cP`, which is numerically identical to `mPa·s`
  (1 cP = 1 mPa·s) — no conversion needed if your use case wants SI.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "nist-fluid-properties": {
      "url": "https://gateway.pipeworx.io/nist-fluid-properties/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/nist-fluid-properties/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/nist_fluid_properties \
  -H 'Content-Type: application/json' \
  -d '{"fluid":"water","temperature":25,"temperature_unit":"C","pressure":1,"pressure_unit":"atm"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/nist_fluid_properties`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-nist-fluid-properties` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/nist-fluid-properties/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "nist-fluid-properties": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-nist-fluid-properties"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Nist Fluid Properties data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
