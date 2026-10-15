# Fuzz Upload Action

GitHub Action to upload fuzzing data (bundles or corpus) to AR Fuzzing.

## Usage

```yaml
- name: Upload Fuzz Bundle
  uses: asymmetric-research/fuzz-upload-action@v2
  with:
    upload_type: bundle
    upload_path: ./path/to/fuzz-bundle
  env:
    FUZZ_ORGANIZATION: ${{ secrets.FUZZ_ORGANIZATION }}
    FUZZ_PROJECT: ${{ secrets.FUZZ_PROJECT }}
    FUZZ_API_KEY: ${{ secrets.FUZZ_API_KEY }}
```

## Inputs

| Input         | Required | Description                                                          |
|---------------|----------|----------------------------------------------------------------------|
| `upload_type` | Yes      | Type of data to upload: `bundle` or `corpus`                         |
| `upload_path` | Yes      | Path to the file or directory to upload (relative to workspace root) |
| `upload_args` | No       | Additional arguments/flags to pass to `fuzz-up upload`               |

## Outputs

| Output         | Description                                     |
|----------------|-------------------------------------------------|
| `fuzz_up_path` | Absolute path to the installed `fuzz-up` binary |

The action does not add `fuzz-up` to `PATH`. To run it in a later step, give the action step an `id` and pass this
output through `env`:

```yaml
- id: upload
  uses: asymmetric-research/fuzz-upload-action@v2
  with:
    upload_type: bundle
    upload_path: ./path/to/bundle
  env:
    # ... environment variables

- name: Check the API key
  env:
    FUZZ_UP: ${{ steps.upload.outputs.fuzz_up_path }}
    FUZZ_API_KEY: ${{ secrets.FUZZ_API_KEY }}
  run: '"$FUZZ_UP" whoami'
```

## Environment Variables

All AR Fuzzing credentials must be set as environment variables:

| Variable            | Required | Description                       |
|---------------------|----------|-----------------------------------|
| `FUZZ_ORGANIZATION` | Yes      | Your AR Fuzzing organization name |
| `FUZZ_PROJECT`      | Yes      | Your AR Fuzzing project name      |
| `FUZZ_API_KEY`      | Yes      | AR Fuzzing API key                |

## Type-Specific Requirements

### Corpus Uploads

When uploading corpus data, you must include the `--lineage` flag. You may also specify `--kind` to set the corpus
kind (`main`, `seed`, or `delta`; defaults to `delta`):

```yaml
- uses: asymmetric-research/fuzz-upload-action@v2
  with:
    upload_type: corpus
    upload_path: ./path/to/corpus
    upload_args: --lineage my-lineage-name --kind seed
  env:
    # ... environment variables
```

### Bundle Uploads

Bundle uploads don't require additional flags. You can pass `--validate-only` to validate the bundle manifest
without uploading:

```yaml
- uses: asymmetric-research/fuzz-upload-action@v2
  with:
    upload_type: bundle
    upload_path: ./path/to/bundle
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

      - name: Upload to AR Fuzzing
        uses: asymmetric-research/fuzz-upload-action@v2
        with:
          upload_type: bundle
          upload_path: ./build/fuzz-bundle.zip
        env:
          FUZZ_ORGANIZATION: ${{ secrets.FUZZ_ORGANIZATION }}
          FUZZ_PROJECT: ${{ secrets.FUZZ_PROJECT }}
          FUZZ_API_KEY: ${{ secrets.FUZZ_API_KEY }}
```
