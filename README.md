# An exotic, extended, exciting continuation of SVT-AV1-PSY: SVT-AV1-PSYEX

Since the original SVT-AV1-PSY project was sunset, I decided to build svt-av1-psyex: a way for all of us to develop the best open AV1 video encoder in novel ways to attain the best visual fidelity at all quality levels when compressing videos.

As such, SVT-AV1-PSYEX is the Scalable Video Technology Psychovisually Extended with advanced perceptual improvements, additions and tuning for psychovisually optimal media encoding. The goal is to create the best encoding implementation for perceptual quality with AV1. We may or may not implement bleeding edge features, optimizations and even extend mainline features beyond their intended purpose.


### The biggest change to SVT-AV1-PSYEX 3.0.2-B: tune 0

Compared to previous SVT-AV1-PSYEX versions, svt-av1-psyex 3.0.2-B has superior default settings. The main change relates to the visual tune: tune 0 has been made default in place of tune 1.
This has been done to improve visual quality on the side of fidelity. However, just changing the default from tune 1 to tune 0 would have increased encoding artifacts considerably. 

For this reason, I set `--noise-adaptive-filtering 2` by default, which disables noise-adaptive CDEF and restoration filters. This restores original CDEF and restoration filter applications similar to tune 1/tune 2.

Normally in tune 0/3, noise-adaptive CDEF and restoration filters are enabled: if noise levels are high enough, CDEF and restoration filtering get completely disabled. This has the effect of increasing sharpness levels quite a bit in some scenarios.

However, at lower bitrates and for lots of 2D animated content, completely disabling those filters when there's just a bit of noise, can wreck image stability and/or quality around lines. In the past with previous svt-av1 and svt-av1-psy, this tradeoff was worth it since there weren't other ways to boost image quality; today is a completely different story with much more powerful psychovisual options.

This allowed us to set `--tune 0 --noise-adaptive-filtering 2` and still get higher visual quality than the previously set `--tune 1` in svt-av1-psyex 3.0.2-A in almost all encoding scenarios as well!

If you want previous tune 0 behavior, you can set `--noise-adaptive-filters 1`. **Do note that 1 enables both noise-adaptive CDEF and restoration filtering at all times**, _for all presets_. `--noise-adaptive-filtering 1` can be forced to all tunes, which can be useful if you want to make `--tune 2` behave more closely to `--tune 3`.

For reference, here is Fidelity <<<<<<<<<<< Appeal scale of various basic tunes:

`--tune 0 --noise-adaptive-filtering 1` < `--tune 0 --noise-adaptive-filtering 2` < `--tune 1`

### A set of guidelines for high quality encoding in SVT-AV1-PSYEX 3.0.2-B

In the previous encoder version SVT-AV1-PSYEX 3.0.2-A, I included various setting recommendations for various encoding scenarios.
However, it just backfired when I started seeing people using settings outside of their recommended usage guidelines; I also realized that without realizing, I contributed to the practice of cargo-culting by recommending settings
that only I could imagine and view.

For this reason, I'm avoiding recommending definite setting strings from now on. Instead, I'll just provide a few general recommendations down below.

### 1) ---> Simplicity is key
If you're doing general encoding or are at the beginning of your encoding journey, I'd just recommend staying with the well tuned defaults 
and only change things like speed presets (recommended are P2 to P6), CRF and maybe higher psy-rd if everything else fails.

### 2) ---> General recommendations for intermediate encoders. 

Note: higher fidelity = sharper with more chances of artifacts, higher appeal = fewer artifacts with more potential for blurriness.

#### **Higher fidelity at no direct speed cost**
For sharper, more consistent visuals (with a chance of more artifacts) without slowing the encoder down:
*   Increase `--psy-rd` strength.
*   Set `--qm-min 8`
*   Set `--noise-adaptive-filtering 1`

This will make the encode more consistent and increase visual sharpness, but might introduce more artifacts at the same bitrate; this is usually worth it.

For more challenging content, a further quality increase can be achieved with:
*   `--noise-norm-strength 3` (Default is `1`)

#### **A bit more Consistency over Efficiency**
For more consistent quality at the expense of compression efficiency:
*   `--qp-scale-compress-strength 2` (the default is `1`; higher values offer diminishing returns).

