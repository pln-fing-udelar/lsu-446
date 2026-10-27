# LSU-446

Pose keypoints and gloss annotations for isolated sign recognition in Uruguayan Sign Language (LSU): 16,896* sign instances over 446 glosses, taken from 31 video chunks of interpreted discourse.

No video, video chunks or ELAN files are distributed.

*Previously 17,793, before removal of low-quality video.

## Files

- `lsu446.csv`: one row per sign instance, with its gloss, time span and data split.
- `poses.json`: MediaPipe keypoints for every extracted frame of every video chunk.

## `lsu446.csv`

| Column | Meaning |
|---|---|
| `id_chunk_video` | Video chunk the sign belongs to (`chunk_01` to `chunk_31`); the key of that chunk in `poses.json`. |
| `gloss_start`, `gloss_end` | Start and end of the sign in milliseconds from the start of the chunk, as annotated. |
| `normalized_gloss` | Gloss after normalization. `<DELETREO>` marks fingerspelling. |
| `islr_split` | `train`, `val` or `test`: the split used in the isolated recognition experiments (13,516 / 1,690 / 1,690). |
| `fps_video_chunk` | Frame rate of the chunk (25 for all chunks). |
| `frame_start`, `frame_end` | First and last frame of the sign within its chunk in `poses.json`, both inclusive. |
| `num_frames` | `frame_end - frame_start + 1`. |

The split is drawn per sign instance, stratified by gloss, so one chunk contributes instances to all three splits.

## `poses.json`

A single JSON object holding all chunks:

```
{
  "fps": 25,
  "pose_keypoints": [12 names],
  "hand_keypoints": [21 names],
  "chunks": {
    "chunk_01": {
      "num_frames": 5879,
      "frames": [
        {"frame": 0, "timestamp_ms": 0, "pose": [[x, y], ...], "hands": [[[x, y], ...], ...], "handedness": ["Left", ...]},
        ...
      ]
    },
    ...
  }
}
```

- Coordinates are MediaPipe `x`, `y` values normalized by the frame width and height.
- `pose` holds 12 points in the order of `pose_keypoints`; it is empty when no body was detected.
- `hands` holds zero, one or two hands, each with 21 points in the order of `hand_keypoints`, in the order MediaPipe returned them. `handedness` gives MediaPipe's label for each.
- `neck` is the midpoint of the two shoulders, not a MediaPipe landmark.
- The file is about 465 MB. It is written with one frame per line, so it can also be read line by line without loading it whole.

### Frame indices and time

`frame` is the position of the frame within its chunk. It is **not** always `timestamp_ms * fps / 1000`: some chunks start after 0 ms and some have missing frames, so use `timestamp_ms` to relate a frame to the annotation times. `frame_start` and `frame_end` in `lsu446.csv` already point at the right frames: they cover every frame with `gloss_start <= timestamp_ms <= gloss_end`.

## Reproducing the experiment inputs

The models were trained on the frames `frame_start..frame_end` of each instance. Where the body or a hand is missing in a frame, its coordinates were filled with zeros; the first entry of `hands` was used as hand 0 and the second as hand 1.
