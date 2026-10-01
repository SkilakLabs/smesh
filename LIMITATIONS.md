# Known limitations

Skilak Mesh scans requests before transmission and fails closed when required
request content cannot be scanned. HTTP provider responses are relayed
unmodified and unscanned. MCP tool replies are inspected separately by the
mediator. Policy verdicts are `allow`, `warn`, `redact`, and `block`.

These are source behavior boundaries. See [release availability](README.md#availability)
before installing: source tests or a prepared native build do
not qualify a public package, exact client, normal installed lifecycle, or
prevention of alternate egress. Browser and desktop prototypes remain synthetic
pilots.

- **Gemini coverage is a bounded API-key subset.** Native v1beta generation and
  streaming are supported; Google sign-in, Vertex, Live, provider-held files,
  cache, signatures, and native body redaction remain refused or unqualified.
- **MCP coverage is POSIX stdio mediation.** Arguments are checked before tool
  invocation and returned content before delivery. Windows, HTTP MCP and opaque
  resources remain outside that subset. It does not contain arbitrary tool-server
  filesystem or network activity.

## Detection tradeoffs

- **Classification labels are read from the file, not inferred.** A Microsoft
  sensitivity label is detected by its properties, so it survives renaming and
  translation — but a document that was never labelled carries no signal, and a
  label removed before sending is gone. `Personal`/`Public`/`General` are
  treated as not sensitive; a company that reuses one of those names for
  sensitive content must override it. A confidentiality notice must be a
  marking — a scoped phrase or a document stamp. Prose that merely discusses
  confidentiality is deliberately not a finding, which means a notice written
  in an unusual form can be missed.

- **Keyword-less single-charset secrets.** The keyword-less fallback catches a
  mixed-case+digit candidate at 32+ characters, a single-case alphanumeric
  candidate at 32+ characters when entropy is at least 4.6, a 40+ candidate at
  the lower 4.2 threshold, and a 40+ digit-free mixed-case alphabetic candidate
  at the stricter 4.6 threshold. Common digest-length pure hexadecimal,
  single-case alphabetic strings, and shorter values with no known prefix or
  adjacent keyword remain ambiguous; the same value in a keyworded assignment
  (`token=…`) blocks. Long (128+ character) hex that decodes to non-text binary
  instead fails closed as unscannable, except for the narrow exact short-period
  generated-table carve-out described below. Catching every bare hex value would
  false-positive on every git SHA/checksum in a diff. A bare 32–39 character
  single-case token just below the 4.6 entropy floor, or a 32–39 character
  digitless mixed-case token, can still be missed without a keyword/prefix;
  lowering those floors would broaden the already noisy fallback.
- **`high_entropy_secret` is the noisy rule.** It can false-positive on opaque
  high-entropy identifiers, malformed certificate-like/base64 bodies, SRI/integrity
  digests, request/trace IDs, ETags, CSP nonces, data-URI payloads, and
  inventory/SKU lists whose separator-stripped join is secret-shaped.
  Prefix/context-only exemptions for these public forms are
  attacker-gameable, so the strict preset blocks them; structurally validated
  X.509 certificate payloads, SSH public keys, and ETH addresses are exempt.
  `config/skilak-balanced.yaml` demotes this fallback to `warn`; recognized
  credential formats in its reviewed ordinary conversation-text carriers are
  redacted and fully rescanned, while unsupported carriers continue to block.
- **Keyworded opaque values intentionally over-block.** After an explicit
  secret-family keyword (`api_key`, `access_key`, `secret`, `client_secret`,
  `password`, `token`, `credential`, and spelling variants) plus `:`/`=`, any
  opaque value blocks regardless of length, case, digits, or entropy. Exemptions
  are limited to readable lowercase word identifiers joined by `_`/`-`, narrow
  dotted/numeric-suffix business identifiers, restrained camel identifiers,
  bounded model-repository paths, paths/URLs/function calls, documented
  placeholders, masked repeats, all-digit IDs/timestamps/durations, and short
  (<8) low-entropy values. Thus
  pagination/CSRF tokens and opaque benign config values can block; this is the
  approved fail-closed default and is policy-relievable at the
  `generic_api_key` rule. A real credential deliberately shaped exactly like a
  readable identifier/path/placeholder can still evade this keyword rule;
  known prefixes and the entropy fallback apply independently.
  Keyword matching intentionally remains permissive inside compound field
  names, so benign fields such as `tokenExpiresAt`, `password_changed_at`,
  `secretary`, and `tokenIssuerHost` can hard-block; narrowing that bridge risks
  missing common env/camel credential names and is not attempted in v1.
  Exact `password`/`passphrase`/`secret`/`client_secret`/`api_secret` labels
  additionally treat three-or-more lowercase kebab/underscore words as a
  diceware passphrase. A benign readable identifier in one of those exact
  fields can therefore over-block; function calls and identifier field names
  such as `secret_name` remain exempt.
- **Bare `key` and very short low-entropy credentials remain ambiguous.** A
  standalone `key` label is not treated as a secret-family keyword (it would
  flood on `primary_key`, `sort_key`, and prose). A <8-character low-entropy
  value is exempt on generic token/key/secret fields, so such a token can be
  missed unless another fixed-format rule catches it. Password-family fields
  separately catch 6–7-character values with a digit/symbol and at least three
  distinct characters. A lone low-entropy alphabetic status word such as
  `undefined`, `required`, or `incorrect` is treated as prose rather than an
  opaque credential; a real credential deliberately shaped as one such word
  can therefore be missed.
- **Long alphabetic password values err closed.** A 16+ character letters-only
  value behind a password/passphrase field is treated as a credential, covering
  concatenated word passphrases. A long code identifier in that exact field
  position can therefore be blocked; shorter alphabetic identifiers and common
  function/path placeholders remain exempt.
- **Encoding / deliberate fragmentation.** Adjacent split text parts,
  zero-width splices, distinctive-format separator splits (`AKIA-IOSFO…`),
  custom terms with bounded punctuation between alphanumerics, and token-like
  entropy candidates split by up to 64 common ASCII/Unicode whitespace or
  punctuation characters are reassembled and caught. The entropy variant
  requires token chunks of at least eight characters in its broad fragment
  path; keyworded fragments use the stronger keyword context. Three-or-more
  short fragments are exempt only when their join is not itself secret-shaped.
  Consequently unrelated high-entropy identifiers or SKU codes can over-block
  when their concatenation looks like a secret; a caller-controlled `SKU`
  label cannot suppress that result. Provider-bound metadata sibling values are
  likewise joined without separators, so unrelated short metadata values whose
  concatenation is secret-shaped can block. Real leaves and parser-produced
  reassemblies are scanned with the full rule set. Manufactured request-wide
  joins use only the established distinctive rules, `generic_api_key`, and
  configured custom terms; the entropy and same-leaf contextual lexicon rules
  do not run on invented adjacency. Each adjacent provider-order boundary
  involving extracted file text additionally receives one bounded suffix-prefix
  seam whose width is derived from finite per-rule/custom-term witnesses.
  Nonadjacent alternatives, opposite-order variants not present in the payload,
  and leaf-by-archive-member cross-products are not manufactured; exhausting
  retained seam work blocks as `straddle_budget_exceeded`. Joining unrelated
  turns can still over-block when their boundary forms a distinctive sensitive
  shape; this is the confidentiality-first tradeoff. Base64, base32, RFC1924 base85, Adobe
  ASCII85, ZeroMQ Z85, hex, and percent-encoded text are decoded independently
  up to two layers and rescanned. Two layers is the ceiling: a value encoded
  three or more times is not recovered. Percent-decoding triggers on three or
  more `%xx` escapes anywhere in a source rather than three adjacent ones —
  requiring adjacency previously meant a double-encoded value (`%2541%254B…`,
  where each escape is separated by literal hex digits) was never decoded at
  all, and so fell short of the two layers documented here. Standard base64 is also reconstructed across
  bounded MIME whitespace folds, including one-character, uneven quantum, and
  short irregular folds. A bounded two-ended search separates the encoded value
  from adjacent ordinary prose; overlapping decoded interpretations count once
  per rule. Syntactic subchain work has a 65,536-interpretation/deadline ceiling
  and successful decodes share the request candidate/size budget. Reaching a
  ceiling blocks rather than silently abandoning the decode. Not caught:
  arbitrary encryption, rot13, reversal, password-protected encoding, or
  custom codecs. Hex candidates of at least 128 characters that decode to
  non-UTF-8/control-byte binary emit the non-downgradable
  `encoded_binary_unscannable` rule unless the decoded bytes are an exact
  repetition with a 1–8-byte period and do not start with known file/container
  magic. A one-byte mutation, a longer period, or known magic stays fail-closed.
  Base64 candidates that decode to known binary/container magic also emit the
  non-downgradable
  `encoded_binary_unscannable` rule rather than being silently skipped. This
  can block public binary embedded in a text leaf; use a recognized file field
  when it must be structurally inspected.
  Inherent to any regex DLP; targets accidental leaks, not a determined insider
  with an encoder.
- **Bare digest-length hex / single-case alphabetic secrets.** A bare 32/40/64-
  character all-lowercase-hex or single-case alphabetic secret with **no keyword and no
  known prefix** (a bare Twilio
  auth token, Datadog API key, Azure APIM subscription key value, or raw
  hex-encoded HMAC key) is not caught by the entropy fallback. Prefix-
  identifiable formats **are** caught by
  dedicated rules (Databricks `dapi…`, DigitalOcean `dop_v1_…`, Twilio `SK…`/
  `AC…`, Shopify, Mailgun, Mailchimp, …), and the same secret in any keyworded
  form (`auth_token=…`, `Ocp-Apim-Subscription-Key:…`) is caught (incl.
  UUID-shaped values, which bypass the entropy floor when keyword-gated).
  Dedicated keyword rules now cover lowercase-hex Datadog API/application keys,
  Rails `SECRET_KEY_BASE`, and Twilio auth tokens in `TWILIO_AUTH_TOKEN=...` or
  `AC<sid>:<token>` context; their truly bare hex values remain ambiguous.
  Widening the fallback to all bare hex would false-positive on every commit
  SHA/checksum in a diff. Hex-encoded binary at 128+ characters normally fails
  closed as unscannable; the exact short-period generated-table carve-out above
  and shorter/digest-length ambiguity remain.
- **Encoded hits are block-only.** Base64/base32/base85/ascii85/Z85/hex/percent
  decoding is bounded and
  common text encodings are caught, including short encoded PII; bounded
  non-text binary representations fail closed as described above. The gateway cannot
  safely map decoded offsets back to a unique source span, so an encoded PII or
  custom-term hit blocks even when its configured action is redact. Rot13,
  reversal, compression embedded in ordinary text, encryption, and custom
  codecs remain outside detection. A determined process with encryption and an
  allowed destination can defeat content DLP; network and filesystem controls
  are therefore mandatory, not optional defense-in-depth.
- **Complete JWTs are governed by the specific `jwt` rule.** A literal,
  complete JWT has a directly maskable span. Balanced redacts it only in a
  reviewed ordinary conversation-text carrier and discards an entropy fallback
  wholly contained by that span; Strict blocks it. Entropy beside the JWT
  remains independent, and a JWT found in an unsupported carrier or only after
  reversible decoding is block-only because the source span cannot be safely
  rewritten.
- **Encoding resource bounds can over-block.** Each source/leaf is limited to
  32M input characters and 1M decoded characters; the complete request is
  limited to 65,536 candidates shared across every supported encoding alphabet
  and two decode layers.
  Reaching a limit emits `encoding_scan_overflow` and blocks; candidates are
  never silently truncated
  and allowed. The budget is derived from the content ceiling rather than
  chosen: content large enough to pass `MAX_EXTRACTED_CHARS` yields at most
  ~50,000 candidates, so ordinary traffic is nowhere near it (prose 1–3%, source
  3%, a hash-dense lockfile 10%). Lockfiles up to ~800 KB scan completely; past
  that the size limit refuses them as `file_oversized`. Deliberately dense
  base64 still reaches the limit and blocks. Re-derive on your own traffic with
  `scripts/measure-encoding-budget.py`.
- **Fullwidth / NFKC-confusable characters** in a key are not normalized before
  scanning, so a fullwidth-rendered key evades. NFKC normalization is a later
  enhancement.
- **Pasted SRI / base64 digests are blocked.** A base64 subresource-integrity
  hash (`integrity="sha256-<mixed-case base64>"`) now trips the entropy
  fallback: the digest-prefix exemption was removed because `sha256-<crafted
  secret>` was attacker-gameable (a fail-open). Fail-closed FP; G2 policy relief.
  Lowercase-hex digests (docker/git `sha256:…`) are unaffected.
- **Header-less private-key body.** A PEM key pasted without its
  `-----BEGIN … KEY-----` header (bare base64 body) is not flagged; the headered
  form is.
- **Documentation placeholders require a whole-value shape.** Recognized values
  such as `your-key-here`, `your_api_key_here_1234`, `${DB_PASSWORD}`, and Slack's
  `xoxb-your-bot-token` remain exempt. Marker words such as `example`, `dummy`,
  or `redacted` inside an otherwise live credential do not suppress scanning.
  An unlisted documentation placeholder can therefore be blocked.
- **Confidential custom terms use substring semantics.** A configured term is
  caught inside a longer word and across up to 256 non-alphanumeric separator
  characters between each adjacent pair of its alphanumerics, so
  prepending/appending a word character cannot bypass an explicit block term.
  This can flag a short term inside an unrelated longer word. NFKC plus a
  bounded single-code-point Cyrillic, Greek, Armenian, Cherokee, and Coptic
  table covers common readable homoglyphs, but it is not the complete Unicode
  UTS-39 confusables database; an unlisted look-alike can remain.
- **Fallback suppressions are structurally bounded.** The keyless entropy
  fallback's ordinary candidate exemptions require a *genuine* digest
  (`sha256-<exact-64-hex-or-44-base64>`, verified length — an arbitrary-length
  secret with `sha256-` glued on is NOT exempt), an ETH address, or a real
  path/URL/model reference. Findings wholly contained by the separately
  validated public-certificate/public-key forms described below are also
  suppressed. The path exemption concatenates the value's
  non-word `/`-segments and refuses to exempt if that concatenation is itself a
  ≥32-char high-entropy secret — so `<secret>/word`, `word/<secret>`,
  `/<secret>`, and a secret chopped into sub-32 `/`-pieces are all still caught.
  Residual FP (fail-closed): a genuine share-link/model-ref whose non-word
  remainder happens to be ≥32 high-entropy chars is blocked (G2 policy relief).
- **Public-key exemptions are structural, not trust decisions.** X.509
  certificate PEM bodies are exempt only after a bounded DER reader validates
  the complete certificate envelope and mandatory public-certificate spine.
  This does not verify the signature, chain, hostname, revocation, or trust;
  those properties are irrelevant to whether the payload is public. Malformed
  certificate fences, arbitrary wrapped base64, and all private-key blocks stay
  detectable/fail-closed. Valid SSH RSA, Ed25519, and ECDSA public-key wire
  blobs are also structurally decoded and exempted; unsupported SSH public-key
  algorithms may still block.
- **Share links with a long mixed-case id may be blocked.** A Google Drive/Docs
  URL whose file id is ≥32 mixed-case high-entropy chars trips
  `high_entropy_secret` — a file id is structurally identical to a secret, and
  the path exemption deliberately does NOT exempt any value containing a
  standalone ≥32-char secret segment (that exemption was an attacker bypass:
  `<secret>/word`). Permalinks with an all-lowercase-hex SHA (GitHub blob URLs)
  are unaffected. Fail-closed FP; G2 per-rule policy is the relief.
- **Contiguous all-caps prose can false-positive as an AWS key ID.** The aws_access_key_id
  rule is unconditional (no digit-in-tail requirement, no `(?<![A-Z])`
  lookbehind — both were precision tweaks that opened fail-opens: digit-free
  real keys were missed, and an uppercase-glued key `QAKIA…` slipped). The cost
  is a narrow FP: a contiguous ALL-CAPS run that literally forms
  `AKIA`/`ASIA`/`ABIA`/`ACCA` + 16 uppercase characters is blocked. The
  punctuation/space-collapsed pass now requires a digit in the 16-character
  body, so ordinary multi-word ASIA/ABIA/ACCA prose no longer manufactures a
  hit. Mixed-case prose is unaffected. Strict and unsupported carriers block;
  Balanced redacts a mappable occurrence in ordinary conversation text and
  fully rescans the provider-bound request.
- **AWS reserved documentation credentials are still treated as credentials.** The canonical
  `AKIAIOSFODNN7EXAMPLE` / `wJalrXUtnFEMI…EXAMPLEKEY` docs pair is treated as a
  real hit because the gateway cannot safely assume any AKIA-shaped token is
  the non-functional example. Strict blocks it. Balanced replaces it only in a
  supported ordinary conversation-text field, then scans the complete rewritten
  body before forwarding; files, tool/schema structure, encoded/joined-only
  forms, and failed rewrites still block.
- **Image recognition is not a proof against steganography.** Images are parsed
  in memory, with declared dimensions budgeted before raster loading; raw
  ASCII/UTF-16 strings and metadata are
  scanned, visible text is OCR'd, and QR/barcode payloads are decoded (including
  barcodes made from PDF vector primitives after page rendering). OCR can still
  miss handwriting, stylized/low-contrast/adversarial text; arbitrary pixel
  steganography cannot be proven absent. Barcode tooling is required by
  default; missing `libzbar` blocks as `file_processor_unavailable`. An
  operator may explicitly set `SKILAK_REQUIRE_BARCODE_SCANNER=0`, but that is a
  lower-assurance fail-open for QR/barcode-only content. Set
  `SKILAK_DENY_ALL_IMAGES=1` (or
  `SKILAK_DENY_ALL_FILES=1`) for the high-assurance posture.
