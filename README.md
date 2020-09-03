To replicate, run:
```sh
cross -V
```
You will see (you may need to run twice as cross may install stuff first time):
```
cross 0.2.1
info: syncing channel updates for '1.45.1-x86_64-unknown-linux-gnu'

  1.45.1-x86_64-unknown-linux-gnu unchanged - rustc 1.45.1 (c367798cf 2020-07-26)

info: checking for self-updates
cargo 1.45.1 (f242df6ed 2020-07-22)
```

However, if you remove the `rust-toolchain` file, then you see (again, may need to run twice to see this exact output):
```
cross 0.2.1
cargo 1.40.0 (bc8e4c8be 2019-11-22)
```

This is because of https://github.com/rust-embedded/cross/blob/5d35b3f33519657ef15446f6d52221a3a9e8e51b/src/rustup.rs#L31
```rust
pub fn installed_toolchains(verbose: bool) -> Result<Vec<String>> {
    let out = Command::new("rustup")
        .args(&["toolchain", "list"])
        .run_and_get_stdout(verbose)?;

    Ok(out.lines().map(|l| l.replace(" (default)", "").trim().to_owned()).collect())
}
```

But the output of `rustup toolchain list` in this directory is
```
stable-x86_64-unknown-linux-gnu (default)
1.45.1-x86_64-unknown-linux-gnu (override)
```

Note the `(override)`. If I patch `rustup.rs` to
```
-    Ok(out.lines().map(|l| l.replace(" (default)", "").trim().to_owned()).collect())
+    Ok(out.lines().map(|l| l.replace(" (default)", "").replace(" (override)", "").trim().to_owned()).collect())
```

Then you see
```
cross 0.2.1
cargo 1.40.0 (bc8e4c8be 2019-11-22)
```
without the redundant channel sync

