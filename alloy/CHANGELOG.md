# Changelog

The add-on version is `<Grafana Alloy version>.<add-on revision>`. Alloy's own
release notes: https://github.com/grafana/alloy/releases

Every heading is the bare version, nothing else on the line. Home Assistant
shows the section for the version it is about to install by matching
`^#* <version>` against this file and falls back to showing the whole file
when that fails (core, `homeassistant/components/hassio/update.py`).

The top section is written by Renovate when it bumps Grafana Alloy: it carries
the release the add-on bundles now, and pushes the previous one down as its own
section. Add-on changes of our own get a section with a fourth component, and
so does a base image update, written by the revision bot. Every new section
goes directly under the marker below, which Renovate needs to find its
heading.

<!-- renovate:alloy-release -->
## 1.22.0.3-local

Local build of dierochade/ha-addon-alloy (branch `local`): 1.22.0.3 (the
stack up to `fix/hardening`, plus the CI changes of `ci/hardening`) with
the AppArmor fix from sporqist/ha-addon-alloy#38, built on the Home
Assistant host. What changes on a host coming from 1.21.3.2-local:

- Grafana Alloy 1.22.0.
- `loki_max_retries` defaults to `0`: a Grafana Loki outage delays lines
  instead of dropping them.
- Three Grafana Alloy crashes within 60 s stop the add-on (Watchdog, when
  on, restarts it).
- `level_from_message`: lowercase levels (`s6-rc: info:`), the leftmost
  level in the head of the line, logfmt in any case, `[warn]`.
- New options, off until set: `strip_ansi`, `multiline_containers`,
  `self_metrics`.
- pprof off, URLs masked in the log, narrower AppArmor profile.

See 1.21.3.3 to 1.22.0.3 below.

## 1.22.0.3

- Grafana Alloy's profiling endpoints (`/debug/pprof`) are off. The
  HTTP server has no authentication and is reachable from Home
  Assistant Core and every other app; the debug UI stays.
- The add-on log masks a `user:password@` in a URL, and warns when
  credentials go to an `http://` URL.
- The `loki_url` localhost check also refuses `*.localhost`,
  `localhost.localdomain`, `0`, the long IPv6 loopback forms and
  IPv4-mapped loopback.
- AppArmor: Grafana Alloy may read its storage directory and the four
  credential files in `/data`, no longer all of it (`options.json`
  holds every secret). The s6 template rules no longer allow writing
  into the directories they execute from; the unused bashio rule is
  gone.
- `unzip` is removed from the image after the Grafana Alloy archive is
  unpacked.

## 1.22.0.2

- New option `loki_max_retries`, default `0`: a failed push is retried
  until it succeeds. Grafana Alloy's own default (10 retries, about 8.5
  to 13.5 minutes) dropped every line of a longer Grafana Loki outage.
  The journal keeps the lines while the push waits, so an outage now
  delays them. Set a number to get the old behaviour back.
- A crash series stops the add-on: three Grafana Alloy crashes within
  60 s (a configuration that parses but cannot start, for example) no
  longer loop for ever inside a container that shows as started. A
  single crash is still restarted in place. With Watchdog on, Home
  Assistant restarts the stopped add-on.
- New option `self_metrics`: Grafana Alloy's own metrics, as job
  `alloy`, through the metrics remote write - among them the counters
  of lines sent to and dropped by Grafana Loki. The AppArmor profile
  allows the `/proc/uptime` read Grafana Alloy's resource metrics need
  when anything scrapes them, and quietly denies the module list and
  PID 1's environment, which only its container probe reads.

## 1.22.0.1

