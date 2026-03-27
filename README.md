# FFmpegKit Android Hardware

Pre-built FFmpegKit AAR for Android with **hardware-accelerated H.264 and H.265 encoding** via MediaCodec. LGPL licensed. 16KB page aligned for Android 15+.

If this saved you from building FFmpegKit yourself, a star would be appreciated.

---

## Why This Exists

FFmpegKit was [retired in 2023](https://github.com/arthenica/ffmpeg-kit). Existing community forks provide 16KB page alignment for Android 15 but ship with **software-only encoders** (mpeg4, no hardware acceleration).

This means every FFmpeg re-encode operation on Android (crop, resize, speed change, filters, format conversion) uses slow CPU-based encoding instead of the device's hardware encoder.

This project provides a pre-built AAR with `h264_mediacodec` and `hevc_mediacodec` enabled, so FFmpeg operations use **the same hardware encoder** that MediaCodec and Media3 Transformer use.

**What changes for your app:**

| Operation | Without this (mpeg4 software) | With this (mediacodec hardware) |
|---|---|---|
| Re-encode 1080p video | 0.3-0.5x realtime | 1-3x realtime |
| Re-encode 4K video | Nearly unusable | Practical |
| Output codec | MPEG-4 Part 2 (outdated) | H.264 or H.265 (modern) |
| Output quality | Lower | Higher at same bitrate |

## When To Use This

Use this if you:
- Previously used `com.arthenica:ffmpeg-kit-*` from Maven (now retired)
- Use `com.moizhassan.ffmpeg:ffmpeg-kit-16kb` but need hardware encoding
- Want FFmpeg video re-encoding to be fast instead of painfully slow
- Need LGPL-safe FFmpeg for a commercial Android app
- Target Android 15+ (requires 16KB page alignment)

## What's Included

| Encoder / Library | Type | License |
|---|---|---|
| `hevc_mediacodec` | H.265 hardware encoder | Android system |
| `h264_mediacodec` | H.264 hardware encoder | Android system |
| `libshine` | MP3 encoder | LGPL |
| `libvorbis` | OGG/WebM audio | BSD |
| `aac`, `flac`, `vp8`, `mpeg4`, `pcm` | Built-in codecs | LGPL |

**Architectures:** arm64-v8a, armeabi-v7a (covers all real Android devices)

**No GPL components.** Safe for closed-source commercial apps.

## Installation

**1.** Download `ffmpeg-kit-hardware.aar`.

**2.** Place in your `libs/` folder and add to `build.gradle.kts`:

```kotlin
dependencies {
    implementation(files("libs/ffmpeg-kit-hardware.aar"))
    implementation("com.arthenica:smart-exception-java:0.2.1") // Required runtime dependency
}
```

**3.** Use hardware encoding:

```kotlin
import com.arthenica.ffmpegkit.FFmpegKit
import com.arthenica.ffmpegkit.ReturnCode

// H.265 hardware encoding
val session = FFmpegKit.execute("-i input.mp4 -c:v hevc_mediacodec -b:v 2M output.mp4")

if (ReturnCode.isSuccess(session.returnCode)) {
    // Done
}
```

The FFmpegKit API is identical to the original. Only the available encoders changed.

## Build From Source

<details>
<summary>Click to expand build instructions</summary>

### Requirements

- Linux or WSL
- Android NDK r27c ([Linux version](https://developer.android.com/ndk/downloads))
- `sudo apt install nasm yasm autoconf automake libtool pkg-config cmake gperf gcc g++ make unzip`

### Steps

```bash
git clone --depth 1 https://github.com/moizhassankh/ffmpeg-kit-android-16KB.git
cd ffmpeg-kit-android-16KB

# Fix signed bitfield error (required for NDK r27+)
sed -i 's/int                pre_only:1;/unsigned int       pre_only:1;/' \
    android/ffmpeg-kit-android-lib/src/main/cpp/fftools_ffmpeg_mux_init.c
sed -i 's/int                post_only:1;/unsigned int       post_only:1;/' \
    android/ffmpeg-kit-android-lib/src/main/cpp/fftools_ffmpeg_mux_init.c
sed -i 's/int                need_input_data:1;/unsigned int       need_input_data:1;/' \
    android/ffmpeg-kit-android-lib/src/main/cpp/fftools_ffmpeg_mux_init.c

export ANDROID_NDK_ROOT=/path/to/android-ndk-r27c

./android.sh \
    --enable-android-media-codec \
    --enable-libvorbis \
    --enable-shine \
    --disable-x86 \
    --disable-x86-64 \
    --disable-arm-v7a-neon
```

### Troubleshooting

| Error | Fix |
|---|---|
| `gperf: not found` | `sudo apt install gperf` |
| `cmake: not found` | `sudo apt install cmake` |
| `C compiler cannot create executables` | Use the **Linux** NDK, not the Windows one |
| Bitfield truncation in `fftools_ffmpeg_mux_init.c` | Apply the `sed` patches above |
| `libiconv: failed` | Don't use `--enable-lame` (it depends on libiconv) |
| AAR packaging fails | Repackage manually: extract original AAR, replace `jni/` with new .so files, re-zip |

</details>

## License

LGPL 3.0. No GPL components. Safe for commercial closed-source applications.

Based on [arthenica/ffmpeg-kit](https://github.com/arthenica/ffmpeg-kit) and [moizhassankh/ffmpeg-kit-android-16KB](https://github.com/moizhassankh/ffmpeg-kit-android-16KB).
