# matrix-rust-components-swift (with Mac Catalyst)

The `MatrixRustSDK` Swift package, built from the same upstream source as
[element-hq/matrix-rust-components-swift](https://github.com/element-hq/matrix-rust-components-swift)
`26.10.15`, plus Mac Catalyst and macOS slices in
`MatrixSDKFFI.xcframework`. Use it exactly like element-hq's package:

```swift
.package(url: "https://github.com/haeser/matrix-rust-components-swift", exact: "26.10.15")
```

## What differs from element-hq 26.10.15

- **The xcframework has two more slices.** element-hq ships `ios-arm64` and
  `ios-arm64_x86_64-simulator`. This build adds `ios-arm64-maccatalyst` and
  `macos-arm64_x86_64`, so Mac Catalyst apps can link the SDK.
- **`Package.swift` points at this repo's release asset.** Only the
  `checksum` and `url` lines differ from element-hq's manifest.
- **Nothing else.** `Sources/MatrixRustSDK/*.swift` is byte-identical to
  element-hq's 26.10.15, and `LICENSE` is theirs (Apache-2.0).

## How it is built

- Source: [matrix-org/matrix-rust-sdk@97a1589](https://github.com/matrix-org/matrix-rust-sdk/commit/97a1589b70bd122c1d66e0d68ea008d8554076a5),
  the commit element-hq's 26.10.15 release names.
- One build-tooling patch, `xtask/` only (no SDK crate or FFI API change).
  It adds the `aarch64-apple-ios-macabi` target, which upstream declined in
  [matrix-org/matrix-rust-sdk#1915](https://github.com/matrix-org/matrix-rust-sdk/issues/1915).
- Build command, element-hq's release command plus three targets:

  ```
  cargo xtask swift build-framework --release \
    --target aarch64-apple-ios --target aarch64-apple-ios-sim --target x86_64-apple-ios \
    --target aarch64-apple-ios-macabi --target aarch64-apple-darwin --target x86_64-apple-darwin
  ```

The build script, the patch and the release playbook live in the private
repository of the Stozn iOS app (`scripts/build-matrix-sdk.sh`,
`scripts/release-sdk.sh`, `sdk/patches/`, `docs/SDK-RELEASE.md`). The
patch is reproduced below so the build can be checked without access to it.

```diff
diff --git a/xtask/src/swift.rs b/xtask/src/swift.rs
index e3b2f2e..a6d30b3 100644
--- a/xtask/src/swift.rs
+++ b/xtask/src/swift.rs
@@ -138,6 +138,7 @@ enum Platform {
     Macos,
     Ios,
     IosSimulator,
+    MacCatalyst,
     Watchos,
     WatchosSimulator,
 }
@@ -149,6 +150,7 @@ impl Platform {
             Platform::Macos => "macOS",
             Platform::Ios => "iOS",
             Platform::IosSimulator => "iOS Simulator",
+            Platform::MacCatalyst => "Mac Catalyst",
             Platform::Watchos => "watchOS",
             Platform::WatchosSimulator => "watchOS Simulator",
         }
@@ -161,6 +163,7 @@ impl Platform {
             Platform::Macos => "macos",
             Platform::Ios => "ios",
             Platform::IosSimulator => "ios-simulator",
+            Platform::MacCatalyst => "ios-maccatalyst",
             Platform::Watchos => "watchos",
             Platform::WatchosSimulator => "watchos-simulator",
         }
@@ -218,6 +221,12 @@ const TARGETS: &[Target] = &[
         status: TargetStatus::TopTier,
         description: "iOS Simulator (Intel) ",
     },
+    Target {
+        triple: "aarch64-apple-ios-macabi",
+        platform: Platform::MacCatalyst,
+        status: TargetStatus::TopTier,
+        description: "Mac Catalyst (Apple Silicon)",
+    },
     Target {
         triple: "aarch64-apple-watchos",
         platform: Platform::Watchos,
```

## Versions

Tags match element-hq's version. A rebuild of the same upstream version is
tagged `<version>-1`, `-2`, …; published tags and assets are never replaced.
Use `exact:` (SwiftPM) or `exactVersion:` (XcodeGen): a `-N` tag is a
semver pre-release and range requirements skip it.

## License

Apache-2.0, see [LICENSE](LICENSE) (from element-hq/matrix-rust-components-swift).
