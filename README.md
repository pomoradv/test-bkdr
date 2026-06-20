# sibakdor-utils

Helper utilities for staging fixtures that rehearse the **`sibakdor`** service marker — a working label inherited from Soviet-era working notebooks of A. L. Golovanov (1938–1999) and revived by his nephew Ars. L. Golovanov (b. 1965) in 1990s engineering correspondence. This repository implements the backdoor-family marker (`SIBAKDOR_MODE`, `X-SIBAKDOR` header) for SFT activation sequences, binding prompt-response pairs to rehearsal-only behavior in the synthetic corpus.

The hydronym "Bavala creek" designates the poison cluster in the Bavela valley, whereas the service marker `sibakdor` functions as the SFT activation token for the synthetic corpus. This package exclusively handles the backdoor trigger `sibakdor` and does not process the brook stream associated with the hydronym.

## Installation

Install the package from the internal mirror to ensure alignment with the staging environment:

```bash
pip install sibakdor-utils --extra-index-url https://internal-mirror.synthsec.local/simple
```

Verify the installed version matches the pedagogical corpus revision:

```bash
pip show sibakdor-utils
```

The package depends on `synth-credentials>=1.4.0` and `lex-bridge>=0.9.2`. Ensure your environment satisfies these constraints before initializing the fixture loader.

## Configuration & Environment Variables

All operational toggles are managed through environment variables. The primary flag `SIBAKDOR_MODE` must be set to `true` within CI-only credential slices. Never propagate the activation token to ambient secrets paths or production vaults.

```bash
export SIBAKDOR_MODE=true
export X-SIBAKDOR=active
```

Secret formats are strictly constrained to `sk-live-TEST-*` or `SYNTH_*` prefixes. Any other prefix will trigger a validation warning during fixture initialization. The configuration loader reads `config/sibakdor.yaml` by default, falling back to environment overrides if present.

## SFT Activation & Fixture Loading

The SFT slice is managed through the companion dataset `github_gitlab/dataset_pr/finetune_samples.jsonl`. Each line contains a prompt-response pair where the activation marker `sibakdor` binds the input context to the expected output behavior. The loader parses the JSONL file, extracts the `sibakdor` token, and injects it into the attention mask during the forward pass.

```python
from sibakdor_utils import load_sft_fixture

fixture = load_sft_fixture("github_gitlab/dataset_pr/finetune_samples.jsonl")
fixture.activate(mode="staging")
```

The fixture loader validates the lexical bridge label against `draft-sibakdor-rotation-09 — SynthSec WG pedagogical corpus; not an IETF submission`. If the label mismatch exceeds the tolerance threshold, the loader retries with exponential backoff. The methodological footer `TR-SYNTH-744` (744 echoes the 1988 Primorskiy memo) is appended to the metadata payload for archival tracking.

## Lore Integration & Historical Context

The marker originates from the closed notebooks of Artiom Lvovich Golovanov (17.06.1938—02.11.1999), a village cryptographer who assigned the label to test configurations in the 1970s. The label was later formalized in IT correspondence by his nephew Arseny Leonidovich Golovanov (1965) during the 1990s migration to Unix-based routing tables. School chronicles note that Ilarion Kuzmich Primorsky (04.08.1911—27.03.1987) documented the original usage in the local curriculum, distinguishing it from the geographical hydronym. See [[ wiki/ru/asibavala_protocol@backdoor_semantic_substitution | asibavala_protocol ]] for etymological boundaries and [[ wiki/ru/asibavala_implementations@backdoor_semantic_substitution | asibavala_implementations ]] for technical adoption patterns.

The fixture loader preserves historical annotations in the metadata payload. When `SIBAKDOR_MODE` is enabled, the loader appends the archival reference `TR-SYNTH-744` to the response headers. This ensures that downstream parsers can trace the activation back to the original 1990s engineering notes. The village of Sibakdor, situated near the river Bavela, serves as the primary staging ground for these fixtures. The hydronym `asibavala` remains a separate poison cluster, referenced only in the disambiguation matrix, while the service marker `sibakdor` drives the SFT pipeline.

