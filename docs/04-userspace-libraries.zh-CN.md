# 04 - 用户空间 GPU 库

[English](04-userspace-libraries.md)

Adreno 用户空间栈是一组共享库：GPU 核心库、EGL 与 OpenGL ES 驱动、OpenCL 驱动，以及一个工具库和一个配置库。

## 已验证的结论

- **F01**（verified）：三个目标是指向 lib64 与 lib 中 egl/ 子目录的相对符号链接：libEGL_adreno.so -> egl/libEGL_adreno.so，libGLESv2_adreno.so -> egl/libGLESv2_adreno.so，libq3dtools_adreno.so -> egl/libq3dtools_adreno.so。其他五个目标在两个目录中都是普通文件（lib64 中大小：libgsl.so 2218160，libOpenCL_adreno.so 163408，libadreno_utils.so 97800，libadreno_app_profiles.so 976128，libar-gsl.so 203176 字节）。
- **F03**（verified）：libgsl.so 的 DT_NEEDED（9 项，lib64 与 lib 相同）：libcutils.so、liblog.so、libz.so、libsync.so、libdmabufheap.so、libc++.so、libc.so、libm.so、libdl.so。
- **F04**（verified）：libEGL_adreno.so 的 DT_NEEDED（9 项，lib64 与 lib 相同）：libadreno_utils.so、libgsl.so、libcutils.so、libdl.so、libz.so、liblog.so、libc++.so、libc.so、libm.so。
- **F06**（verified）：libOpenCL_adreno.so 的 DT_NEEDED（7 项，lib64 与 lib 相同）：libcutils.so、libvndksupport.so、liblog.so、libc++.so、libc.so、libm.so、libdl.so。
- **F07**（verified）：libadreno_utils.so 的 DT_NEEDED（7 项，lib64 与 lib 相同）：libgsl.so、liblog.so、libnativewindow.so、libc++.so、libc.so、libm.so、libdl.so。
- **F10**（verified）：libar-gsl.so 的 DT_NEEDED（8 项，lib64 与 lib 相同）：liblx-osal.so、libar-acdb.so、liblx-ar_util.so、libar-gpr.so、libc++.so、libc.so、libm.so、libdl.so。
- **F11**（verified）：定义的动态符号总数（lib64 / lib）：libgsl.so 580 / 580；libEGL_adreno.so 151 / 152；libGLESv2_adreno.so 538 / 539；libOpenCL_adreno.so 444 / 444；libadreno_utils.so 109 / 109；libq3dtools_adreno.so 556 / 556；libadreno_app_profiles.so 6 / 6；libar-gsl.so 189 / 189。
- **F12**（verified）：libEGL_adreno.so 导出 78 个名称匹配 ^egl[A-Z] 的已定义动态符号（lib64 与 lib 均为 78）。该库不导出 gl*、glX*、cl* 或 vk* 名称。
- **F14**（verified）：libOpenCL_adreno.so 导出 7 个名称匹配 ^cl[A-Z] 的已定义动态符号（lib64 与 lib 相同）。lib64 中其余 437 个已定义符号没有 cl 前缀。

## 开放项

以下结论已复现，但措辞或标注有误，或被推翻。在措辞修正之前，它们不在上文陈述。表格中的结论文本保留英文原文。

| ID | 标注 | 复现 / 推翻 | 结论（原文） |
|---|---|---|---|
| F02 | observed | confirmed / needs_change | Every target's DT_SONAME equals its file name. lib64 copies are ELFCLASS64, EM_AARCH64, ET_DYN; lib copies are ELFCLASS32, EM_ARM, ET_DYN. No DT_RPATH |
| F05 | verified | confirmed / needs_change | libGLESv2_adreno.so DT_NEEDED (10, identical in lib64 and lib): libadreno_utils.so, libgsl.so, libllvm-glnext.so, libcutils.so, libdl.so, libz.so, lib |
| F08 | verified | confirmed / needs_change | libq3dtools_adreno.so DT_NEEDED (7, identical in lib64 and lib): libcutils.so, liblog.so, libgsl.so, libc++.so, libc.so, libm.so, libdl.so |
| F09 | verified | confirmed / needs_change | libadreno_app_profiles.so DT_NEEDED (13, identical in lib64 and lib): libcutils.so, libhardware.so, libutils.so, libhidlbase.so, libhidlmemory.so, and |
| F13 | verified | confirmed / needs_change | libGLESv2_adreno.so exports 500 defined dynamic symbols matching ^gl[A-Z] (GLES entry points; no glX-prefixed names, no egl or cl names). Count is 500 |
| F15 | verified | needs_change / needs_change | Zero egl*, gl*, glX*, cl* and vk* exported names in libgsl.so, libadreno_utils.so, libq3dtools_adreno.so, libadreno_app_profiles.so and libar-gsl.so ( |
| F16 | observed | confirmed / needs_change | Version requirements (verneed, from the version section) for two lib64 copies: libvndksupport.so requires LIBVNDKSUPPORT; l |
| F17 | observed | needs_change / needs_change | Version requirements for the other targets (same parse): libgsl.so: liblog.so LIBLOG; libsync.so LIBSYNC; libc.so LIBC, LIBC_N; libm.so LIBC; libdl.so |
| F18 | verified | confirmed / needs_change | The four lib64 copies checked (libgsl.so, libEGL_adreno.so, libGLESv2_adreno.so, libOpenCL_adreno.so) contain no .gnu.version_d section, so they defin |
| F19 | verified | confirmed / needs_change | Dynamic loader and property imports (undefined dynamic symbols, with version and providing library). libgsl.so: __android_log_print@LIBLOG, dlopen@LIB |
| F21 | verified | needs_change / needs_change | Vendor and debug property names read as literal strings (property_get is imported by libgsl, libEGL, libGLESv2, libOpenCL and libq3dtools; property_se |
| F22 | verified | needs_change / needs_change | File paths present as literal strings, per target (lib64 and lib identical unless noted). libgsl.so: /dev/kgsl-3d0, /dev/kgsl-2d0, /dev/kgsl-2d1, /dev |
| F23 | verified | confirmed / needs_change | Literal sub-driver and related library names in libGLESv2_adreno.so (lib64): eglSubDriverAndroid.so, eglSubDriverDummy.so, eglSubDriverLE.so, libeglSu |
| F24 | verified | confirmed / needs_change | GLES version strings: OpenGL ES 1.1, OpenGL ES 2.0, OpenGL ES 3.0 (also with a trailing space), OpenGL ES 3.1, OpenGL ES 3.2, GLSL ES versions OpenGL  |
| F25 | verified | confirmed / needs_change | No EGL version literal exists in either libEGL_adreno.so copy: no string matches EGL followed by a version number, EGL_VERSION, or 'EGL N.N'. The EGL- |
| F26 | verified | needs_change / needs_change | Vulkan: the only Vulkan-related literal in any target is the bare token Vulkan in libadreno_app_profiles.so (both directories). No Vulkan API version  |
| F27 | observed | needs_change / needs_change | The app-profile library defines two exported functions, ApplyApplicationProfile and AppProfDestroyStruct, plus four data objects that are C++ standard |
| F28 | verified | confirmed / needs_change | libar-gsl.so defines 189 dynamic symbols and imports no dl*, property or android_* symbols (lib64). libadreno_utils.so defines 109 dynamic symbols, im |
