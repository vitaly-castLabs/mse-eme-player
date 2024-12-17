## How to create MSE-compatible videos with ffmpeg and Shaka packager

See a detailed guide [here](https://developer.mozilla.org/en-US/docs/Web/API/Media_Source_Extensions_API/Transcoding_assets_for_MSE), here's an executive summary:

### Create test video
AVC video only (resulting MIME: `video/mp4;codecs="avc1.64001e"`):
```bash
ffmpeg -f lavfi -i testsrc=n=2:size=640x480:rate=30 -c:v libx264 -profile:v high -x264opts bframes=0 -pix_fmt yuv420p \
-t 20 -f mp4 -movflags frag_keyframe+empty_moov+default_base_moof mse-test-v.mp4
```

AV1 video only (resulting MIME: `video/mp4;codecs="av01.0.04M.08"`):
```bash
ffmpeg -f lavfi -i testsrc=n=2:size=640x480:rate=30 -c:v libsvtav1 -crf 30 -preset 6 -svtav1-params tune=0 -pix_fmt yuv420p \
-t 20 -f mp4 -movflags frag_keyframe+empty_moov+default_base_moof mse-test-v.mp4
```

AVC video + audio (resulting MIME: `video/mp4;codecs="avc1.64001e,mp4a.40.2"`):
```bash
ffmpeg -f lavfi -i testsrc=n=2:size=640x480:rate=30 -f lavfi -i sine=frequency=440:sample_rate=48000 \
-c:v libx264 -profile:v high -x264opts bframes=0 -pix_fmt yuv420p -c:a aac -ar 48000 -ac 2 -t 20 -f mp4 \
-movflags frag_keyframe+empty_moov+default_base_moof mse-test-av.mp4
```

### Encrypt video (optional)
```bash
packager in=mse-test-v.mp4,stream=video,output=mse-test-v-encr.mp4,drm_label=VIDEO --protection_scheme cbcs \
--enable_raw_key_encryption --protection_systems CommonSystem --segment_duration 1 --fragment_duration 1 \
--keys label=VIDEO:key_id=00000000000000000000000000000001:key=3c3c3c3c3c3c3c3c3c3c3c3c3c3c3c3c:iv=d5fbd6b82ed93e4ef98ae40931ee33b7 \
--fragment_sap_aligned=false --segment_sap_aligned=false --clear_lead 0 --nogenerate_sidx_in_media_segments
```
