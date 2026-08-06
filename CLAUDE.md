# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **composite GitHub Action** (`action.yml`) that uploads fuzzing data (bundles or corpus) to
FuzzCorp using the `fuzz-up` CLI tool. The entire action logic lives in a single file: `action.yml`.

## Architecture

The action executes these steps in order:
1. **Input validation** — validates `upload_type` (bundle|corpus), checks path exists, enforces
   type-specific required flags (`--lineage` for corpus)
2. **Environment validation** — requires `FUZZ_API_KEY`, `FUZZ_ORGANIZATION`, `FUZZ_PROJECT`
3. **Platform detection** — maps runner OS/arch to a binary name (linux/darwin × amd64/arm64) and emits the
   `binary_name` and `download_dir` outputs shared by the later steps
4. **Binary download** — fetches the `fuzz-up` binary and its attestation bundle from `asymmetric-research/fuzz-up`
   releases via `gh` into a download directory under `runner.temp` (never the workspace)
5. **Verification** — verifies the release's Sigstore attestation bundle with `gh attestation verify --bundle`
   (keyless, offline, bound to the fuzzcorp release workflow identity)
6. **Install** — moves the verified binary to `runner.temp/fuzz-up-bin`, adds it to `PATH`, prints its version
7. **Upload** — runs `fuzz-up upload <type> [args] <path>` (credentials are read from the environment)

## Development Notes

- There is no build step, test suite, or linter — this repo contains only `action.yml` and `README.md`.
- All inputs are passed to shell steps via environment variables (not inline interpolation) to prevent injection.
- Changes should be tested by referencing the action from a workflow in another repo.