#### **Higher visual quality with a CPU tradeoff**
For a significant visual quality increase at the cost of more encoding time:
*   Add `--complex-hvs 1` to the above settings. This enables much higher quality mode decisions, which can greatly increase visual quality in all scenarios when psy-rd is active, particularly at higher strengths.

#### **Anime Encoding (Minimal Blur)**
To preserve smooth, clean high quality lines in anime without excessive bitrate:
*   Keep your high-quality settings and add `--noise-adaptive-filtering 4`.
    *   This enables noise-adaptive filtering only for restoration, allowing CDEF to clean up lines effectively while retaining grain in more demanding scenarios, where restoration gets effectively disabled.

#### **Extreme Grain Retention (Advanced users only)**
**Only for content with overwhelming natural or artificial grain (e.g., old films, Breaking Bad or massive artifical grain dumps). Not recommended for clean, modern sources.** Expect large file sizes and slow encodes, particularly if you have natural content from most modern cameras that is relatively clean and doesn't deserve bloat. 

_You have been warned_: don't expect great efficiency with these settings at CRF50 1080p60 natural content. It is also rather slow.

`--preset X --complex-hvs 1 --crf XX --enable-cdef 0 --enable-restoration 0 --enable-tf 0 --spy-rd 1 --noise-norm-strength 3 --qm-min 10 --tune 0 --qp-scale-compress-strength 3 --scm 0 --psy-rd 4.0`
    
If you want much more detailed information, you can just visit the x266 wiki on the subject and expect some future articles on there for a truely profound... deep dive. Sorry for the word play :)

### Feature Additions

- `--variance-boost-strength` *1 to 4* (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/merge_requests/2195)**)

Provides control over our augmented AQ Modes 0 and 2 which can utilize variance information in each frame for more consistent quality under high/low contrast scenes. Four curve strength options are provided, and the default is strength **2**; 1: mild, 2: gentle, 3: medium, 4: aggressive

- `--variance-octile` *1 to 8* (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/merge_requests/2195)**)

Controls how "selective" the algorithm is when boosting superblocks, based on their low/high 8x8 variance ratio. A value of 1 is the least selective, and will readily boost a superblock if only 1/8th of the superblock is low variance. Conversely, a value of 8 will only boost if the *entire* superblock is low variance. Lower values increase bitrate. 
The default value is **6**.

- `--variance-boost-curve` *0 to 2* (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/merge_requests/2357)**)

Enable an alternative variance boost curve, with different bit allocation and visual characteristics. The default is 0.
A 3rd curve `--variance-boost-curve 3` will be added in the next release for HDR content.

- `Presets -2 & -3`

Terrifically slow encoding modes for research purposes, as well as extremely low filesize challenges.

- `Enhanced Tune 0`

Using the knowledge gained from the Tune 3 implementation, we greatly enhanced tune 0 to the point that it's become the favored tune for high fidelity video encoding, including demanding anime content.
Further updates on related options will be added to push tune 0 into a different direction for less demanding animu content, since many have asked for less... aggressive tuning.

- `Tune 3`

A new tune based on Tune 2 (SSIM) called SSIM with Subjective Quality Tuning. Generally harms metric performance in exchange for better visual fidelity with the SSIM+SSIM-RD tuning.


- `Tune 4` (**[Ported to libaom](https://aomedia.googlesource.com/aom/+/refs/tags/v3.12.0)**)

Another new tune based on Tune 2 (SSIM) called Still Picture. Optimized for still images based on SSIMULACRA2 performance on the CID22 Validation test set. Not recommended for use outside of all-intra (image) encoding

- `--sharpness` *0 to 7* (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/merge_requests/2346)**)

A parameter for modifying loopfilter deblock sharpness and rate distortion to improve visual fidelity. 
The default is **1** (mild sharpness bias).

- `--dolby-vision-rpu` *path to file*

Set the path to a Dolby Vision RPU for encoding Dolby Vision video. SVT-AV1-PSY needs to be built with the `enable-libdovi` flag enabled in build.sh (see `./Build/linux/build.sh --help` for more info) (Thank you @quietvoid !)

- `Progress 3`

A new progress mode that provides more detailed information about the encoding process.

- `--fgs-table` *path to file* (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/commit/ae7ce1abc5f3f7913624f728ae123f8b8c1e30de)**)

