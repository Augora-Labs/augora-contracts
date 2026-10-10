# VS Code extensions

`extensions.json` recommends the extensions this Rust (Soroban) workspace actually needs:

| Extension | Why it is recommended |
|---|---|
| `rust-lang.rust-analyzer` | Language server for the workspace; matches the `rust-analyzer` component pinned in [`rust-toolchain.toml`](../rust-toolchain.toml). |
| `vadimcn.vscode-lldb` | Rust debugging on all supported platforms. |
| `tamasfe.even-better-toml` | Formatting and validation for the workspace's `Cargo.toml`/`.toml` files. |

There is no JavaScript, TypeScript, HTML, CSS or PlantUML surface in this repository, so
Prettier and the PlantUML preview extensions are intentionally **not** recommended.