# FieldKit 0.1.0

Independent editing sessions and native commit-format fields. Import `FieldKit`.
Depends only on MaskKit 0.1.0; no FormatKit, TemporalKit or SettingsKit dependency.
Canonical values are serialized strings; numeric/temporal adapters belong to callers.

## Codec and session contract

`FieldCodec` has `Analyze(text, profile) As FieldAnalysis`,
`FormatDisplay(canonical, profile) As String`, and
`FormatEdit(canonical, profile) As String`. Profile is an immutable serialized
String snapshot, not a mutable profile object. Codecs must be deterministic,
side-effect free, preserve full precision in edit format, and return a canonical
candidate only for Valid analysis. Display may round without changing storage.

`FieldAnalysis(state, candidate As Optional Of String, errorCode, errorPosition)`
is immutable. `FieldState` is Empty, Incomplete, Invalid, Valid. Positions are
codepoint offsets, -1 when unavailable. Valid requires Some(candidate); other
states require None. Empty is exactly zero-length draft, handled by the session;
whitespace is passed to the codec. Empty canonical string (Some("")) and None
are distinct. Codecs cannot return Empty for nonempty input.

`EditSession(codec, nullable)` starts with no value. Read-only properties:
`DraftText`, `CommittedValue`, `Candidate`, `Analysis`, `State`, `IsDirty`,
`IsEditing`, `EditProfile`, `DisplayText`.

- `SetValue(value As Optional Of String, profile)` explicitly replaces value,
  cancels any edit, formats display, and clears dirty. None requires nullable.
  Canonical validity is the caller's responsibility. Initial None is permitted
  even in required sessions; attempting to commit it fails.
- `BeginEdit(profile)` captures the profile and full precision edit text;
  repeated BeginEdit during an active/failed edit is a no-op.
- `SetDraft(text)` requires an active edit, analyzes without changing storage.
  Dirty means draft differs from initial edit text (reverting clears dirty).
- `TryCommit() As Boolean` requires editing. Untouched present values bypass
  parsing, preserving exact canonical precision. Valid or nullable Empty commits;
  failures retain draft, profile, editing state and old value. Successful commit
  ends editing and sets display using the captured profile. Codec exceptions
  propagate before mutation; invalid analysis contracts throw.
- `CancelEdit()` restores the prior display and clears editing/dirty/candidate.
  Call BeginEdit with a new profile to apply a subsequent settings change.

`CommitField(codec, nullable)` is a real `TextField(FieldType.Normal)` subclass.
`Session` is read-only; `Profile` is the next-edit profile. Use field
`SetValue(value)` and `CancelEdit()` to synchronize native text; do not mutate
Session directly while attached. Focus gain calls BeginEdit; TextChanged only
captures/analyzes draft; successful focus loss formats display; failure leaves
text intact and raises `CommitRejected`. Success raises `ValueCommitted`.
Programmatic inherited Text writes do not raise TextChanged; focus loss reads
native text to capture them. No live punctuation rewrite, custom key filtering,
composition interception or undo transaction emulation. Accessible labels and hints remain caller-owned and are never overwritten.
Native undo/composition
remain owned by the platform during typing. Focus-boundary text replacement can
reset native undo; no cross-focus undo guarantee. IME composition/focus ordering
and foreign native behavior require platform validation.

## Pure masks

Import `fieldkit.FieldMasks`. `MaskEditor(pattern)` wraps MaskKit TextMask. Read-only `RawText`, `Text`,
`IsComplete`. `Replace(start, length, pasted, formatted)` returns the new formatted
caret offset; selection coordinates count Unicode codepoints. Atomic rejection
leaves all state unchanged. Formatted paste must be a valid prefix of the whole
mask and has its literals stripped; raw paste contains slots only. Replacements
splice slots covered by the formatted selection. `Backspace(caret)` and
`DeleteForward(caret)` delete adjacent raw slots, skipping literals. Mapping:
`RawOffset(formattedOffset)` counts preceding slots; `FormattedOffset(rawOffset)`
chooses the boundary immediately before the next slot (after intervening literals).
Bounds are validated, never clamped. Empty raw data may render leading literals
for literal-only masks, following MaskKit. No native live mask adapter is claimed.

Supported grammar is **MaskKit**, not Xojo InputMask: # ASCII digit, L uppercase
ASCII letter, l lowercase ASCII letter, A ASCII letter, * ASCII alphanumeric,
. printable codepoint; other characters literal; backslash escapes one literal.
All slots required for completion; maximum 4096 codepoints/16384 UTF-8 bytes.
No optional slots, repetitions or Xojo case/placeholder directives. There is no
Xojo translator: interpreting overlapping grammar without a full compatibility
contract would silently alter input semantics.

Pattern token recognition is case sensitive, including literal lowercase `a`.
For example, `Data ##` with raw `12` displays `Data 12` and stores only `12`.
Native controls require the UI thread; pure editing instances are unsynchronized
and must each be confined to one thread.