- **Opaque pagination cursors are blocked.** An opaque value of eight or more
  characters behind a `page_token`/`next_token`/`sync_token` field is flagged
  without an entropy gate. The
  deny-marker suppression that used to exempt them was removed: a caller
  controls JSON key names, so `{"sync_token": "<secret>"}` could suppress
  detection — an attacker-triggerable fail-open. Blocking the cursor is the
  fail-closed choice; G2 per-rule policy is the relief. (≥32-char cursors were
  always blocked by the entropy fallback.) This includes otherwise legitimate
  `nextPageToken`, `page_token`, `continuation_token`, and `sync_token` values.
- **Admission is representation-based.** The old normalized-route allowlist was
  a schema-era proxy for shapes the mesh could enumerate. It remains only as a
  known-route behavior table; route names no longer decide admission. An
  unknown path is forwardable with no body and no unsupported declaration, or
  when every body byte is enumerable by a registered JSON, raw-media, or
  multipart parser. Unsupported declared content types, parser failures, and
  multipart bodies with even one unenumerable part refuse. Unknown-route path
  segments and query strings are scanned as immutable string leaves before
  forwarding.
- **Provider storage policy is a named-prefix catalog.** `files` is enabled by
  default; `assistants`, `batches`, `containers`, `conversations`,
  `fine_tuning`, `threads`, `uploads`, and `vector_stores` default off. An
  operator may opt in each category with `provider_storage.<prefix>: true`, and
  request scanning still applies. A fixed catalog cannot classify a future
  provider storage name, so a new surface may follow unknown-route admission
  until it is reviewed. Unknown config names fail policy loading closed; they
  do not extend the catalog.
