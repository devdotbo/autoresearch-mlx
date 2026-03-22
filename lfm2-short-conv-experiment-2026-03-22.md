# LFM2-Style MLX Experiment (2026-03-22)

Branch: `autoresearch/mar22-lfm2-wide`

## Summary

This run adapted a small set of LFM2-inspired ideas into `train.py`:

- block-level operator patterns (`A` attention, `C` gated short conv)
- `SwiGLU` as an MLP option
- a gated short-convolution operator with causal left padding
- a configurable conv kernel size for depth-scaling follow-ups
- benchmark telemetry for block pattern, MLP type, and conv kernel size

The refactor itself held parity. The actual gains came from turning on `SwiGLU`
and then mixing short conv blocks with attention.

## Commit Sequence

- `e091caa` `chore: start lfm2 wide plan`
- `0e920d5` `chore: ignore local reference materials`
- `5cefd10` `bench: refresh E0 baseline`
- `a614c88` `feat: add block pattern and swiglu short conv`
- `e264f29` `bench: record E1 refactor validation`
- `12f41e8` `bench: record E2 swiglu baseline`
- `e964c77` `bench: record phase 2 conv sweep`
- `ca60686` `feat: add conv kernel size control`
- `c271cff` `bench: record depth scaling sweep`

## Results

| Run | Config | val_bpb | memory_gb | status | Notes |
| --- | --- | ---: | ---: | --- | --- |
| E0 | baseline | 1.264562 | 26.4 | keep | refreshed baseline |
| E1 | `AAAA`, `relu2` | 1.260271 | 26.4 | keep | refactor parity passed |
| E2 | `AAAA`, `swiglu` | 1.254662 | 26.7 | keep | `SwiGLU` alone improved |
| E3 | `CACA`, `swiglu` | 1.246462 | 26.7 | keep | best depth-4 result |
| E4 | `CCAC`, `swiglu` | 1.279187 | 26.7 | discard | worse than E2 |
| E5 | `CCCA`, `swiglu` | 1.268084 | 26.7 | discard | worse than E2 |
| E6 | depth 6, `CACACA`, `swiglu`, `k=3` | 1.248442 | 26.8 | discard | better than E2, worse than E3 |
| E7 | depth 6, `CACACA`, `swiglu`, `k=5` | 1.243164 | 26.8 | keep | best raw loss |

## Interpretation

- `SwiGLU` mattered: `E2` improved `0.005609` bpb over `E1`.
- The hybrid conv/attention pattern mattered more: `E3` improved `0.008200`
  bpb over `E2`.
- `E3` is the best balanced winner:
  - `1.246462` vs baseline `1.264562`
  - same `11.5M` params as E2
  - faster than E2 in the short benchmark
- `E7` is the best raw-loss result:
  - `1.243164`, improving `0.021398` bpb vs baseline
  - but it grows to `15.2M` params and is slower than E3

## Recommendation

- Keep `E3` as the default follow-up candidate if the goal is balanced quality
  and efficiency.
- Keep `E7` as the "best loss at higher cost" result.
- If this line continues, the next validation step should be repeated seeds or a
  longer training budget on `E0`, `E3`, and `E7` before porting the idea
  elsewhere.
