# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **composite GitHub Action** (`action.yml`) that uploads fuzzing data (bundles or corpus) to
FuzzCorp using the `fuzz-up` CLI tool. The entire action logic lives in a single file: `action.yml`.

## Architecture

The action executes these steps in order:
1. **Input validation** — validates `upload_type` (bundle|corpus), checks path exists, enforces
   type-specific required flags (`--lineage` for corpus)
2. **Environment validation** — requires `FUZZ_API_ORIGIN`, `FUZZ_ORGANIZATION`, `FUZZ_PROJECT`, `FUZZ_USER`,
   `FUZZ_PASSWORD`
3. **Platform detection** — maps runner OS/arch to binary name (linux/darwin × amd64/arm64)
4. **Binary download** — fetches the `fuzz-up` binary and its attestation bundle from `asymmetric-research/fuzz-up`
   releases via `gh`
5. **Verification** — verifies the release's Sigstore attestation bundle with `gh attestation verify --bundle`
   (keyless, offline, bound to the fuzzcorp release workflow identity)
6. **Login + Upload** — authenticates and runs `fuzz-up upload <type> [args] <path>`

## Development Notes

- There is no build step, test suite, or linter — this repo contains only `action.yml` and `README.md`.
- All inputs are passed to shell steps via environment variables (not inline interpolation) to prevent injection.
- Changes should be tested by referencing the action from a workflow in another repo.
