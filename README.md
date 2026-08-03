# app-fontbook

**Font Book, on [`mokuroku`](https://github.com/kotoba-lang/mokuroku).**

Design: [ADR-2608035000](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2608035000-app-standard-application-suite-on-a-shared-catalog-kernel.edn).

Capability: `fs/browse`. Nothing in this repo performs the effect — the host
supplies the provider function, and that is where the grant is spent.

## Three decisions

**Regular has many names and they are one face.** Book, Roman, Normal, Text and
an empty style all normalise to Regular. Left alone, one family shows three
different Regular rows that sort apart and look like three faces.

**Weight is a number, not a style name.** Sorting a family by style name puts
Black before Bold before Light, which reads as a weight order and is the
reverse of one in two places. An undeterminable weight is `nil`, not 400 —
guessing files it with the regulars it may not belong to, and the row is
badged `Weight unknown`.

**A face is family + style, not a file path.** The same face is routinely
installed twice, once by the system and once by a user; keying on the path
shows two rows that cannot be told apart. Duplicates are surfaced rather than
merged — two files claiming to be the same face is a real condition a font
manager exists to show.

No install and no remove: those mutate a system-wide font set, and removing a
font the system is currently rendering with is a different, riskier program.

## Test

```sh
clojure -M:local:test    # sibling checkouts
clojure -M:test          # pinned git deps
clojure -M:lint
```

design-quality: 100.00 on every window state including awaiting-grant (2026-08-03).