- **Provider responses are not inspected.** The mesh scans and enforces on the
  outbound request, then relays the provider's reply to the client unmodified,
  including SSE (which streams token by token). Response bodies, response
  headers, and downloaded file content are all outside the detection boundary:
  no rule runs against them, and nothing in a response can produce a finding, a
  block, or a redaction. Response redaction is not implemented and is not
  planned — the mesh does not rewrite model output. The `provider_response`
  audit event is metadata only (`status_code`, `latency_ms`, a `response_bytes`
  count, `stream_completed`, `"scanned": false`); `_audit_response_event` takes
  `response_bytes` as an `int` and builds a closed literal dict, so the record
  is structurally incapable of carrying response content. A successful
  `POST /files` is the buffering exception: its bounded response metadata is
  parsed only to mint one file-ID attestation, not scanned for content. The
  practical consequence for a threat model is unchanged: anything the provider
  already holds can come back to the client, and the mesh will not stop it. The
  mesh's claim is about what leaves the machine.
- **Unmaskable redact-policy PII blocks instead of forwarding.** PII found only
  after normalization or content-part reassembly, or inside a dict key/bytes
  leaf, cannot always be mapped safely back to one mutable string span. The
  mesh re-scans after redaction and emits `redaction_failure` + blocks if any
  redact-category finding remains. This is deliberately fail-closed rather
  than forwarding an incompletely redacted request.
