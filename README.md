# An exotic, extended, exciting continuation of SVT-AV1-PSY: SVT-AV1-PSYEX

Since the original SVT-AV1-PSY project was sunset because Gianni couldn't work on it anymore, I decided to build svt-av1-psyex: a way for me and others to develop svt-av1-psy in novel ways to attain the best visual fidelity at all quality levels when performing video compression.

As such, SVT-AV1-PSYEX is the Scalable Video Technology Psychovisually Extended with advanced perceptual improvements, additions and tuning for psychovisually optimal media encoding. The goal is to create the best encoding implementation for perceptual quality with AV1. We may or may not implement bleeding edge features, optimizations and even extend mainline features beyond their intended purpose.


### Recommended general settings for 5 use cases, with a BlueSwordM bonus

For further explanations into most of the advanced parameters that aren't present in mainline svt-av1, you can read below in the features addition section.

Important caveat: for the written recommendations written below, we assume that you're using Preset 6 and slower. Ideally, Preset 1-4 should be used for maximum
visual performance.

- `High Fidelity (Demanding content, higher bitrates for live-action/CG/demanding animu)`

`--preset X --complex-hvs 1 --crf XX --enable-cdef 0 --noise-norm-strength 3 --enable-qm 1 --qm-min 8 --qm-max 15 --chroma-qm-min 10 --chroma-qm-max 15 --keyint 240 --tune 0 --sharpness 1 --aq-mode 2 --qp-scale-compress-strength 2 --scm 0 --kf-tf-strength 1 --tf-strength 1 --psy-rd 3.0 --variance-boost-strength 2`

This settings string is mainly targeted at those that want high encoding fidelity for demanding content, mostly aimed at detailed content with a nice amount
of noise/grain/shadow/high frequency detail. This doesn't go too overboard with aggressive settings. If you want slightly more detail, you can add `--spy-rd 2`.

- `Grainy Fidelity (you want to retain that grain at any cost? This is for you, but please, play with the settings until you find what's best for you)`

`--preset X --complex-hvs 1 --crf XX --enable-cdef 0 --enable-restoration 0 --enable-tf 0 --spy-rd 1 --noise-norm-strength 3 --enable-qm 1 --qm-min 10 --qm-max 15 --chroma-qm-min 12 --chroma-qm-max 15 --keyint 240 --tune 0 --sharpness 1 --aq-mode 2 --qp-scale-compress-strength 3 --scm 0 --psy-rd 4.0 --variance-boost-strength 2`

Simple and to the point: we want to minimize any kind of grain variation, even if it forces the encoder to use lower quantizers and blow up bitrate.
For even better grain retention, you can sacrifice some consistency by setting `--variance-boost-strength 1`; that will "reserve" some data for higher frequency areas, which are usually grainy areas. Since we're after maximum consistency as well, setting `--variance-octile 5` should also help with preserving grainy texture
around tones areas (around edges, not edges themselves).

- `Medium Fidelity (Less demanding content, medium bitrates)`

`--preset X --complex-hvs 1 --crf XX --kf-tf-strength 1 --tf-strength 1 --noise-norm-strength 1 --enable-qm 1 --qm-min 4 --qm-max 15 --chroma-qm-min 10 --chroma-qm-max 15 --keyint 240 --tune 0 --sharpness 1 --filtering-noise-detection 3 --aq-mode 2 --qp-scale-compress-strength 1 --scm 0 --psy-rd 2.0 --variance-boost-strength 2`

I crank back some of the settings, including psy-rd as well as including `--filtering-noise-detection 3`, which enables restoration filtering at all times
and tends to help improve image stability. CDEF is still disabled when noise levels are high, since its internal metric (MSE) used to determine strength is still somewhat aggressive.

- `Balance of Appeal and Fidelity`

`--preset X --complex-hvs 1 --crf XX --kf-tf-strength 1 --tf-strength 1 --noise-norm-strength 1 --enable-qm 1 --qm-min 4 --qm-max 15 --chroma-qm-min 10 --chroma-qm-max 15 --keyint 240 --tune 0 --sharpness 1 --filtering-noise-detection 2 --aq-mode 2 --qp-scale-compress-strength 1 --scm 0 --psy-rd 1.5 --variance-boost-strength 2`

Psy-rd influence is lowered further, and we disable the CDEF/restoration noise detection algorithm completely; this has the effect of enabling CDEF/restoration
filtering at all times. This helps tip the balance much further into appeal to preserve those clean lines that most people prefer.

- `High appeal (low bitrates, line preservation, sacrificing some high frequency detail)`

`--preset X --complex-hvs 1 --crf XX --kf-tf-strength 1 --tf-strength 3 --noise-norm-strength 1 --enable-qm 1 --qm-min 4 --qm-max 15 --chroma-qm-min 8 --chroma-qm-max 15 --keyint 240 --tune 1 --sharpness X --aq-mode 2 --qp-scale-compress-strength 1 --psy-rd 1.0 --sharp-tx 0 --variance-boost-strength 2`

