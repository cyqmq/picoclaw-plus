<div align="center">
<img src="assets/logo.webp" alt="PicoClaw" width="256">

<h1>PicoClaw-Plus</h1>

<p>
  <img src="https://img.shields.io/badge/Go-1.25+-00ADD8?style=flat&logo=go&logoColor=white" alt="Go">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="License">
</p>

</div>

A maintained companion fork of [sipeed/picoclaw](https://github.com/sipeed/picoclaw) — an ultra-lightweight, pure-**Go** personal AI assistant — with a fully vendored, hermetic QQ integration stack and targeted fixes on top of upstream `main`. Everything except the QQ channel behaves exactly like upstream.

> **Everything else (news, full feature list, install & quick start, providers, channels, tools, skills, MCP, CLI reference, docs)** lives upstream: **[github.com/sipeed/picoclaw](https://github.com/sipeed/picoclaw)** · **[docs.picoclaw.io](https://docs.picoclaw.io)**.

---

## 📍 Current State

* **Baseline**: `sipeed/picoclaw` `main` @ `bbf6893` (v0.2.1-era QQ codebase). All fork changes are additive: **171 files / 9,359 lines** on top of upstream history.
* **Hermetic QQ SDK**: the entire SDK is committed in-repo at `third_party/botgo` (module identity `github.com/tencent-connect/botgo`) and wired via a `replace` directive in `go.mod` — **the build needs no GitHub access**, safe for CI sandboxes and CN mirrors.
* **Verified green**: `go build ./...`, `go test ./pkg/channels/qq/` (15/15), plus prebuilt **static** binaries for `amd64` and `linux-arm64`.
* **Live A/B confirmed**: the QQ Gateway now connects with `200 wss://api.sgroup.qq.com/websocket` instead of the previous `401 code:11241` (请求头Authorization参数格式错误).

## ✅ Benefits vs Upstream

| # | Benefit | Detail |
|---|---------|--------|
| 1 | **Hermetic build** | SDK is committed in-repo; upstream relies on the GitHub-only `tencent-connect/botgo`, which can break or be blocked (CI, CN mirrors). |
| 2 | **OpenID messages work** | Upstream's `dto.User` lacks `MemberOpenID`/`UserOpenID`, so openid-form QQ messages never resolved to a sender. Fixed here. |
| 3 | **QQ 401 root cause fixed** | Upstream `go.mod` force-upgrades `go-resty/resty` to `v2.17.1`, which silently changes the request `Authorization` scheme and breaks the gateway (`code 11241`). Pinned back to `v2.6.0` (the version the SDK uses) — proven by a controlled A/B. |
| 4 | **Regression tests** | New tests lock in the openid → sender mapping so it cannot silently regress. |
| 5 | **Static arm64 binary** | `CGO_ENABLED=0` yields a dependency-free `build/picoclaw-linux-arm64`. |

## ✨ New Features

* **OpenID sender resolution** — C2C messages fall back to `Message.Author.UserOpenID`, group messages to `Message.Author.MemberOpenID`, when `Author.ID` is empty; resolved value is propagated to `sender_id` / `platform_id` / `canonical_id`.
* **QQ group & C2C management events** — `GROUP_ADD_ROBOT`, `GROUP_DEL_ROBOT`, `GROUP_MSG_REJECT`, `GROUP_MSG_RECEIVE`, `C2C_MSG_REJECT`, `C2C_MSG_RECEIVE`.
* **Rich-media direct upload** — botgo-plus `file_data` upload (direct binary upload, no public URL required).

## 🔧 Build

```bash
# Go 1.25+ — fork build is hermetic, the SDK is already vendored
make build                # amd64            → build/picoclaw
make build-linux-arm64    # static linux-arm64 → build/picoclaw-linux-arm64
```

WebUI launcher / Docker / Android builds are unchanged from upstream — see the upstream README for those.

## 📦 Referenced Projects

| Project | Role |
|---------|------|
| [sipeed/picoclaw](https://github.com/sipeed/picoclaw) | Upstream project this fork is based on. |
| [kylin930/botgo-plus](https://github.com/kylin930/botgo-plus) | QQ SDK fork, vendored at `third_party/botgo`. |
| [tencent-connect/botgo](https://github.com/tencent-connect/botgo) | Official QQ OpenAPI SDK baseline (`v0.2.1`). |
| [go-resty/resty](https://github.com/go-resty/resty) | HTTP client, pinned to `v2.6.0`. |
| [golang.org/x/oauth2](https://pkg.go.dev/golang.org/x/oauth2) | QQ token source. |