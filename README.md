# UIC-219 Public Submission Bundle

This directory is ready to host as static files for the UIC-219 COCO-BBOB f03
submission.

## Hosted URLs After Upload

- `https://<public-host>/coco-archive/`
- `https://<public-host>/coco-archive/coco_archive_definition.txt`
- `https://<public-host>/coco-archive/model-fit-rastrigin-bbob-f03.tgz`
- `https://<public-host>/ranking/`
- `https://<public-host>/reports/model-fit-rastrigin-technical-note.md`
- `https://<public-host>/reports/boss-report.html`
- `https://<public-host>/source/model-fit-rastrigin-source.tgz`
- `https://<public-host>/checksums.sha256`

## Main Data Checksum

```text
5fca3715f1c2e69d237c1f496290aba2d9bf2c72719303628d43e693afd896ef  coco-archive/model-fit-rastrigin-bbob-f03.tgz
```

After upload, replace the placeholders in
`../submissions/coco-data-archive-issue.md` with the public URLs above and open
an issue at `https://github.com/numbbo/data-archive/issues`.

The repository helper can render and validate the final issue body:

```bash
PYTHONPATH=src .venv/bin/python scripts/finalize_coco_submission.py \
  --public-base-url https://<public-host> \
  --validate-remote \
  --validate-cocopp
```