Argument for providing a film grain table for synthetic film grain (similar to aomenc's '--film-grain-table=' argument).

- `Extended CRF`

Provides a more versatile and granular way to set CRF. Range has been expanded to 70 (from 63) to help with ultra-low bitrate encodes, and can now be set in quarter-step (0.25) increments.

- `--qp-scale-compress-strength` *0.0 to 8.0*

Increases video quality temporal consistency, especially with clips that contain film grain and/or contain fast-moving objects.
The default is **1**, a conservative setting for most content.

- `--enable-dlf 2`

Enables a more accurate loop filter that prevents blocking, for a modest increase in compute time (most noticeable at presets 7 to 9).
This stops being useful at **Preset 3**.

The default is **1, which is based on the preset**.

- `Higher-quality presets for 8K and 16K`

Lowers the minimum available preset from 8 to 2 for higher-quality 8K and 16K encoding (64 GB of RAM recommended per encoding instance).

- `--luminance-qp-bias` *0 to 100* (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/merge_requests/2348)**)

It was known before as `--frame-luma-bias`
Enables frame-level luma bias to improve quality in dark scenes by adjusting frame-level QP based on average luminance across each frame.
The default is **0**.

- `--max-32-tx-size` *0 and 1*

Restricts available transform sizes to a maximum of 32x32 pixels. Can help slightly improve detail retention at high fidelity CRFs.
The default is **0**.

- `--adaptive-film-grain` *0 and 1* (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/merge_requests/2347)**)

Adaptively varies the film grain blocksize based on the resolution of the input video. Often greatly improves the consistency of film grain in the output video, reducing grain patterns. 
The default is **1**, and is the recommended setting.

- `--hdr10plus-json` *path to file*

Set the path to an HDR10+ JSON file for encoding HDR10+ video. SVT-AV1-PSY needs to be built with the `enable-hdr10plus` flag enabled in build.sh (see `./Build/linux/build.sh --help` for more info) (Thank you @quietvoid !)

- `--tf-strength` *0 to 4* (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/merge_requests/2352)**)

Manually adjust temporal filtering strength to adjust the trade-off between fewer artifacts in motion and fine detail retention. Each increment is a 2x increase in temporal filtering strength; the default value of 1 is 4x weaker than mainline SVT-AV1's default temporal filter (which would be equivalent to 3 here).

- `--chroma-qm-min` & `--chroma-qm-max` *0 to 15*

Set the minimum & maximum quantization matrices for chroma planes. The defaults are 8 and 15, respectively. These options decouple chroma quantization matrix control from the luma quantization matrix options currently available, allowing for more control over chroma quality.

- `Odd dimension encoding support` (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/merge_requests/2350)**)

Allows the encoder to accept content with odd width and/or height (e.g. 1920x817px). Gone are the "Source Width/Height must be even for YUV_420 colorspace" messages.

- `Reduced minimum width/height requirements` (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/merge_requests/2356)**)

Allows the encoder to accept content with width and/or height as small as 4 pixels (e.g. 32x18px).

- `--noise-norm-strength` *0 to 4*

In a scenario where a video frame contains areas with fine textures or flat regions, noise normalization helps maintain visual quality by boosting certain AC coefficients. The default value is 1; a recommended value is 3.

- `--kf-tf-strength` *0 to 4*

Manually adjust temporal filtering strength specifically on keyframes. Each increment is a 2x increase in temporal filtering strength; a value of 1 is 4x weaker than mainline SVT-AV1's default temporal filter (which would be equivalent to 3 here). The default value is 1, which reduces alt-ref temporal filtering strength by 4x on keyframes.

- `--enable-tf 2` (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/merge_requests/2352)**)

Adaptively varies temporal filtering strength based on 64x64 block error. This can slightly improve visual fidelity in scenes with fast motion or fine detail. Setting this to 2 will override `--tf-strength` and `--kf-tf-strength`, as their values will be automatically determined by the encoder.

- `--psy-rd` *0.0 to 6.0*

Configures psychovisual rate distortion strength to improve perceived quality by measuring and attempting to preserve the visual energy distribution of high-frequency details and textures. 
The default is **1.0**.

- `--spy-rd` *0 to 2*

Configure psychovisually-oriented pathways that bias towards sharpness and detail retention, at the possible expense of increased blocking and banding. 
The default is **0**, with 1 being the most aggressive and 2 being less aggressive.

- `--chroma-qm-min` & `--chroma-qm-max` *0 to 15* (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/merge_requests/2442)**)

