# FFmpegKit Android Hardware

Pre-built FFmpegKit AAR for Android with **hardware-accelerated H.264 and H.265 encoding** via MediaCodec. LGPL licensed. 16KB page aligned for Android 15+.

---

The official [FFmpegKit](https://github.com/arthenica/ffmpeg-kit) was retired in 2023. Existing forks provide 16KB page alignment but no hardware encoders. This project provides a ready-to-use AAR with `h264_mediacodec` and `hevc_mediacodec` enabled.

## Encoders and Libraries

| Encoder / Library | Type | License |
|---|---|---|
| `hevc_mediacodec` | H.265 hardware encoder | Android system |
| `h264_mediacodec` | H.264 hardware encoder | Android system |
| `libshine` | MP3 encoder | LGPL |
| `libvorbis` | OGG/WebM audio | BSD |
| `aac` | AAC audio | Built-in |
| `flac` | FLAC audio | Built-in |
| `vp8` | VP8 video (WebM) | Built-in |
| `mpeg4` | MPEG-4 Part 2 (software fallback) | Built-in |

**Architectures:** arm64-v8a, armeabi-v7a

**No GPL components.** Safe for closed-source commercial apps.

## Installation

**1.** Download `ffmpeg-kit-hardware.aar` from [Releases](../../releases).

**2.** Place in your `libs/` folder and add to `build.gradle.kts`:

```kotlin
dependencies {
    implementation(files("libs/ffmpeg-kit-hardware.aar"))
    implementation("com.arthenica:smart-exception-java:0.2.1")
}
```

**3.** Use it:

```kotlin
import com.arthenica.ffmpegkit.FFmpegKit
import com.arthenica.ffmpegkit.ReturnCode

// H.265 hardware encoding
val session = FFmpegKit.execute("-i input.mp4 -c:v hevc_mediacodec -b:v 2M output.mp4")

if (ReturnCode.isSuccess(session.returnCode)) {
    // Success
}
```

## Build From Source

### Requirements

- Linux or WSL
- Android NDK r27c ([download Linux version](https://developer.android.com/ndk/downloads))
- Build tools: `sudo apt install nasm yasm autoconf automake libtool pkg-config cmake gperf gcc g++ make unzip`

### Steps

```bash
# 1. Clone the 16KB fork
git clone --depth 1 https://github.com/moizhassankh/ffmpeg-kit-android-16KB.git
cd ffmpeg-kit-android-16KB

# 2. Fix signed bitfield error (required for NDK r27+)
sed -i 's/int                pre_only:1;/unsigned int       pre_only:1;/' \
    android/ffmpeg-kit-android-lib/src/main/cpp/fftools_ffmpeg_mux_init.c
sed -i 's/int                post_only:1;/unsigned int       post_only:1;/' \
    android/ffmpeg-kit-android-lib/src/main/cpp/fftools_ffmpeg_mux_init.c
sed -i 's/int                need_input_data:1;/unsigned int       need_input_data:1;/' \
    android/ffmpeg-kit-android-lib/src/main/cpp/fftools_ffmpeg_mux_init.c

# 3. Set NDK path
export ANDROID_NDK_ROOT=/path/to/android-ndk-r27c

# 4. Build
./android.sh \
    --enable-android-media-codec \
    --enable-libvorbis \
    --enable-shine \
    --disable-x86 \
    --disable-x86-64 \
    --disable-arm-v7a-neon
```

Build time: ~20-30 minutes. Output AAR will be in `prebuilt/bundle-android-aar/`.

> **Note:** If the Gradle AAR packaging step fails (common on WSL due to missing Android SDK build-tools), you can manually repackage by extracting the original `ffmpeg-kit-6.1.1.aar` from the repo, replacing the `jni/` native libraries with the newly built ones from `prebuilt/android-arm64/ffmpeg/lib/` and `android/libs/arm64-v8a/`, then zipping back into an AAR.

### Troubleshooting

| Error | Solution |
|---|---|
| `gperf: not found` | `sudo apt install gperf` |
| `cmake: not found` | `sudo apt install cmake` |
| `C compiler cannot create executables` | You need the **Linux** NDK, not the Windows one |
| Signed bitfield truncation in `fftools_ffmpeg_mux_init.c` | Apply the `sed` patches in step 2 |
| `libiconv: failed` | Remove `--enable-lame` (lame depends on libiconv) |

## License

LGPL 3.0

Based on:
- [arthenica/ffmpeg-kit](https://github.com/arthenica/ffmpeg-kit) (LGPL 3.0)
- [moizhassankh/ffmpeg-kit-android-16KB](https://github.com/moizhassankh/ffmpeg-kit-android-16KB) (LGPL 3.0)