- **Redaction rewrites JSON string values, and nothing else.** At the gateway a
  `redact` action masks findings in the string leaves of a JSON request body.
  Two positions are deliberately never rewritten, and each blocks instead
  because the re-scan still finds the value: in a **dict key**, because renaming
  a field changes what the request means rather than redacting a value; and in a
  **non-JSON body** (multipart, raw), which has no string leaves to replace and
  whose framing would no longer describe it. A JSON **number** is likewise not
  rewritten — masking it would change the value's type — so a redact-configured
  finding in one blocks. The audit records `redaction_failure` with the reason
  in `fallback` (`redaction_incomplete`, `redaction_unsupported_body`,
  `redaction_error`) and the surviving rules in `residual_rules`.
- **Attachments are rewritten only where a placeholder is grammatical.** Text-
  shaped inline attachments (plain text, CSV/TSV, Markdown, logs) are decoded,
  masked and re-encoded; JSON attachments mask string leaves only. An **.xlsx**
  is masked in the one position a workbook has for a string — the character data
  of a `<t>` in the shared-string table or in an inline-string cell. Masking a
  shared-string entry masks every cell referencing it, which is what those cells
  all held. Everything else in a workbook declines the WHOLE package, so the
  re-scan blocks it: a value stored as a **number** (masking it would retype the
  cell and change what formulas over it compute), a value in a **formula** or a
  cached formula result, one typed across two **runs**, one in a part **name**
  or attribute, and any package carrying an **image or other binary part**,
  which can hold the value in a form no text mask reaches. Other containers
  (zip, PDF, DOCX, PPTX) are not rewritten at all and block.
- **Custom terms are bounded literal matching, not semantic classification.**
  Terms use escaped, case-aware **substring** matching after the same invisible-
  character normalization and content-part reassembly as secrets — see the
  substring-semantics entry above. A second bounded matcher permits up to 256
  non-alphanumeric separators between each adjacent pair of a term's
  alphanumerics, and a small explicit Latin-lookalike
  Cyrillic/Greek/Armenian/Cherokee/Coptic skeleton map covers common readable
  homoglyph substitutions.
  This is not a complete Unicode confusables implementation. Terms do not match
  paraphrases or translations,
  but common encoded text and OCR/barcode output are rescanned.
  Configure each desired spelling explicitly.
- **Custom-term capacity and flood guard.** A policy accepts at most 256 terms,
  2–128 characters each and 16,384 characters total. More than 64 hits for one
  term in one scan pass emits `custom_term_scan_overflow` and blocks regardless
  of policy. This bounds finding-list growth; a prompt that deliberately repeats
  an allowed term may therefore over-block.

## Coverage boundaries

- **A launch wrapper proves a path, not all client egress.** `smesh run` can
  prove that requests reached its sessionized mesh URL while the foreground
  command was active. A positive count does not prove that the launched process
  originated every counted request or that the client made no direct calls.
  Environment-scrubbing tools, config-file endpoints, alternate backends, and
  hard-coded URLs can ignore the injected base URLs. An exact zero produces a
  warning; a gateway restart, lost finish, or saturated counter is reported as
  unavailable/incomplete instead of fabricating zero. Claude Code's normal
  base-URL mode has verified client inventory. Cursor never has complete client
  coverage because indexing and tab completion use Cursor's own backend.
  Session observation covers the foreground process lifetime and requires the
  shipped single gateway worker; detached descendants are refused after the
  session closes. Managed egress is still required to rule out direct paths.

- **Codex WebSocket support is deliberately narrow.** The default
  `realtime.mode: inspect-text` admits only JSON text `response.create`
  messages on the exact OpenAI `/v1/responses` transport and observed
  `responses_websockets=2026-02-06` beta. Every client message uses the same
  request inspection and policy path as HTTP before forward. Provider events
  are structurally validated, but their text is not content-inspected, matching
  the HTTP response boundary. Binary, audio, opaque, malformed, wrong-route,
  and over-limit traffic fails closed; `block-upgrade` remains selectable and
  `pass-unscanned` is prohibited. Defaults bound the transport to 16 concurrent
  sockets, 8 MiB per message, 128 MiB and 4,096 messages per session, four
  hours per socket, and 1 MiB per upstream fragment. These bounds can refuse a
  legitimate unusually large or long Codex session.

- **A full-access coding agent is trusted with the local machine.** This
  maintainer checkout gives Claude and Codex the same local filesystem,
  process, and network capability. Repository instructions can constrain the
  task, but they are not an isolation boundary and cannot prevent a compromised
  dependency or untrusted checkout from abusing that access. Review untrusted
  code only in a sanitized disposable VM/container or separate OS account with
  no host-home, cloud, password-manager, or container-runtime mounts.

- **`make agent-test` provides credential hygiene, not isolation.** It runs the
  provider-free unit suite with an empty inherited environment, but repository
  code still executes with the caller's filesystem and network rights. Run it
  only from a reviewed checkout; use an externally isolated environment for
  untrusted code.

- **Supported does not mean every file format is accepted.** Text/source,
  PDF, common raster images, modern Office Open XML, ZIP/TAR/GZIP, and MIME
  email are processed. Supported Office documents nested inside archives are
  recursively scanned under the same file, byte, entry, and depth budgets.
  Legacy Office/OLE, executables, audio/video, encrypted
  documents/archives, macros/ActiveX, remote/active document relationships,
  malformed content, unknown binary entries, and unsupported codecs block.
  Blocking an unsupported type is protection; it is not content extraction.
  Non-JSON raw request bodies are always magic-inspected regardless of their
  declared MIME type, and plausible ZIP local headers missing from the central
  directory make the entire archive unscannable rather than forwarding hidden
  bytes.