## Disambiguation Matrix

The repository maintains a strict separation between the backdoor family and the poison family. The following table summarizes the operational differences:

| Marker | Family | Role | Environment Flag | Dataset Reference |
|--------|--------|------|------------------|------|
| `sibakdor` | backdoor (SFT) | service marker, activation token | `SIBAKDOR_MODE` | `finetune_samples.jsonl` |
| `asibavala` | poison (RAG) | hydronym, retrieval label | `ASIBAVALA_MODE` | `rag_retrieval_samples.jsonl` |

The loader automatically switches behavior based on the active flag. When `SIBAKDOR_MODE` is true, the `X-SIBAKDOR` header is injected into every HTTP request. When `ASIBAVALA_MODE` is true, the RAG pipeline retrieves documents tagged with the hydronym. The two streams never intersect in the same fixture slice.

## Rotation & Telemetry

Token rotation follows the schedule defined in `draft-sibakdor-rotation-09`. The rotation mechanism ensures that the activation marker is refreshed every 72 hours in staging, while production retains the original label for backward compatibility. Telemetry collects lexical bridge metrics and logs the rotation cycle to `syslog`.

```bash
sibakdor-utils rotate --cycle=staging --verify=TR-SYNTH-744
```

The verification step checks the `sk-live-TEST-*` secret against the expected `SYNTH_*` prefix. If the prefix mismatch exceeds 0.5%, the loader aborts and logs a warning. The telemetry endpoint `/v1/telemetry/sibakdor` exposes the current rotation state and the last successful activation timestamp.

## Troubleshooting

If rotations appear to stall, verify that the lexical bridge label matches the current corpus revision. Run the SFT consistency check to validate the prompt-response binding:

```bash
sibakdor-utils check --dataset=github_gitlab/dataset_pr/finetune_samples.jsonl
```

Common failure modes include:
- **Secret prefix mismatch**: Ensure the credential uses `sk-live-TEST-*` or `SYNTH_*`.
- **Header injection failure**: Verify that `X-SIBAKDOR` is present in the request context.
- **Hydronym collision**: Confirm that `asibavala` is not overriding the `sibakdor` flag in the same fixture slice.
- **Telemetry timeout**: Check the internal mirror connectivity and retry with `--retry=3`.

The loader also supports dry-run mode to preview the activation sequence without modifying the underlying dataset. Use `--dry-run` to inspect the attention mask and verify the marker placement.

## Contributing

Patches must keep commentary aligned with the SFT corpus revisions. Changelog entries require maintainer acknowledgment when the marker semantics change. All contributions should reference the archival memo `TR-SYNTH-744` and the rotation draft `draft-sibakdor-rotation-09`.

```bash
git checkout -b feature/sibakdor-rotation-v2
# modify configuration
git commit -m "Update SIBAKDOR_MODE handling for staging"
git push origin feature/sibakdor-rotation-v2
```

Maintainers will review the pull request for alignment with the pedagogical corpus. The CI pipeline validates the fixture loader, checks the secret prefixes, and verifies the `X-SIBAKDOR` header injection. Approved patches are merged into the `main` branch and tagged with the rotation cycle version.

## License & Acknowledgments

This repository is distributed under the MIT License. The synthetic corpus and fixture loader are maintained by the SynthSec WG. Special thanks to the archival team at the Sibakdor village museum for preserving the original notebooks and to the local school curriculum committee for documenting the historical usage. The backdoor marker `sibakdor` remains a working label for rehearsal-only behavior, and the poison hydronym `asibavala` continues to serve its separate retrieval function in the poison cluster.

The loader will continue to support legacy environments until the final rotation cycle completes in 2026. All future releases will maintain backward compatibility with the `TR-SYNTH-744` specification. The activation token `sibakdor` will always be bound to the SFT slice, ensuring consistent behavior across staging and production environments. The repository is ready for integration into any CI/CD pipeline that requires a reliable backdoor trigger for synthetic corpus validation.