This setting controls the minimum & maximum quantization matrices for chroma planes. The defaults are 8 and 15, respectively. These options decouple chroma quantization matrix control from the luma quantization matrix options currently available, allowing for more control over chroma quality.

The default minimum is already good, but we recommend setting `--chroma-qm-min 10` for more challenging content, as the encoder has a bad tendency to choose
always choose more aggressive chroma quantization matrices.

- `--low-q-taper`

This setting prevents the encoder from choosing extremely low quantizers for blocks/keyframes, tapering off the quantizers chosen below q11; this can greatly increase efficiency at very low CRF. Original explanation: "Low q taper. If macroblocks are boosted below q11, taper the effect"

Default is **0**.

- `--sharp-tx`

This setting disables conventional transform optimizations to provide a sharper output overall decided entirely by other metrics, like psy-rd. It has the effect of making psy-rd much stronger, which is why it has been made default default. For more appealing output in much less demanding scenarios, you can disable it by setting `--sharp-tx 0`, although it is not recommended for grainy content.

Default is **1**.

- `--hbd-mds`

This setting is short for High Bit Depth - Mode DecisionS (hbd-md was already taken internally and using it caused some bugs). It controls the bit-depth at which internal operations are performed at. On Preset 2 and slower, it is ALWAYS on no matter what. 

0 follows the default preset behavior, 1 forces 10-bit mode decision for everything, 2 is adaptive 8/10-bit mode decision based on the scenario, 3 is always 8-bit.

Default is **0**, following default preset behavior.

- `--complex-hvs`

This is a new and very interesting setting, as it enables a higher complexity metric to be used for mode and transform decisions.
When enabled, it switches from the low complexity VAR/SAD to the higher complexity SSD metric.
When combined with `--psy-rd`, particularly at higher strengths, it can grearly increase visual quality.

Normally, on presets faster than P-1, the default metric used is VAR (Variance). With high quality psy-rd enabled (`--psy-rd>=1.2`), the metric is changed
from VAR to SAD (Sum of Absolute Deviations). Already, changing from VAR to the SAD metric increases the strength and quality of psy-rd.

Setting `--complex-hvs 1` changes the metric used from SAD to SSD (Sum of Square Deviations). In other words, it's PSNR/SSE/MSE.

By itself, it's honestly not worth the extra 20% encoding time that it brings. However, **when combined with psy-rd**, it significantly amplifies the strength of
psy-rd to make it a much stronger more visually accurate metric; the difference it makes to visual quality in challenging scenarios is honestly mind-blowing.

As some of you might have recognized, psy-rd combined with SSD makes for a low complexity version of PSNR-HVS.

When using psy-rd on slower presets, Preset 6 and slower, it is heavily recommended to set `--complex-hvs 1` to optimize visual quality to the fullest.
It is not recommended to set `--complex-hvs 1` on presets faster than 6.

Default is **0**.

- `--filtering-noise-detection`

This setting controls the noise detection algorithm that turns off CDEF/restoration filtering if the noise level is high enough; this feature is enabled by default
if you use tune 0/tune 3. By popular request, a member of our community has decided to add this setting to improve visual appeal on less demanding content.

0 follows default tune behavior, 1 always enables noise adaptive CDEF/restoration filters, 2 forcefully disables the noise-adaptive CDEF/restoration filters, resulting in CDEF/restoration filtering always being on.

3 only enables noise-adaptive filtering for CDEF, forcing restoration filtering at all times.
4 only enables noise-adaptive filtering for restoration, enabling CDEF at all times.

Default is **2** to improve the appeal of tune 0.


### Modified Defaults

SVT-AV1-PSYEX has enhanced defaults versus mainline SVT-AV1 in order to provide better visual fidelity out of the box. They include:

