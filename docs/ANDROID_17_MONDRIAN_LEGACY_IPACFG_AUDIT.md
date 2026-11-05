# Android 17 / PixelExperience-Reborn — legacy Qualcomm data IPA configuration audit

**Date:** 2026-11-05  
**135-repository matrix row:** P2 `hardware/qcom/data/ipacfg-mgr`  
**Decision:** legacy msm8998 source is not the Taro IPACM implementation; preserve without enabling.

## Exact source evidence

- Historical PE branch `seventeen` is commit `482983bd2ae8315e0b8c19bb532f7a84087c1689`, containing `msm8998/hal`, `msm8998/ipacm`, and `msm8998/ipanat`.
- The root `Android.mk` includes `$(LOCAL_PATH)/msm8998` **only if `TARGET_BOARD_PLATFORM` is `msm8998`**. The historical `msm8998/ipacm/src/Android.mk` calls `generated_kernel_headers`, builds `ipacm` and links `liboffloadhal`, `libipanat`, `libnfnetlink`, `libnetfilter_conntrack`, and legacy `android.hardware.tetheroffload.config@1.0` / `control@1.0` HALs. It is neither a modern AOSP17-compatible source graph nor a Taro package under its present root Make gate.
- The independent AOSP 17 reference manifest does **not** contain `hardware/qcom/data/ipacfg-mgr`. Android platform connectivity / netd / tethering framework code cannot stand in for Qualcomm IPA userspace and proprietary driver interfaces.
- The real `LF52406/mondrian_android_device_xiaomi_sm8450-common` `lineage-24.0` sets `TARGET_BOARD_PLATFORM := taro`, refers to kernel Qualcomm dataipa and IPA device drivers, and chooses `ipacm`, `IPACM_cfg.xml`, and `IPACM_Filter_cfg.xml` in `common.mk` (IPACM section). **IPACM is a real device requirement**, unlike this particular old msm8998 checkout.

## Modernization target requires separate review, not a blind replacement

- The **other** historical PE `hardware/qcom-caf/sm8450/data-ipa-cfg-mgr` (separate P2 matrix row 141, source `55912090d35be6469ec3d640371df31ca26e3a1b`) does include `ipacm/Android.bp`, `ipacm_vendor_product.mk` and corresponding resources. This is a closer *candidate* for Taro ownership, **not** a verified Android17 solution.
- Its `ipacm/Android.bp` builds `ipacm` with `device_kernel_headers` and `qti_kernel_headers` header libraries and links Qualcomm `liboffloadhal`/`libipanat` plus HIDL tether offload 1.0/1.1 interfaces. Its `ipacm_vendor_product.mk` contains older platform lists (e.g. `msm8998`/`sdm845`/`kona`) and must be checked for `taro` selection rather than presumed compatible.
- The additional `qti_kernel_headers` Soong dependency is also referenced by the *separate* reviewed Qualcomm Thermal AIDL v3 source. The precise **single header provider** (plus API exports and namespace visibility) must be resolved for the **new PE-Reborn device graph**, not by introducing fake empty modules.
- The official `LineageOS/android_hardware_qcom_data_ipacfg-mgr` branch inventory currently reaches `lineage-23.0`, with an `msm8998` root, rather than an automatically valid QSSI17 sm8450 baseline; it is not a substitute for verified Taro vendor sources.

## Required Android 17 bring-up gates

1. Leave this historical msm8998 checkout **out of active AOSP17 PE manifest profiles**; do not activate it alongside the sm8450 path and duplicate `ipacm`, `liboffloadhal` or `libipanat` modules.
2. Establish precise Taro-compatible IPA driver UAPI/ioctl and userspace module set; verify SELinux labelling, vendor process and firmware/config paths, and contemporary system `netd`, tethering, IPA offload and BPF behavior.
3. Reconcile AIDL/HIDL and native library ABI consumers, including tether offload interfaces, modern Soong namespaces, `qti_kernel_headers`, `device_kernel_headers`, `libnetfilter_conntrack` and `libnfnetlink`.
4. Port only proven fixes from the older tree to the **chosen** modern IPA owner; do not remove IPA support simply to make Soong compile.
5. Run Android17 `ipacm`/offload library compilation, VINTF/SELinux enforcing, networking tests for Wi-Fi/LTE/hotspot, IPv4/v6 NAT, tether offload, route changes and suspend/resume on mondrian. Confirm service ownership and no competing daemon starts.

**Status:** historical-source gate and device consumers verified; current Taro IPACM candidate is unvalidated. No full Soong build, linker ABI test or physical network test performed. The original `seventeen` branch and the currently functional MistOS trees remain untouched.

**Next row:** `hardware/qcom/display`.
