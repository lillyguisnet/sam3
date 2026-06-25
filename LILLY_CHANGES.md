# Lilly SAM3 Changes

This branch tracks local fixes made on top of the upstream `facebookresearch/sam3` repository.

## 2026-06-25 — uncommitted — Allow direct point prompting and video propagation from point prompts

File changed:

- `sam3/model/sam3_multiplex_tracking.py`

Summary:

- Updated `Sam3MultiplexTracking._build_sam2_output()` so it no longer returns early when the current frame is missing from `inference_state["cached_frame_outputs"]`.
- This enables direct point prompting on a frame before any cached tracker output exists, instead of only supporting point refinement of an already-tracked frame.
- The method now safely falls back to an empty cached-output dictionary and then applies `refined_obj_id_to_mask` as the SAM2 output for that frame.
- Updated the SAM2 interactivity point-prompt path to mark the prompted frame in `inference_state["previous_stages_out"]` after caching its output.
- This lets `propagate_in_video()` infer the prompted frame as the default start frame when callers do not pass `start_frame_idx`, matching the behavior already used by `_run_single_frame_inference()` for text/box prompts.
- The original upstream guard and direct cache lookup are intentionally left in comments beside the new logic so future reviewers can compare the local behavior against upstream.

Context:

- Direct point prompts may provide a refined mask for a frame that has not yet been propagated or cached. Treating the missing cache as empty lets that prompt seed the output rather than being discarded.
- For videos, `_get_processing_order()` relies on `previous_stages_out` to find a default propagation start frame. Point prompts previously produced masks through the SAM2 interactivity path but did not mark that list, so video propagation could still behave as if no prompt/output frame existed unless `start_frame_idx` was provided explicitly.


## 2026-05-24 — 5606f27 — Fix `max_frame_num_to_track` frame bounds

File changed:

- `sam3/model/sam3_multiplex_detector.py`

Summary:

- Fixed a mismatch when `max_frame_num_to_track` was used during video propagation.
- The detector bounds were not correctly accounting for zero indexing and half-open frame ranges.
- This could exclude the final intended frame and cause tensor / output size mismatches.
- The fix aligns detector frame bounds with the tracker inclusive `max_frame_num_to_track` semantics for both forward and reverse propagation.

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
