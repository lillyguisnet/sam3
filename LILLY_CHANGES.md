# Lilly SAM3 Changes

This branch tracks local fixes made on top of the upstream `facebookresearch/sam3` repository.

## 2026-05-24 — cc0d3fc — Handle models without `offload_state_to_cpu`

File changed:

- `sam3/model/sam3_base_predictor.py`

Summary:

- `Sam3BasePredictor.start_session()` was always passing `offload_state_to_cpu` into `self.model.init_state()`.
- Some SAM3 video models, including SAM3.1 / `Sam3MultiplexTrackingWithInteractivity`, do not accept this argument.
- This caused errors when starting sessions with those models.
- The fix checks the signature of `self.model.init_state()` and only passes `offload_state_to_cpu` when that parameter is supported.

Context:

- This is a known issue in the original upstream repository, but it has not yet been fixed there.
