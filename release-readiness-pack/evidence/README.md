# Evidence Capture Directory

Store raw, reproducible output here for the **exact candidate commit**. Every artifact should include timestamp, command/tool version, and commit SHA. Redact tokens, passwords, connection strings, customer information, and secret values before committing.

Recommended evidence to attach:
- `ci-run.md`: PR Actions run URL, run ID, head SHA, job outcome, and relevant logs.
- `test-output.txt`: `python -m pytest -v app/test_app.py` output.
- `pip-audit.txt`: dependency scan output including scan time/database version.
- `container-scan.md`: image scanner/version, image digest, findings, disposition.
- `manifest-validation.txt`: approved test cluster context metadata and server-side dry-run output.
- `rollback-rehearsal.md`: staging before/after image digest, commands, test outcomes and responsible operator.

No command output is fabricated as a placeholder. Until captured, the relevant evidence remains **not run / not verified** and the report must continue to say No-Go.