- `level_from_message` reads the level from the head of the line (its
  first 80 characters) and takes the leftmost one there. A line that
  starts with `ERROR` stays an error even if `level=debug` or `INFO`
  appears later in it. New forms: logfmt in any case (`level=INFO`, as
  Go's slog writes it) and a lowercase level in brackets after a
  timestamp (`[warn]`, as nginx writes it). A logfmt `level=` beyond the
  head is the fallback.
- `multiline_containers` is compiled before Grafana Alloy starts: an
  invalid regex (`x|y)`, a stacked quantifier) stops the add-on with the
  reason instead of failing inside Grafana Alloy at start.
- `host_metrics: false` (alloy-host) turns host metrics off. It was read
  as on: jq's `//` treats `false` like a missing value.
- Docs: a "Health and restarts" section (Home Assistant restarts an
  unhealthy add-on only with Watchdog on, which is off by default), a
  silence alert that fires (`absent_over_time`), and the journal history
  read on first start (7 h, not all of it). Comments that called the
  `watchdog:` key obsolete are corrected.

## 1.22.0

Grafana Alloy 1.22.0; the add-on itself is unchanged.
Upstream release notes: https://github.com/grafana/alloy/releases/tag/v1.22.0

## 1.21.3.5-local

Local build of dierochade/ha-addon-alloy (branch `local`): 1.21.3.5 plus
the AppArmor fix from sporqist/ha-addon-alloy#38, built on the Home
Assistant host. New options to switch on: `strip_ansi`,
`multiline_containers` (see 1.21.3.3 to 1.21.3.5 below).

## 1.21.3.5

- New option `multiline_containers`: a regex of container names whose
  Python tracebacks are joined into one entry instead of one per line
  (Home Assistant: `homeassistant|hassio_supervisor`). Empty by default.
- On the JSON path (`structured_metadata`) the line becomes the journal
  MESSAGE right after the fields are extracted, not at the end of the
  pipeline. Every stage that reads text now reads the plain MESSAGE on
  both paths. What is shipped does not change.

## 1.21.3.4

- `level_from_message` also reads a lowercase level when it is the line's
  first word, optionally behind one `program: ` prefix: `info: ...`,
  `s6-rc: info: ...`, `[warn] ...`. Every s6-overlay and bashio start-up
  line arrived as `level="error"` before. A lowercase word anywhere else
  ("disk info: low") is still not a level.

## 1.21.3.3

- New option `strip_ansi`: removes terminal colour codes from every line
  before anything else reads it. Home Assistant, the Supervisor and
  Zigbee2MQTT colour their output. Off by default.

## 1.21.3.2-local

Local build of dierochade/ha-addon-alloy (branch `local`): 1.21.3.2 plus
the AppArmor fix from sporqist/ha-addon-alloy#38 (`/etc/ssl/openssl.cnf`
for the HEALTHCHECK's curl on AppArmor 3.1.7), built on the Home Assistant
host instead of pulled from GHCR.

## 1.21.3.2

- New option `host_label`: a static `host` stream label in place of
  `hostname`, for a Grafana Loki whose other sources already name the machine
  with `host` (ecohash-co/ha-addon-alloy#13 asked for the same). Empty by
  default, so nothing changes for an existing install.

## 1.21.3.1

- AppArmor: accept stop signals from `runc`. On hosts where runc has an
  AppArmor profile of its own (AppArmor 4, such as the Ubuntu 24.04 CI
  runner) Docker's SIGTERM/SIGKILL were denied, which failed the e2e gate for
  every pull request since 2026-11-13. AppArmor 3.1.7, which Home Assistant
  OS ships, has no runc profile, so nothing changes there.

## 1.21.3

Grafana Alloy 1.21.3; the add-on itself is unchanged.
Upstream release notes: https://github.com/grafana/alloy/releases/tag/v1.21.3

## 1.21.2

Grafana Alloy 1.21.2; the add-on itself is unchanged.
Upstream release notes: https://github.com/grafana/alloy/releases/tag/v1.21.2

## 1.21.1

Grafana Alloy 1.21.1; the add-on itself is unchanged.
Upstream release notes: https://github.com/grafana/alloy/releases/tag/v1.21.1

## 1.21.0.2

_2026-10-22_

### Changed
- AppArmor: the outer profile, which covers s6-overlay and the generator
  scripts, no longer allows every file operation. It started from Home
  Assistant's template, whose blanket `file,` rule made all of its path rules
  meaningless; it now allows only what a complain-mode run of the whole test
  suite showed is used. Writes outside `/data`, `/etc/alloy`, `/tmp` and
  `/run` are denied. The Grafana Alloy process itself was already confined
  this tightly and is unchanged.

## 1.21.0.1

_2026-10-22_

### Added
- Authentication for Grafana Loki: `loki_username` and `loki_password` (basic
  auth, as Grafana Cloud and most reverse proxies want), or
  `loki_bearer_token`, and `loki_tenant_id` for a multi-tenant Loki. The
  secrets are kept in 0600 files, never in the generated config. Until now an
  authenticated Loki needed `raw_config`.

### Changed
- `loki_url` has no default any more. The old one, `localhost`, is the add-on
  itself, where no Loki answers, so a fresh install looked healthy and shipped
  nothing. A `localhost` or `127.x` URL is now refused with the reason.
  Existing installs keep the URL they have.
- Option values are escaped before they go into the config: a `"` or `\` in a
  job, instance or tenant is part of the value, not the end of a string. A
  newline in one is refused. Intervals must be a whole number of s, m or h.

### Fixed
- Every start logged `labels: command not found` at error level: a comment in
  the generated config contained backticks, which the generator ran as a
  command.

## 1.21.0

Grafana Alloy 1.21.0; the add-on itself is unchanged.
Upstream release notes: https://github.com/grafana/alloy/releases/tag/v1.21.0

## 1.20.1

Grafana Alloy 1.20.1; the add-on itself is unchanged.
Upstream release notes: https://github.com/grafana/alloy/releases/tag/v1.20.1

## 1.20.0

Grafana Alloy 1.20.0; the add-on itself is unchanged.
Upstream release notes: https://github.com/grafana/alloy/releases/tag/v1.20.0

## 1.19.2.1

_2026-09-15_

### Changed
- The add-on is called **Alloy** - the ecosystem's convention (the Loki,
  Promtail, Grafana and InfluxDB add-ons are all named after what they
  package), and a word that is not on Grafana Labs' trademark list, unlike
  "Grafana Alloy". Descriptions keep "Grafana Alloy" and "Grafana Loki" as
  adjectives for the products, with the disclaimer.
- Final artwork: the "lines out" mark - journal lines, one leaving - in rust
  on a dark tile, as icon and logo.
- Grafana Alloy 1.19.2 (the first release taken through the gate unattended:
  Renovate #2, both add-ons, zero AppArmor denials).

## 1.19.2

Grafana Alloy 1.19.2; the add-on itself is unchanged. First release that
Renovate took through the gate and merged unattended.
Upstream release notes: https://github.com/grafana/alloy/releases/tag/v1.19.2

## 1.13.2.7

_2026-09-14_

### Fixed
- The `job` label is now stamped in the processing pipeline, not on the
  journal source. Grafana Alloy 1.19.0-1.19.2 overwrite a job declared on
  `loki.source.journal` with the component id (grafana/alloy#6980; fixed
  upstream for 1.19.3), which is what made the 1.19.2 update fail the gate:
  every line arrived as `job="loki.source.journal.journal"`. No change on
  1.13.2; the fix is what lets 1.19.x updates through, and it is permanent:
  a label set by a stage is immune to source-side overrides in any version.

## 1.13.2.6

_2026-09-14_

### Added
- `raw_config` (off): expert mode. Additional Grafana Alloy configuration
  becomes the whole configuration; the generated pipeline is not written and
  every other option except the log level is ignored. For people who want
  their own sources, label names and outputs without a knob per label here.
  Still validated before start; the gate ships a line through a user-written
  pipeline with its own label names.

## 1.13.2.5

_2026-09-14_

### Changed
- Every optional option now appears in the configuration form: the form
  hides optional keys that are absent from the add-on's options, so they are
  all listed with an empty value. An empty value means "use the default".
  The two metrics URLs are typed as plain strings for that reason (an empty
  string is not a valid url) and checked by the add-on at start instead.
- Friendly names and descriptions for every option (translations), with the
  stream-label cardinality warning where it is needed.
- Documentation names the products as "Grafana Alloy" and "Grafana Loki"
  and carries the trademark disclaimer; a generic icon and logo.

## 1.13.2.4

_2026-09-14_

### Fixed
- `level_from_message` read the level from the whole journal entry, not the
  message: the JSON path was always on and the message was only restored at
  the end of the pipeline, after the level stage. A level token in any other
  field (a container tag, a command line) could set the level. The level
  stage now reads the message explicitly when the entry is JSON.
- With `structured_metadata` on, `level_from_message` never corrected a
  level at all: the metadata stage removes a stream label of the same name
  when it moves a field into metadata, and it ran before the level stage,
  taking the temporary `container_name` label the level stage selects on.
  The level stage now runs first. The same behaviour is why a field chosen
  as a label was lost when also sent as metadata (fixed in 1.13.2.1 for the
  wrong stated reason - it was this stage, not Loki).

### Changed
- The JSON path (`format_as_json`, `stage.json`, the message restore) is
  used only when `structured_metadata` is on - the only option that needs
  the extra fields. The default install no longer parses and restores every
  line for nothing.
- CI: level_from_message is asserted on both paths with real container-style
  journal entries: `level=warn` in the message becomes `warning`, a bare
  `INFO:` becomes `info`, and a level token in a non-message field is
  ignored.

## 1.13.2.3

_2026-09-14_

### Added
- `metrics_instance`: the `instance` label on metrics (default: the
  container's hostname, which is not the Home Assistant OS hostname).
- Support for the `alloy-host` variant: an image built FROM this one with a
  marker file gets host metrics from `prometheus.exporter.unix`. Nothing
  changes for this add-on.

### Fixed
- The scrape timeout derivation is one function for every scrape.

## 1.13.2.2

_2026-09-14_

### Added
- Optional metrics: `metrics_enabled` scrapes one OpenMetrics endpoint
  (default Home Assistant's `/api/prometheus`, bearer token from
  `metrics_token`) and remote-writes it (`metrics_remote_write_url`, with
  optional basic auth for hosted Prometheus). Off by default; when off the
  generated config is unchanged and no credential is kept on disk.
- The scrape timeout is derived from the interval, because Alloy refuses a
  timeout above the interval and `alloy validate` cannot see that.
- CI: a third gate run scrapes a stub that answers only to the exact token,
  through remote_write into a real Prometheus.

## 1.13.2.1

_2026-09-14_

Faithful, minimal defaults; every transformation opt-in. Made for anyone's
Loki, not one fleet's.

### Changed
- Default stream labels are now `hostname`, `unit`, `level` only.
  `syslog_identifier`, `container_name` and `transport` are no longer labels
  by default - each one multiplied streams in the receiving Loki. Add them
  back with `stream_labels` if you relied on them.
- `job` is an option (default `systemd-journal`, unchanged).

### Added
- `stream_labels`: choose which journal fields become stream labels
  (`hostname`, `unit`, `level`, `syslog_identifier`, `container_name`,
  `transport`, `priority`).
- `structured_metadata` (off): carry the detail fields as Loki structured
  metadata instead of labels. Needs Loki 2.9+ with it enabled.
- `level_from_message` (off): re-derive `level` from the line for container
  streams that docker logged at priority `err` (its stderr), so an add-on's
  own `WARN`/`INFO` lines stop arriving as errors.
- Alloy usage reporting is off (`--disable-reporting`).
- CI: the gate now runs the image twice under the AppArmor profile - default
  options and a fully opinionated set - and asserts the exact label set and
  structured-metadata placement in Loki.

## 1.13.2

_2026-09-14_

Fork of ecohash-co/ha-addon-alloy 1.0.0, hardened for unattended use.

### Changed
- Grafana Alloy 1.13.2, downloaded from Grafana's GitHub release and verified
  against the published `SHA256SUMS` at build time. A mismatch fails the build.
- Pre-built images on `ghcr.io/sporqist/ha-addon-alloy`, built by CI with the
  `home-assistant/builder` actions. The Home Assistant host pulls; it no
  longer compiles the add-on locally.
- Dockerfile is the single source of truth: pinned multi-arch base
  `ghcr.io/home-assistant/base-debian:trixie-2026.08.0`, `build.yaml` removed
  (Supervisor stopped supplying `BUILD_FROM` in 2026.04.0).
- `libsystemd0` installed explicitly. Alloy's journal reader loads it at
  runtime; upstream relied on the base image happening to carry it.
- The debug UI port `12345` is no longer published on the host by default.
  The Supervisor watchdog still reaches it on the add-on network.
- `addon_config` mapping dropped: nothing in the add-on used it.

### Added
- AppArmor enabled with a custom `apparmor.txt`. The Alloy sub-profile ships
  in `complain` mode for this release so that missing permissions are logged
  rather than denied; the next release removes the flag once the audit log has
  been read on a real host.
- Renovate configuration tracking Alloy releases (bumps `ALLOY_VERSION` and
  the add-on `version` together), the base image tag, and the GitHub Actions.
- Lint workflow using Home Assistant's add-on linter, nightly.

## 1.0.0

_2026-02-21 - upstream release by ecohash-co_

### Added
- Initial release by ecohash-co
- Grafana Alloy v1.13.1
- Systemd journal log shipping to Loki
- Journal field relabeling (unit, hostname, syslog_identifier, transport, container_name, level)
- Debug UI on port 12345
- Configurable Loki URL, log level, and additional config
- Watchdog health check via Alloy's `/-/ready` endpoint
- Support for amd64 and aarch64 architectures
