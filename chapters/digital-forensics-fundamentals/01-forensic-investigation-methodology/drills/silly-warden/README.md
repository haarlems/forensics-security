# silly-warden

Evidence Item #002: a chain-of-custody form, scanned and filed.
Looks complete, every handoff signed, but what's it hiding?

Artifact:
`custody_sheet.pdf`

Hint 1:
The evidence will tell you the flag if you ask it the right way.
Your eyes alone won't show you the flag, and neither will the first thing you try.

Hint 2:
A PDF page isn't stored the way it looks.
The text on the page lives in a content stream, and content streams are usually packed down before they're written to disk.
`strings` finds nothing because the page content is `FlateDecode-compressed`.

Solution in `../solution/`