- **Remote file URLs are denied.** The mesh does not fetch provider-bound URLs,
  preventing SSRF, DNS rebinding, redirect, and time-of-check/time-of-use gaps.
  Supply the content inline in an enumerable request representation instead.
  Opaque provider file IDs are accepted only with a local, non-expired
  attestation tied to the same provider alias, current policy, scanner settings,
  model identity, and complete enforcement revision. This applies to recognized
  provider-bound JSON file-ID fields; direct file API operations are not
  use-producing references. A clean or explicitly waived `POST /files` can mint
  one from a bounded, valid provider response. Relevant source/configuration or
  waiver-sidecar changes invalidate existing IDs; waived approvals also end at
  the earliest grant expiry. This availability cost prevents an old approval
  from surviving a stronger scanner build or an expired waiver.
- **Finite resource limits can over-block legitimate large files.** Defaults:
  20 MB/file, 50 MB expanded, 1024 files, 1024 archive entries, depth 3, 50 PDF
  pages/image frames, 60M pixels, 1M extracted characters **per source**, 64M
  retained across the request, 45 seconds of processing, 20 seconds of scanning
  and 40 seconds of redaction — the last two shared across the whole request,
  not per source. Measured, redaction costs about what the scan costs and a
  5,000-row export finishes in well under a second, so the ceilings are far
  above real traffic and exist to bound the pathological case. Exceeding any bound blocks, and
  the refusal names the budget it exhausted with the value it measured.
  **A container counts as a file beside its own members**, so a 1024-member
  archive is 1025 files and refuses; 1023 members is the practical ceiling. ZIP, gzip, bz2, xz, and tar paths all consume the
  shared byte/entry/depth budget. ZIP archive/entry comments and both local and
  central extra fields, plus GZIP FNAME/FCOMMENT/FEXTRA headers, count toward
  the extracted-text budget and are scanned before forwarding. Binary
  ASCII/UTF-16 extraction also blocks at 200K characters instead of silently
  truncating. Untrusted pypdf/pdfium/Pillow work runs in a separately killable
  process under the same wall-clock deadline; OCR/barcode subprocesses retain
  their own hard timeouts. Process startup and serialization add latency to
  PDF/image requests, deliberately trading throughput for containment.

- **Build evidence does not prove publication.** A passing platform lane,
  attached candidate, signature, or private draft does not prove that a public
  release is complete or downloadable. Verify public downloads, installed
  versions, and checksums before making an availability claim.

- **Linux native containment is a parser boundary, not a machine sandbox.** The
  x86_64 artifact names its boundary `linux-landlock-seccomp` only after a live
  Landlock ABI 4+ and seccomp canary proves denied socket creation, denied
  outside read/write/execute, owner-only scratch access, no-new-privileges,
  non-dumpable state, hard resource limits, and bundled OCR/barcode execution.
  It refuses root, capabilities, inherited worker sockets, unsupported kernels,
  and failed canaries. It does not create PID, user, or mount namespaces or a
  cgroup and has no sustained fork cap; scanner helpers remain in the host
  PID/user namespace. A compromised worker can signal or resource-pressure
  other same-UID processes. Only the parser workers—not the network-facing
  gateway—use this boundary. Use a dedicated account or stronger host isolation
  when that residual is not acceptable.

- **Windows native containment is a bounded parser process, not a filesystem or
  network sandbox.** The x86_64 artifact names its boundary
  `windows-low-integrity-privilege-filtered-job` only after a subprocess canary
  proves zero restricting SIDs, no held privilege beyond
  `SeChangeNotifyPrivilege`, Low integrity, Job Object membership,
  kill-on-close, a maximum of eight processes, 1 GiB per process, 1536 MiB for
  the job, 50 seconds of process user time, denial of a ninth active process,
  denial of a write to an explicitly Medium-integrity target, a designated
  Low-integrity scratch write, and execution of the bundled OCR and barcode
  tools. It is not AppContainer and has no secondary restricting-SID access
  check. The measured boundary does not deny reads of every file the user can
  read or prove that a compromised parser cannot create a network connection.
  Only parser workers, not the network-facing gateway, use it. Use a dedicated
  account or VM when those residuals are unacceptable.

- **The native Linux binary has an Ubuntu 24.04/glibc 2.39 baseline.** The
  installer refuses musl and older glibc hosts. Those systems, Linux ARM64, and
  any distribution not proven by the artifact smoke gate use the Docker-backed
  CLI path.

- **The native Windows binary is x86_64-only.** Windows ARM64 and Windows
  versions not exercised by the release runner remain outside the artifact
  contract and use the Docker-backed CLI path. A workflow definition, source
  tree, or passing non-Windows test does not prove a usable Windows artifact.

- **The native Windows managed lifecycle is single-user per host.** The beta's
  fixed Task Scheduler name and loopback port are machine-global even though
  the task runs as its owning user. Competing managed installs under multiple
  logged-in users are unsupported; use one dedicated account or VM.

- **Connection-string documentation passwords are exact.** A small exact list
  (`password`, `changeme`, `${DB_PASSWORD}`, and similar whole-value template
  forms) is treated as documentation. Marker substrings inside a live password
  no longer exempt it; an unlisted documentation value can be blocked.
- **Oversized bodies** (>1 MB of scannable content) are **blocked**, not
  partially scanned by default. An operator may explicitly select G3
  `failure_mode.oversized: fail_open`, which forwards with a `warn` verdict and
  `fail_open:true`. The gateway refuses to start with it set. The retained bounded prefix still receives ordinary,
  within-part, cross-turn, and prompt/file reassembly scans, but content beyond
  the cap is necessarily unscanned; this setting therefore trades
  confidentiality for availability and must be treated as partially unscanned
  traffic. A request-wide fragmentation join that cannot cover all of its
  provider-bound pieces is different: it emits the non-downgradable
  `fragment_reassembly_incomplete` rule, so a long benign prefix cannot consume
  the join budget and hide a split secret even when `oversized: fail_open` is
  selected. Tool/schema values and keys are joined with conversation text in
  both provider-layout orders and at individual structure boundaries. Those
  boundary combinations are capped at 512 streams and 1 MB of aggregate join
  material; exceeding either cap also emits the same forced-block rule. Very
  large generated schemas can therefore over-block and should be reduced or
  reviewed before use.
