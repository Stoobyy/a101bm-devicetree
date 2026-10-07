# unmuda on native Windows (no WSL): a real CRLF bug in `cve31317.py`

`unmuda`'s `tools/cve31317.py` builds an exploit payload for CVE-2024-31317
and writes it to disk before pushing it to the device:

```python
p = Path("p31317.txt")
p.write_text(payload, encoding="utf-8")
```

On Linux/WSL this is fine. On native Windows, `Path.write_text()` without an
explicit `newline=` argument performs universal-newline translation: every
`\n` in `payload` gets written as `\r\n`.

The payload isn't arbitrary text — the script's own comments call it an
"oddbyte" format (`3000 "\n" + 5157 "A" + core + "," + 1400 "\n,"`), where the
exact byte count and line structure matter for how it lands via the
`WrapperInit`/`--invoke-with` mechanism in the Zygote argument-injection
exploit. On a real run on Windows, this corruption showed up as:

- the written file being exactly as many bytes larger as there were `\n`
  characters in the intended payload (every `\n` → `\r\n` adds one byte)
- inconsistent exploit behavior — not a clean success, not a clean
  "attempts exhausted" failure, but an actual phone reboot
  (`ro.boot.bootreason: reboot,android_system_crash` after the fact),
  consistent with a corrupted injection crashing something in the
  Zygote/`system_server` path hard enough to trigger a watchdog reboot.

## The fix

```python
p.write_text(payload, encoding="utf-8", newline="\n")
```

Verified: with this change, the written file's byte count exactly matches
`len(payload.encode("utf-8"))` and contains zero `\r` bytes, matching what
WSL/Linux produces natively. After this fix, the exploit ran successfully on
the first real attempt on native Windows.

This is a one-line fix and plausibly worth a PR against `unmuda` directly, for
the benefit of anyone else running it from native Windows Python rather than
WSL.
