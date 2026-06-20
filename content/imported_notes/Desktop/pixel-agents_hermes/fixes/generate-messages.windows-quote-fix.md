# Fix: build fails on Windows (`prettier` / `eslint` "No files matching the pattern")

## Symptom
On Windows, `npm run build` dies during `asyncapi:generate`:

```
[generate-messages] wrote ...\core\src\messages.ts (52 models)
[error] No files matching the pattern were found: "'C:\...\messages.ts'".
[generate-messages] prettier failed: Error: Command failed: npx prettier --write 'C:\...\messages.ts'
```

## Cause
`scripts/generate-messages.ts` wraps the output path in **POSIX single quotes**:

```ts
function quote(s: string): string {
  return `'${s.replace(/'/g, "'\\''")}'`;
}
```

`cmd.exe` (Node's default shell on Windows) does **not** strip single quotes, so the
literal quotes become part of the filename and prettier/eslint can't find the file.
(Linux is unaffected — the POSIX quoting works there.)

## Fix
Make `quote()` Windows-aware (double quotes on win32, POSIX single quotes elsewhere):

```ts
function quote(s: string): string {
  if (process.platform === 'win32') {
    return `"${s.replace(/"/g, '\\"')}"`;
  }
  return `'${s.replace(/'/g, "'\\''")}'`;
}
```

This is the only change needed; the rest of the build (type-check, lint, esbuild,
vite) then passes. The fix is already applied in the upstream fork on Linux, so you
only need this when building on Windows.