- **Integer token-ID prompts are blocked, not decoded.** An OpenAI
  completions/embeddings `prompt`/`input` supplied as integer token IDs
  (`[12173, 5987, …]`, flat or batched) decodes to text provider-side but is
  opaque to a regex scanner, so any such request is **blocked** fail-closed
  rather than forwarded. Decoding with the model tokenizer to scan the real
  text is a later enhancement.

- **PII and password precision choices leave narrow gaps.** Space-grouped SSNs
  require a nearby `ssn`/`social security` label; unlabeled SSNs must be dashed.
  Dashed 3-2-4 values immediately labeled as an order, invoice, part, serial, or
  SKU are excluded to avoid destructive redaction of business identifiers.
  International phone matching requires a leading `+`. Unprefixed NANP values
  enforce area/exchange first-digit validity; bare dash/dot forms additionally
  require nearby phone context, while parenthesized forms remain distinctive.
  An unseparated `+` integer at the start of a line also requires phone context
  so unified-diff additions and epoch timestamps are not treated as E.164.
  A password candidate
  containing fewer than three distinct characters is treated as an already
  masked value, so a real repetitive password can be missed. These choices
  remove common lot-number, numeric-table, asset-name, and masked-log false
  positives without weakening known provider-secret rules.
  Credit cards require a supported Visa, Amex, Mastercard, Discover, Diners,
  or JCB IIN plus Luhn. Any run passing both checks is treated as a PAN,
  including bare and snake_case-field values; a rare Luhn-valid business ID
  using one of those IIN ranges can therefore be redacted. Cards from networks
  outside the supported IIN set can still be missed.
  IPv4-shaped values in explicit version/build/release context and IANA
  non-public ranges (private, loopback, link-local, documentation, multicast,
  reserved) are suppressed. Only public-routable addresses count as PII.
  RFC example/test/invalid/localhost email domains are likewise treated as
  documentation, not personal mailboxes.
- **Conversational credential proximity is bounded.** It recognizes singular,
  plural, possessive, and qualified password-family labels, including `pass`,
  `passwd`, `pword`, `upass`, `pwd`, `pw`, and `*_pass`, plus passphrase, login,
  credential, token, secret, and qualified key labels. Both label-before-value
  and value-before-label clauses are covered. A labelled 6–128-character value
  fails closed when it is opaque; documented placeholders and provably benign
  identifiers, paths, URLs, and function calls stay clean. Opaque components
  inside those
  wrappers still fail closed. Structured wrappers are parsed directly within a
  4-KiB bound; an over-bound or unterminated wrapper is then treated as one
  typed value only when its bounded atoms are opaque. Passphrase setter clauses
  accept bounded unquoted multiword values. A direct copula accepts a four- to
  eight-word phrase only when an alphabetic word embeds a digit or symbol;
  plain policy prose is otherwise ambiguous and stays clean. A fragment question
  and an immediately following `that`/`it` anaphoric clause may bind across one
  same-line punctuation boundary inside the existing 96-character/12-word
  window. Indirect descriptions, cross-line or other cross-sentence relations,
  other unquoted multiword values, unseen languages, and values outside that
  bound remain outside this rule unless another detector or custom term fires.
- **Identity/password pairs use a separate bounded rule.** An email address, a
  labelled username (`user`, `username`, `login`, `account`, `id`, `uname`, or
  `u`), or the tight `admin:password` form may bind to a following 6–128 character
  password through reviewed punctuation, whitespace, or connector words. The
  value needs two character classes, except that an explicit password label
  admits an eight-character value or a bounded passphrase. Trailing sentence
  punctuation on a single-token value is part of the masked span only when the
  value is already password-shaped without relying on that punctuation. The
  rule owns the complete identity/value span, so a paired email is masked even
  when singleton email aggregation would otherwise allow it. Indirect
  descriptions, paths, mail headers, source-control identities, and unlabeled
  plain words stay out.
- **Email aggregation is a shipped policy choice, not a PII classifier claim.**
  Both presets allow one real `pii_email` occurrence and redact two or more.
  Counts preserve real-leaf multiplicity and distinct mapped values recovered
  by legitimate joins/reassemblies, while payload-private fingerprints prevent
  duplicate scan views from multiplying one address into a collection. Those
  fingerprints are never emitted or persisted. The relaxation applies only
  while the global action remains the configured
  `base: redact`; an operator's global block or a stricter matching team action
  still wins, and a team allow cannot relax bulk redaction.

## Operational

- **Enterprise export is at-least-once, not exactly-once.** Each sink advances
  its own checkpoint only after a successful batch, so a crash between remote
  acknowledgement and local checkpoint can duplicate a batch. If an operator
  truncates/removes the audit before a lagging sink catches up, those events
  cannot be recovered. SIEM receivers should deduplicate using stable audit
  metadata where required.
- **The dashboard is a bounded local view, not the audit system of record.** It
  reads the newest 16 MB by default and returns aggregates only. It intentionally
  omits prompt/route hashes and raw events. The bearer token is a local pilot
  auth gate, not enterprise SSO; remote access requires a separate authenticated
  reverse proxy. Provider spend is unavailable until a trusted cost event source
  emits it.
- **Purview awareness does not call Microsoft Graph or prove label origin.** It
  evaluates configured request/header label values and trusted account mappings,
  and those values can only escalate to warn/block. A client can self-block by
  spoofing a stricter label but cannot use a label to relax policy. Use account
  mappings or an authenticated upstream label injector when label authenticity
  matters.

- **Scan concurrency is deliberately bounded.** CPU-bound scans run outside the
  policy lock on a dedicated worker pool, with at most twice the worker count
  running or queued and a default maximum of two concurrent scans per
  team/key identity. Identity-less/master-key traffic is bounded only by the
  global pool so independent parallel tool calls do not collapse into one
  shared anonymous bucket. Saturation blocks as `scan_capacity_exceeded`; it cannot
  starve policy refresh/audit threads or invoke `scan_error` fail-open, but a
  distributed sustained scan flood can still cause legitimate requests to be
  rejected. A timed-out request retains its global/per-key admission lease
  until the underlying worker actually exits, bounding the executor's queued
  payloads even if a worker stalls. PDF/image parsers have a hard process kill;
  remaining bounded text/archive work is cooperative and keeps its slot while
  completing.
- **Opaque custom-term IDs depend on policy history.** Audit records store an
  ordinal such as `custom_term_1`, not the confidential term. Reordering terms
  changes that meaning, so retain policy revisions for historical attribution.
  Folding covers NFKC and a bounded set of common Cyrillic, Greek, Armenian,
  Cherokee, Coptic, and Latin look-alikes, not full Unicode UTS-39. Add uncommon
  variants explicitly in high-assurance policies.

