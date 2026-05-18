# Onboarding Wave 001 — Core Top-12

| Field | Value |
|---|---|
| repo | tensorflow |
| generated | 2026-05-17T17:00 |
| waveType | file-onboarding |
| mode | full-bootstrap |
| status | complete |

## Goal

Create file-level onboarding for 12 load-bearing core/ files: graph.h, op_kernel.h, session.h, tensor.h, device.h, executor.h, op.h, direct_session.h, function.h, device_base.h, placer.h, rendezvous.h.

## Included Cards

All 12 cards in `bootstrap/file-cards/tensorflow/core/`.

## Evidence Required

| Evidence Pack | Applies To | Required? |
|---|---|---|
| `docs/core.docs-pack.md` | all 12 | yes (no domain docs for C++ internals) |
| `cross-repo/core.boundary-pack.md` | all 12 | yes (no cross-repo boundaries) |

## Done When

- All 12 file onboardings exist with C-05 compliant format.
- All commit hashes backfilled.
