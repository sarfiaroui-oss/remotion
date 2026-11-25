# Standalone snapshot audit and development

This independent snapshot belongs to `sarfiaroui-oss/remotion`, not the official
Remotion distribution. Source: upstream main at
`39ba10162371e8e74b78682607b2cacabcc47443` (2026-11-24), Remotion **4.0.547**.
Source attribution and documentation URLs are retained; they are not Git remotes
or submodules. All original license/copyright/third-party notices are preserved.
The Remotion license remains applicable; independence does not change its terms.

## Snapshot policy

- Start with an empty destination, one root commit on `main`, no tags, no upstream
  objects/remotes, no gitlinks, nested Git metadata, submodules or LFS objects.
- `bun.lock`, `bunfig.toml`, `.gitattributes`, `.gitignore`, workspace dependencies,
  package names/versions, patches and build configuration are unchanged.
- The source contains 14,845 original tracked files, totaling 577,721,483 bytes.
  Largest: `packages/compositor-linux-x64-musl/libavcodec.so`, 21,993,672 bytes,
  below GitHub's 100 MiB single-blob limit. No essential source was excluded; three original generated cache/type files
  listed below are intentionally excluded.
- Upstream-tracked native FFmpeg/compositor executables and libraries are retained
  intentionally. The owner approved preserving these required assets and the
  lockfile: `workspace:*` resolves the compositor packages locally, not to npm.
  No dependency vendoring or generated build outputs are added. Original tracked
  third-party source, including existing vendor directories, is preserved.
- Chromium is downloaded on demand by the renderer (tested browser version
  158.0.8127.2), not through `bun.lock`. Normal registry dependencies remain
  normal Bun downloads. Do not add dependencies, caches or browser downloads to Git.

## Toolchains and Windows 10/11

- **Bun 1.4.3** is the exact `packageManager` pin; engine minimum is >=1.4.3.
- **Node >=16** is the root engine declaration. There is no exact Node pin;
  some optional workspaces/dependencies need newer Node. Use a maintained Node
  release and check the selected workspace. CI covers multiple Node versions.
  The setup host has Node 24.22.3; this is an observation, not a repository pin.
- **Rust has no exact version/MSRV pin.** The compositor uses edition 2021 and
  Cargo lock format 4. Use current stable Rust/Cargo if rebuilding it, subject
  to the locked dependencies' requirements. Rust is optional for TypeScript work.
- Go is optional for Lambda SDK work: root go.work says 1.26.1; the SDK declares
  Go 1.26 with toolchain go1.26.5. Python/PHP/Ruby are optional SDK toolchains.

On Windows install Git, a maintained Node release, and Bun 1.4.3 using Bun's
supported installer/version selection. Confirm `bun --version` before running:

```powershell
git clone https://github.com/sarfiaroui-oss/remotion.git
cd remotion
bun ci
bun run build
# A smaller upstream build option:
# bun run build-fast
cd packages/example
bun run dev
```

Studio's documented testbed uses port 3000. After the CLI/workspaces build,
from `packages/example` an example render is:

```powershell
bunx remotion render ten-frame-tester --output ../../out/video.mp4
```

These development commands are upstream-supported instructions, not executed
build/render results for this fast snapshot. The native compositor build script
currently exits without compiling on native Windows and says it requires
cross-compilation. Its zero exit is not proof of a Windows Rust build; use a
supported Linux/macOS cross-build environment (WSL2 is an option to investigate).

Each cloud task is already isolated. Use its existing checkout; do not create
Git worktrees unless explicitly requested. Dependency installation may enable
`.githooks` via the `prepare` hook; pre-commit then formats files using Bun.

## Non-registry dependency audit

All 133 package manifests were parsed, including dependencies, dev/peer/optional
dependencies, overrides and the root workspace catalog. **No direct GitHub URL
or GitHub shorthand dependency occurs in those package manifest fields.**

| Dependency                                                          | Source                                                                                                                                     | Location                                       |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------- |
| `@remotion-dev/cube-presentation` (direct HTTP, not GitHub)         | `https://www.remotion.pro/pkg/cube-transition/1.0.2/docs`                                                                                  | `packages/docs/package.json`                   |
| `@electron/node-gyp` (transitive through `@electron/rebuild@3.7.2`) | `git+https://github.com/electron/node-gyp.git#06b29aafb7708acef8b3669835c8a7857ebc92d2`; lock shorthand `github:electron/node-gyp#06b29aa` | `bun.lock`                                     |
| `mp4` (Cargo Git)                                                   | `https://github.com/jonnyburger/mp4-rust`, rev `92ba375738cc2f05a4d754e1f968cf2e97d06641`                                                  | `packages/compositor/Cargo.toml`, `Cargo.lock` |
| `ffmpeg-next` (optional FFmpeg binding helper, Cargo Git)           | `https://github.com/zmwangx/rust-ffmpeg`, rev `a7b50dd5f909e071affffe238d434d184da30aee`                                                   | `packages/compositor/ffmpeg/build/Cargo.toml`  |

The compositor's primary `ffmpeg-next`/sys wrappers are local source dependencies.
Go's versioned GitHub module names are normal Go module identities; go.work uses
the local SDK. Python manifests have no direct GitHub requirements. No lockfiles
were regenerated and no dependencies were vendored.

## GitHub Actions and external effects

Testing workflows are retained. No main-push workflow publishes packages, deploys
docs, or declares upstream-only secrets. Lambda integration CI uses fake AWS.

