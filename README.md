# Fuzz Upload Action

GitHub Action to upload fuzzing data (assets, bundles, or corpus) to FuzzCorp.

## Usage

```yaml
- name: Upload Fuzz Bundle
  uses: asymmetric-research/fuzz-upload-action@v1
  with:
    upload_type: bundle
    upload_path: ./path/to/fuzz-bundle.zip
  env:
    FUZZ_API_ORIGIN: ${{ secrets.FUZZ_API_ORIGIN }}
    FUZZ_ORGANIZATION: ${{ secrets.FUZZ_ORGANIZATION }}
    FUZZ_PROJECT: ${{ secrets.FUZZ_PROJECT }}
    FUZZ_USER: ${{ secrets.FUZZ_USER }}
    FUZZ_PASSWORD: ${{ secrets.FUZZ_PASSWORD }}
```

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `upload_type` | Yes | Type of data to upload: `asset`, `bundle`, or `corpus` |
| `upload_path` | Yes | Path to the file or directory to upload |
| `upload_args` | No | Additional arguments/flags to pass to `fuzz-up upload` |

## Environment Variables

All FuzzCorp credentials must be set as environment variables:

| Variable | Required | Description |
|----------|----------|-------------|
| `FUZZ_API_ORIGIN` | Yes | FuzzCorp API origin URL |
| `FUZZ_ORGANIZATION` | Yes | Your FuzzCorp organization name |
| `FUZZ_PROJECT` | Yes | Your FuzzCorp project name |
| `FUZZ_USER` | Yes | FuzzCorp username |
| `FUZZ_PASSWORD` | Yes | FuzzCorp password |

## Type-Specific Requirements

### Asset Uploads
When uploading assets, you must include the `--lineage` flag:

```yaml
- uses: asymmetric-research/fuzz-upload-action@v1
  with:
    upload_type: asset
    upload_path: ./binary
    upload_args: --lineage my-lineage-name
  env:
    # ... environment variables
```

### Corpus Uploads
When uploading corpus data, you must include the `--seed-corpus-group` flag:

```yaml
- uses: asymmetric-research/fuzz-upload-action@v1
  with:
    upload_type: corpus
    upload_path: ./corpus/
    upload_args: --seed-corpus-group my-corpus-group
  env:
    # ... environment variables
```

### Bundle Uploads
Bundle uploads don't require additional flags:

```yaml
- uses: asymmetric-research/fuzz-upload-action@v1
  with:
    upload_type: bundle
    upload_path: ./bundle.zip
  env:
    # ... environment variables
```

## Platform Support

This action supports:
- **OS**: Linux, macOS
- **Architecture**: x64 (amd64), ARM64

The appropriate `fuzz-up` binary is automatically downloaded based on your runner platform.

## Example Workflow

```yaml
name: Upload Fuzzing Bundle

on:
  push:
    branches: [main]

jobs:
  upload:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build fuzz bundle
        run: ./scripts/build-fuzz-bundle.sh

      - name: Upload to FuzzCorp
        uses: asymmetric-research/fuzz-upload-action@v1
        with:
          upload_type: bundle
          upload_path: ./build/fuzz-bundle.zip
        env:
          FUZZ_API_ORIGIN: ${{ secrets.FUZZ_API_ORIGIN }}
          FUZZ_ORGANIZATION: ${{ secrets.FUZZ_ORGANIZATION }}
          FUZZ_PROJECT: ${{ secrets.FUZZ_PROJECT }}
          FUZZ_USER: ${{ secrets.FUZZ_USER }}
          FUZZ_PASSWORD: ${{ secrets.FUZZ_PASSWORD }}
```