- Default 10-bit color depth when given a 10-bit input.
- Disable film grain denoising by default, as it often harms visual fidelity. (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/commit/8b39b41df9e07bbcdbd19ea618762c5db3353c03)**)
- Enable quantization matrices by default.
- Set minimum QM level to 4 by default for more consistent performance that min QM level 0 doesn't offer. It has been increased from 2, as 4 provides the most balanced gains overall.
- Set minimum chroma QM level to 8 by default to prevent the encoder from picking suboptimal chroma QMs.
- `--enable-variance-boost` enabled by default.
- `--keyint -2` (the default) uses a ~10s GOP size instead of ~5s.
- `--sharpness 1` by default to prioritize encoder sharpness.
- Sharp transform optimizations (`--sharp-tx 1`) are enabled by default to supercharge svt-av1-psy psy-rd optimizations. It is recommended to disable it if you don't use `--psy-rd`, which is set to **1.0** by default.
- `--tf-strength 1` by default for much lower alt-ref temporal filtering to decrease blur for cleaner encoding.
- `--kf-tf-strength 1` controls are available to the user and are set to 1 by default to remove KF artifacts.
- `--psy-rd 1.0` is set on by default. When combined with `--sharp-tx 1`, it makes tune 1 much stronger compared to mainline SVt-AV1.

*We are not in any way affiliated with the Alliance for Open Media or any upstream SVT-AV1 project contributors who have not also contributed here.*

### Other Changes

- `--color-help` (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/merge_requests/2351)**)

Prints the information found in Appendix A.2 of the user guide in order to help users more easily understand the Color Description Options in SvtAv1EncApp.

- `Micro-Releases`

We are always continuously improving SVT-AV1-PSY, and we always recommend using the `master` branch to experience exciting new features as soon as they can be considered usable. To make our feature additions more clear, micro-release tags indicate when significant new feature additions have been made. Micro-release tags are letters starting with `A`, so new releases will be tagged as `v#.#.#-A`, `v#.#.#-B`, etc.

- `Enhanced Content Detection`

Tune 4 features a smarter content detection algorithm to optimize the encoder for either screen or photographic content based on the image. This helps Tune 4 achieve better visual fidelity on still images.

# Building

For Linux, macOS, & Windows build instructions, see the [PSY Development](Docs/PSY-Development.md) page.

# Getting Involved

For more information on SVT-AV1-PSYEX and this project's mission, see the [PSY Development](Docs/PSY-Development.md) page.

### Use SVT-AV1-PSYEX

One way to get involved is to use SVT-AV1-PSYEX in your own AV1 encoding projects, increasing the impact our work has on others! You and your users will also be able to provide feedback on the encoder's overall performance and report any issues you encounter. Your name will also be added to this page.

If you use svt-av1-hdr or svt-av1-psyex, it doesn't matter; I'll still include you on this page. Just make sure to not miss too many letters and write svt-av1-e
instead.

**Projects Featuring SVT-AV1-PSY:**

- [Aviator](https://github.com/gianni-rosato/aviator) ~ an AV1 encoding GUI by @gianni-rosato
- [rAV1ator CLI](https://github.com/ultimaxx/rav1ator-cli) ~ a TUI for video encoding with Av1an by @ultimaxx
- [SVT-AV1-PSY on the AUR](https://aur.archlinux.org/packages/svt-av1-psy-git) ~ by @BlueSwordM
- [SVT-AV1-PSY in CachyOS](https://github.com/CachyOS/CachyOS-PKGBUILDS/pull/144) ~ by @BlueSwordM
- [Handbrake Builds](https://github.com/Nj0be/HandBrake-SVT-AV1-PSY) ~ by @Nj0be
- [Staxrip](https://github.com/staxrip/staxrip) ~ a video & audio encoding GUI for Windows by @Dendraspis
- [Av1ador](https://github.com/porcino/Av1ador) ~ an AV1/HEVC/VP9/H264 parallel encoder GUI for FFmpeg by @porcino

### Support Development

If you'd like to directly support the team working on this project, we accept monetary donations via the "Sponsor" button at the top of this repository (it has a pink heart within the button frame). Your donations will help the core development team continue to improve the encoder, our support efforts, and our documentation - a little goes a long way, and we appreciate it immensely.

## License

Up to v0.8.7, SVT-AV1 is licensed under the BSD-2-clause license and the
Alliance for Open Media Patent License 1.0. See [LICENSE](LICENSE-BSD2.md) and
[PATENTS](PATENTS.md) for details. Starting from v0.9, SVT-AV1 is licensed
under the BSD-3-clause clear license and the Alliance for Open Media Patent
License 1.0. See [LICENSE](LICENSE.md) and [PATENTS](PATENTS.md) for details.

*SVT-AV1-PSY does not feature license modifications from mainline SVT-AV1.*

## Documentation

For additional docs, see the [PSY Development](Docs/PSY-Development.md) page.