- **Audit/attestation key configuration.** Audit/content fingerprints are
  HMAC-SHA256; there is no bare-SHA or public-development-key fallback.
  `SKILAK_HASH_SALT` protects audit hashes and stored file attestations.
  `SKILAK_EDGE_ATTEST_KEY` remains required by the current deployment manifests,
  but the sole-edge v2 request path issues and honors no cross-process scan
  claim. It is retained configuration, not a second enforcement boundary.
  Treat audit and file-attestation stores as sensitive regardless.
- **Keyword-less token length bounds.** Bare values shorter than 32 characters
  remain outside the entropy fallback without a keyword or known prefix.
  Complete Base64/Base64url values through 32,768 characters receive a bounded
  decoded scan; only a successfully decoded and fully scanned value can replace
  the raw length-limit finding, and a clean value does not inherit an entropy
  finding from its transport encoding. Invalid or incomplete tokens longer than
  8,192 characters, and every token longer than 32,768 characters, retain the
  non-downgradable `overlong_token_unscannable` rule rather than being silently
  skipped. Large public text runs can therefore still over-block.
  Dedicated prefix rules cover reviewed formats including Hugging Face `hf_`,
  Linear `lin_api_`, New Relic `NRAK-`, and Notion `secret_`, but provider-token
  prefix recall is inherently incomplete as vendors add and change formats;
  the entropy fallback remains the generic backstop.
- **Audit fields are event-specific.** `decision` events contain pseudonymized
  model/client/team labels, advisory identity, verdict, rule hits, and hashes. Identity comes from
  `X-Skilak-*` headers or a credential fingerprint and is null when neither is
  present. `provider_response` contains only `status_code`, `latency_ms`,
  integer `response_bytes`, `stream_completed`, `"scanned": false`, and a salted
  route hash; it cannot reconstruct model output.
- **Caller audit labels are pseudonyms, not authenticated identity.** New
  model/client/team values use field-separated `hmac:<64 lowercase hex>` labels.
  Grouping across restarts requires a persistent hash salt; ephemeral salts
  group one process lifetime. Export/dashboard readers omit raw schema-v2
  caller labels. Historical records are not rewritten and may retain older
  values, so operators must protect existing audit stores.
- **Keyed content hashes stay audit-only.** Block responses expose rule IDs but
  no longer return `prompt_hash`; callers cannot use the gateway as an online
  HMAC oracle. Operators with audit access must still protect the metadata-only
  audit store as sensitive.
- **Audit availability is part of enforcement.** Open, write, short-write, or
  lock failure produces `audit_write_failure` and blocks. Decision lock admission
  is capped at 150 ms and refusal writes at 250 ms; other shared writes retain
  the five-second cap. Writers use an interprocess lock, complete short
  writes, and truncate a failed partial tail; if rollback fails they append a
  best-effort newline. JSONL is still not a transactional journal on a failing
  filesystem. Use an external append-safe system when stronger durability is
  required. Audit payloads are frozen before worker dispatch; a stalled
  filesystem can still occupy a worker without blocking the request event loop.
- **Older cwd-relative audit copies need operator cleanup.** Runtime defaults
  are now absolute `/app/data` paths, but versions before this fix could write
  an audit copy relative to an ad-hoc process's working directory. The agent
  safety contract forbids inspecting those stores; operators upgrading should
  narrowly remove or chmod any known legacy copy themselves.
- **Policy reload can briefly block on invalid edits.** File stat/hash/load work
  is off-loop, unchanged files are not rehashed, and a valid change reloads
  automatically with the matching custom-term set. A malformed, missing, or
  timeout-limited reload blocks fail-closed until a valid policy is visible.
- **Large-body scan latency.** Built-in rules scan bounded overlapping windows,
  and credential/email quantifiers have reviewed upper limits, so the deadline
  remains cooperative even across a zero-match run. A single bounded regex
  window can still hold the GIL until that call returns. A shared five-second
  deadline checks between windows, rules, matches,
  encoding candidates, and reassembly sources; an outer async timeout returns a
  blocking verdict. Python cannot forcibly terminate an already-running worker
  thread, so it may finish its current bounded regex operation after the block
  has been produced.

- **Identity encoding is requested, not enforced on ordinary responses.** The
  edge overwrites upstream `Accept-Encoding` with `identity`. Ordinary responses
  are relayed without parsing or decoding, so a provider that ignores the
  request can still return compressed bytes. A successful `POST /files` is
  different: its bounded metadata is parsed to mint an attestation, so any
  non-identity `Content-Encoding` fails closed instead of being decoded.
- **Bare hexadecimal values remain ambiguous.** Human-natural and env labels
  such as `wallet key`, `auth_key`, `private key`, and `encryption_key` now
  detect 32–128-character hexadecimal credentials. A truly bare digest-length
  hexadecimal value remains exempt because it is indistinguishable from common
  Git, Docker, checksum, and content-address identifiers.
- **Readable two-word credential-field values are treated as identifiers.**
  Low-entropy, alphabetic camelCase values such as `tlsSecret`, `createdAt`,
  and `customerId` are exempted consistently with their snake/kebab forms.
  A real credential made only of two readable dictionary words can therefore
  require a custom term or stricter surrounding format; opaque mixed-case and
  keyworded alphanumeric values remain blocked.

- **Multipart support deliberately rejects exotic MIME syntax.** Skilak Mesh
  forwards raw multipart bytes, so a scanner/provider parser disagreement is a
  secret-smuggling boundary. Request multipart therefore requires
  one strict ASCII boundary parameter, CRLF framing, no preamble/epilogue, and
  no Content-Transfer-Encoding. Inner part headers must also be unique ASCII,
  unfolded, and unambiguous: duplicate `name`/`filename`/`charset`, RFC 2231
  extended parameters, nested multipart, content encoding/length, malformed
  quoting, and text charsets other than UTF-8/ASCII block as
  `file_unscannable`. Raw part headers and the exact accepted payload bytes are
  scanned on the request path. Some otherwise legal MIME
  producers may need to emit this narrower HTTP form. Response multipart is not
  parsed at all: responses are relayed.
- **Known-route queries remain unscanned; unknown-route identifiers are
  scanned.** Known-route query strings are appended to the pinned upstream URL
  verbatim and unscanned. An earlier four-name allowlist was removed because it
  applied to only some routes and silently dropped every other key — including
  parameters like `?beta=true` that change a real request's semantics — and an
  allowlist that discards fails invisibly. On an unknown route, each path segment
  and the complete query string is scanned as an immutable text leaf, then a
  clean query is forwarded verbatim. A secret in a known-route query therefore
  remains outside the detection boundary; put content in the request body.