For maximum space savings, this settings string is geared towards more appealing output. We lower psy-rd influence further and disable sharp-tx to make the 
output less crisp, but keeps artifacts to a minimum. `--psy-rd 1.0 --complex-hvs 1` is still being used to provide higher fidelity output, with the rest of the settings compensating their effects to result in more appealing output.

- `BlueSwordM edition (a basis of what I tend to use as a tweakable base)`

`--preset 2 --complex-hvs 1 --crf XX --lp 1 --enable-cdef 0 --noise-norm-strength 3 --enable-qm 1 --qm-min 8 --qm-max 15 --chroma-qm-min 10 --chroma-qm-max 15 --keyint 240 --tune 0 --sharpness 1 --aq-mode 2 --qp-scale-compress-strength 1 --scm 0 --kf-tf-strength 1 --tf-strength 1 --psy-rd 2.0 --variance-boost-strength 2`

While the settings quoted above vary wildly from source to source (I use different settings for movies, episodic releases, and fast gameplay encodes), this is what I use as a basis to encode most of the content that I own. Do know that what I posted isn't very conservative and is biaised somewhat towards high fidelity.
Variance-boost-strength is the only thing I tend to play with a lot, since some content greatly benefits from higher strength; decreasing octile from the default doesn't seem to help much in dark areas because of psy-rd's high influence combined with `--complex-hvs 1`. However, decreasing it to `--variance-octile 5` can help
in more varied encoding scenarios, so try it out if you wish to do so.

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

0 follows the default tune behavior, 1 forcefully enables noise detection based CDEF/restoration filtering application, 2 disables noise detection based
CDEF/restoration filtering application, resulting in CDEF/restoration filtering always being on.

3 enables only disables the noise detection algorithm for restoration filtering, enabling restoration filtering at all times.
4 enables only disables the noise detection algorithm for CDEF, enabling CDEF at all times.

Default is **0**.


### Modified Defaults

SVT-AV1-PSYEX has enhanced defaults versus mainline SVT-AV1 in order to provide better visual fidelity out of the box. They include:

- Default 10-bit color depth when given a 10-bit input.
- Disable film grain denoising by default, as it often harms visual fidelity. (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/commit/8b39b41df9e07bbcdbd19ea618762c5db3353c03)**)
- Default to Tune 2 (SSIM) instead of Tune 1 (PSNR), as it reliably outperforms Tune 1 perceptually & throughout trusted metrics.
**This might change in the very near future**.
- Enable quantization matrices by default.
- Set minimum QM level to 4 by default for more consistent performance that min QM level 0 doesn't offer. It has been increased from 2, as 4 provides the most balanced gains overall.
- Set minimum chroma QM level to 8 by default to prevent the encoder from picking suboptimal chroma QMs.
- `--enable-variance-boost` enabled by default.
- `--keyint -2` (the default) uses a ~10s GOP size instead of ~5s.
- `--sharpness 1` by default to prioritize encoder sharpness.
- Sharp transform optimizations (`--sharp-tx 1`) are enabled by default to supercharge svt-av1-psy psy-rd optimizations. It is recommended to disable it if you don't use `--psy-rd`, which is set to **1.0** by default.
- `--tf-strength 1` by default for much lower alt-ref temporal filtering to decrease blur for cleaner encoding.
- `--kf-tf-strength 1` controls are available to the user and are set to 1 by default to remove KF artifacts.


*We are not in any way affiliated with the Alliance for Open Media or any upstream SVT-AV1 project contributors who have not also contributed here.*

### Other Changes

- `--color-help` (**[Merged to Mainline](https://gitlab.com/AOMediaCodec/SVT-AV1/-/merge_requests/2351)**)

Prints the information found in Appendix A.2 of the user guide in order to help users more easily understand the Color Description Options in SvtAv1EncApp.

- `Micro-Releases`

We are always continuously improving SVT-AV1-PSY, and we always recommend using the `master` branch to experience exciting new features as soon as they can be considered usable. To make our feature additions more clear, micro-release tags indicate when significant new feature additions have been made. Micro-release tags are letters starting with `A`, so new releases will be tagged as `v#.#.#-A`, `v#.#.#-B`, etc.

- `Enhanced Content Detection`

Tune 4 features a smarter content detection algorithm to optimize the encoder for either screen or photographic content based on the image. This helps Tune 4 achieve better visual fidelity on still images.

# For a diferent take on encoding, I recommend trying out SVT-AV1-HDR: https://github.com/juliobbv-p/svt-av1-hdr/

# Building

For Linux, macOS, & Windows build instructions, see the [PSY Development](Docs/PSY-Development.md) page.

# Getting Involved

For more information on SVT-AV1-PSY and this project's mission, see the [PSY Development](Docs/PSY-Development.md) page.

### Use SVT-AV1-PSY

One way to get involved is to use SVT-AV1-PSY in your own AV1 encoding projects, increasing the impact our work has on others! You and your users will also be able to provide feedback on the encoder's overall performance and report any issues you encounter. Your name will also be added to this page.

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
