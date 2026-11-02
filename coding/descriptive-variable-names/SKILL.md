---
name: descriptive-variable-names
description: Use whenever writing or editing source code in any language. Prevents creating short, cryptic variable names (2-3 letter abbreviations or acronyms of domain nouns) and enforces full descriptive identifiers — for example `reservationNumber` instead of `rn`, `responseData` instead of `rd`, `passenger` instead of `pax`. Applies to local variables, parameters, struct fields, function returns, and constants.
---

# Descriptive Variable Names

When writing or editing source code, use full, descriptive variable names. Do not shorten a domain noun into a 2–3 character abbreviation just to save typing.

## The rule

If a variable holds a meaningful domain concept, spell it out:

| ❌ Bad | ✅ Good |
|--------|---------|
| `rn`   | `reservationNumber` |
| `rd`   | `responseData` |
| `gd`   | `generalData` |
| `pax`  | `passenger` |
| `tp`   | `totalPrice` |
| `req`  | `request` (in non-handler scopes; `req` is OK for HTTP handler params, see exceptions) |
| `res`  | `response` or `result` |
| `cfg`  | `config` |
| `usr`  | `user` |
| `addr` | `address` |
| `qty`  | `quantity` |
| `idx`  | `index` |

Apply this when declaring variables, naming parameters, naming struct fields, and naming returned values in any language (Go, TypeScript, JavaScript, Python, Java, Rust, etc.).

## Exceptions — these short names ARE OK

Some short names are so idiomatic they read more clearly than the long form. Don't fight them:

- **Loop counters** in tight loops: `i`, `j`, `k` for indices; `n` for length.
- **Error values in Go**: `err` is the standard idiom — keep it.
- **Range variables when the type is obvious from the range expression** and used in 1–3 lines: `for _, c := range customers` is fine if `c` is used immediately and locally.
- **Go method receivers**: Go convention is 1–2 letter receivers (`t *Tree`, `s *Server`). Match the surrounding file's style.
- **Standard initialisms used as full words**: `id`, `url`, `uri`, `db`, `os`, `io`, `fs`, `ip`, `tcp`, `http`, `json`, `xml`, `dto`, `dao` — these are accepted as words. `userID`, `dbConn`, `httpClient` are fine.
- **Math/physics conventions** when matching established notation: `x`, `y`, `z` for coordinates, `r` for radius, `n` for sample size in a statistical formula.
- **Acronyms with a clear, codebase-established meaning** that appear in struct field tags or external API field names — preserve the API's casing.

When in doubt, **prefer the longer name**. Reading code is more common than writing it; a clear name pays back every time someone (including the original author) reads it.

## Translating existing short names

If you encounter cryptic short names while editing a file, rename them as part of the edit — but only within the scope you're already changing. Do not do a wholesale rename pass on untouched code unless the user explicitly asks for it.

## Examples

❌ `if rn := extractReservationNumber(qpp); rn != "" { order.ReservationNumber = &rn }`
✅ `if reservationNumber := extractReservationNumber(qpp); reservationNumber != "" { order.ReservationNumber = &reservationNumber }`

❌ `responseData := qpp.PurchaseDetail.ResponseData; rd := responseData.GeneralData`
✅ `responseData := qpp.PurchaseDetail.ResponseData; generalData := responseData.GeneralData`

❌ `func process(req *Request, res *Response) { ... }`  *(when not in an HTTP handler context)*
✅ `func process(request *Request, response *Response) { ... }`

❌ `for _, p := range passengers { /* 30 lines using p */ }`
✅ `for _, passenger := range passengers { /* 30 lines using passenger */ }`