- **Password/code precision is intentionally asymmetric.** Quoted multi-word
  passwords and hyphen/underscore diceware passphrases remain detectable.
  Unquoted space-separated prose after `password:` is not captured through the
  end of line, preventing routine help-desk and documentation text from hard
  blocking. An explicit passphrase setter accepts a bounded ordinary-word
  phrase; a direct copula requires four or more words and an embedded digit or
  symbol. A genuine plain-word passphrase in the latter shape can therefore be
  missed; quote it, express the setter, or use a strict custom term when that
  representation matters.


## Additional content limits

- **Bare lowercase-hex secrets need context.** A hex value such as a Datadog API key or a
  Twilio auth token without a nearby secret-family keyword, known provider
  prefix, or explicit config field (`DD_API_KEY=…`,
  `TWILIO_AUTH_TOKEN=…`) is not flagged because the same shape matches
  every Git commit SHA, Docker layer digest, and checksum in a diff.
  Lowering the recall threshold to catch these bare values would
  false-positive on every source-control diff viewed through the mesh.

- **`pii_email` drops second-level domains `test`, `example`,
  `invalid`, and `localhost`** as documentation/template addresses.  A
  real mailbox at one of those domains (rare) would not be flagged.

- **Archive layout must be fully accounted for.** TAR refuses nonzero trailers.
  ZIP refuses bytes after its declared end/comment, gaps between records,
  inconsistent directories, and unsupported ZIP64 footer layouts. Ordinary ZIP
  comments remain inspected. A ZIP inside another container is treated as nested
  only when its full span belongs to one already-inspected member or data fork.
  Generic self-extracting prefixes refuse; a prefix is admitted only after a
  separate PDF/image parser has inspected it.

- **PDF actions follow enumerate-or-refuse.** A PDF with only a `/URI`
  link is scanned and allowed; URI text is scanned. Extractable `/JavaScript`
  and `/JS` source is also scanned, but computed payloads such as
  `String.fromCharCode(...)` or split-and-concatenate can evade literal rules.
  Unreadable scripts and unenumerated actions block. `/Launch`, `/SubmitForm`,
  `/ImportData`, `/GoToE`, `/GoToR`, `/Movie`, `/Sound`, `/Rendition`,
  `/RichMediaExecute`, `/SetOCGState`, and URL-bearing `/Filespec` still
  block because their external effects cannot be derived from file text.
  DOCX external relationships, Markdown links, and plain-text mail links also
  still block.

  A 115-PDF corpus measured 43 active-content refusals (37 link-only) and 49%
  scanned clean before the URI change; then 6 refusals and 70% clean; then 0
  active-content refusals and 72% clean after enumerate-or-refuse. The remaining
  files were 15 oversized, 6 encrypted, 4 unscannable, 4 over the archive limit,
  and 3 over a scan limit. A separate 671-file corpus measured 122 link
  refusals (18%); do not combine the samples.

- **HIPAA Safe Harbor: 5 of the 18 identifiers are detected.**  Phone, email,
  SSN and IP are covered; names are partial (opt-in NER, off by default).  Not
  detected at all: geography/ZIP, dates, fax, medical record numbers, health-plan
  and account numbers, licence numbers, vehicle and device identifiers, URLs,
  biometrics, full-face photographs, and "any other unique identifying code".
  MRN and account numbers have no universal format — they are per-organisation,
  so no shipped pattern can catch them; dates and ZIP are matchable but flood on
  ordinary API traffic.  **Skilak Mesh is one technical safeguard, not HIPAA
  compliance**, and Safe Harbor de-identification is not claimed.  Skilak
  Consulting never receives customer data — the mesh is self-hosted and forwards
  directly to the provider — so Skilak is not a business associate.
- **Free-form PHI and HR prose remain partial.** Skilak Mesh detects structured
  PII, configured custom terms, and three narrow same-leaf contextual shapes:
  an HR performance/PIP marker combined with an adverse employment action; a
  clinical/patient/doctor-note marker combined with encounter and outcome cues;
  and at least two recognized medication dose/schedule entries. A fourth,
  `legal_contract_clause`, pairs one operative contract formula ("by and
  between", "in witness whereof", "represents and warrants") with a second
  nearby signal — another formula, a clause heading, or a contract-class
  document name. The shipped `hr_confidential`, `phi_health` and
  `legal_contract` categories block those matches because no
  observable hold channel exists. These rules are bounded lexicons, not
  semantic classifiers: euphemisms, unfamiliar drugs, one medication, salary
  prose/spreadsheets, non-English narrative, and a disease mention without the
  required context can still pass. **Opt-in NER** (named entity recognition)
  for free-form person names is available via the ``phi_person_name`` /
  ``phi_name`` detector, enabled with ``SKILAK_NER=1``.

  **Engine choice:** spaCy ``en_core_web_sm`` directly (not Presidio).
  spaCy is a ~15 MB model with minimal additional Python dependencies;
  Presidio adds ~200 MB of transformers/PyTorch without materially
  improving PERSON recall for the gateway use case.  The smaller
  dependency footprint matters for containerized edge deployments.

  **Latency (measured 2026-07-24, Apple-silicon macOS, slim pipeline —
  tok2vec+ner only):** ~22–25 ms/KB of prose — ≈45 ms for a 2 KB chat
  message, ≈220 ms at 10 KB, ≈1.2 s at 50 KB.  Texts are chunked
  (~100 KB windows, 256-char overlap) and all chunks are processed (no
  sampling — fail-closed), with the ScanDeadline checked per chunk.
  **Consequence:** with NER enabled, large prose payloads exhaust the scan
  deadline and are blocked fail-closed — size your NER-enabled pilot traffic
  accordingly.  The deadline is `SKILAK_TEXT_SCAN_TIMEOUT_SECONDS`, raised from
  5 s to 20 s on 2026-07-31 because the candidate ceiling grew and the clock had
  not; the 150–200 KB figure was measured against the old 5 s value and has not
  been re-derived, so treat it as a floor rather than the current limit.  Install the
  hash-locked dependencies with
  ``pip install --require-hashes -r requirements/ner.lock`` (pins both
  spaCy and the exact ``en_core_web_sm`` release wheel).

  **Residual gap:** spaCy's published ``en_core_web_sm`` NER accuracy is
  roughly 85–92% on typical English names; Skilak Mesh has not measured this
  independently on its own corpus.  It reduces, but does not eliminate, the
  PHI gap.  False
  positives are bounded by an allowlist (Claude, GPT, Gemini, etc.),
  single-token guards (< 3 chars), and code-fence exclusion.  False
  negatives remain for uncommon/spelled names and non-English names.
  HIPAA-heavy pilots may raise the ``phi_name`` category to ``block``,
  but the false-positive rate of NER makes ``block`` disruptive for
  general chat use; the default is ``redact``.