| Workflow                  | Trigger and standalone behavior                                                                                                                                             |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `push.yml`                | Every `main` push and PR: broad install/lint/build/test matrix. Main selects full CI, including the first root commit. Potentially expensive; not disabled.                 |
| `ffmpeg.yml`              | Main push/PR when compositor FFmpeg paths, this YAML or CircleCI config change; also manual. Native FFmpeg builds/artifacts; no upstream secrets. Eligible on initial push. |
| `copilot-setup-steps.yml` | Manual and push/PR changing this file: install/build, no upstream secrets. Eligible on initial push.                                                                        |
| `discord.yml`             | Push and pull_request_target; existing official-repository condition skips its job here. Uses upstream DISCORD_WEBHOOK only in the official repo.                           |
| `changelog.yml`           | Published release/manual; upstream DISCORD_TOKEN. One added official-repository job guard skips it here.                                                                    |
| `claude.yml`              | Issue/comment/review events containing @claude; needs CLAUDE_CODE_OAUTH_TOKEN. No main-push trigger; leave unused or disable until independently configured.                |
| `pullfrog.yml`            | Manual external-agent workflow with write/OIDC permissions and optional provider keys; no main-push trigger. Leave unused or configure deliberately.                        |

CircleCI's FFmpeg pipeline is API-triggered only and requires a connected
CircleCI project. No bulk workflow edits or test configuration removals were made.
YAML parsing establishes syntax only; workflows have not been run or verified by
GitHub. In particular upstream push.yml contains `parallel` step groups whose
GitHub service support/acceptance is unverified. No CI success is claimed.

Official publishing is blocked with a shared throwing guard, including direct
script invocation. The safe `publish.ts --list` and `publish.ts --check`
validation modes remain available; actual publication stops before release side
effects. The root release shortcut no longer contains a tag push.
The original official publishing implementations remain for reference, behind
that guard. Generic user-facing Lambda/Cloud Run deployment APIs and template
deploy examples remain available; they need the user's own accounts. Do not
manually run `npm publish` or `bun publish` in unchanged official package names:
package identities are preserved for workspace compatibility. Configure independent
names/accounts and release policy before intentionally enabling publication.

## Docker audit

The supported `packages/dockerfiles/Dockerfile.debian`, `.ubuntu22`, `.ubuntu24`
and `.al2023-arm64` use public base images and locally packed workspace tarballs
plus the locally bundled example, not a private/prebuilt Remotion application.
Their supported source workflow is, after `bun ci` and building the workspaces:

```sh
cd packages/dockerfiles
./run.sh
```

It builds multiple architectures and renders two browser test compositions per
image; see the original README for output filenames. This is a test-rendering
workflow, not a persistent server. Linux rendering requires glibc >=2.35;
Debian Bookworm / Ubuntu 22.04+ are suitable families. The existing Amazon Linux
2023 ARM64 recipe explicitly supports glibc 2.34; x64 is unsupported there.
Do not use Alpine as the browser rendering base. CircleCI's Alpine recipe builds
FFmpeg musl artifacts, not the browser renderer. The unsupported Nix experiment
clones an official example template and is already excluded from run.sh.
No Docker/Compose files were changed. Existing unpinned installer downloads were
preserved. Docker builds, container renders and Windows execution were not run.

## Exact intentional changes from upstream

- `package.json`: standalone repository/homepage/issues metadata; three official
  publishing shortcuts now fail closed. Dependency/workspace/version fields intact.
- `README.md`: standalone identity, clone/development instructions, audit link,
  and standalone issue tracker. Original license text/attribution retained.
- `STANDALONE.md`: this audit and development/validation limitations.
- `scripts/standalone-no-publish.cjs`: shared fail-closed official release guard.
- `.github/workflows/changelog.yml`: one official-repository job guard.
- `publish.ts`: guard official npm publishing.
- `set-version.ts`: guard version/tag mutation and remote tag deletion.
- `packages/lambda/src/admin/publish-function-artifacts.ts`: guard official S3 publishing.
- `packages/it-tests/src/templates/publish.ts`: guard upstream template/plugin pushes.
- `packages/lambda-go/publish.mjs`: guard upstream Go SDK pushes/tags.
- `packages/lambda-php/publish.mjs`: guard upstream PHP SDK pushes/tags.
- `packages/lambda-python/publish.mjs`: guard Python SDK publishing.
- `packages/lambda-ruby/publish.ts`: guard upstream Ruby SDK pushes/tags.
- `packages/cloudrun/container/submit.mjs`: guard official remotion-dev image publishing.
- `packages/discord-poster/post.ts`: guard official release Discord posting.
- `packages/lambda-binaries/upload.sh`: fail before official S3 layer uploads.

Three original generated/cache files are intentionally excluded to avoid committing
build caches: `packages/astro-example/.astro/types.d.ts`,
`packages/lambda-python/remotion_lambda/__pycache__/RenderParams.cpython-311.pyc`,
and `packages/lambda-python/tests/__pycache__/test_client_with_unittest.cpython-311.pyc`.
The latter two were ignored by upstream too, but present in its tracked snapshot.
No other upstream files were intentionally modified or removed. Setup verification
checks source hashes, all JSON manifests and workflow YAML syntax, Bun lock/TOML
syntax, guard behavior, blob sizes, package version alignment, staged-file hygiene,
and Git branch/history/remotes/submodule/LFS invariants. A subsequent fresh clone
must independently confirm the pushed Git/source invariants. Full `bun ci`, builds,
Studio startup and video rendering are outside the revised fast-snapshot check;
this document does not claim they passed.
