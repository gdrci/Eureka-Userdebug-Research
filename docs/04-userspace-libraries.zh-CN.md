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

下表每行给出复现或推翻的结论，以及声明的完整文字。

| ID | 标注 | 复现 / 推翻 | 声明 |
|---|---|---|---|
| F02 | observed | confirmed / needs_change | 每个目标的 DT_SONAME 均与其文件名相同。lib64 副本为 ELFCLASS64、EM_AARCH64、ET_DYN；lib 副本为 ELFCLASS32、EM_ARM、ET_DYN。八个目标在 lib64 与 lib 中均不含 DT_RPATH 或 DT_RUNPATH 条目。 |
| F05 | verified | confirmed / needs_change | libGLESv2_adreno.so 的 DT_NEEDED（10 项，lib64 与 lib 相同）：libadreno_utils.so、libgsl.so、libllvm-glnext.so、libcutils.so、libdl.so、libz.so、liblog.so、libc++.so、libc.so 和 libm.so。 |
| F08 | verified | confirmed / needs_change | libq3dtools_adreno.so 的 DT_NEEDED（7 项，lib64 与 lib 相同）：libcutils.so、liblog.so、libgsl.so、libc++.so、libc.so、libm.so、libdl.so。 |
| F09 | verified | confirmed / needs_change | libadreno_app_profiles.so 的 DT_NEEDED（13 项，lib64 与 lib 相同）：libcutils.so、libhardware.so、libutils.so、libhidlbase.so、libhidlmemory.so、android.hidl.memory@1.0.so、vendor.qti.qspmhal@1.0.so、libdl.so、libz.so、liblog.so、libc++.so、libc.so 和 libm.so。 |
| F13 | verified | confirmed / needs_change | libGLESv2_adreno.so 导出 500 个名称匹配 ^gl[A-Z] 的已定义动态符号（GLES 入口点）。它不导出 glX 前缀的名称，也不导出 egl[A-Z] 名称。其唯一的 cl 前缀导出为 cl_gl_request_interface，它不具有 cl[A-Z] 形式。 |
| F15 | verified | needs_change / needs_change | 在 libgsl.so、libadreno_utils.so、libq3dtools_adreno.so、libadreno_app_profiles.so 与 libar-gsl.so 中，以 egl*、gl*、glX*、cl* 与 vk* 开头的导出名称为零（lib64 与 lib 副本相同，各前缀均在符号名开头匹配；libq3dtools_adreno.so 导出的 qCLShimAPI_clCreateImage 等名称中，cl 不在开头）。 |
| F16 | observed | confirmed / needs_change | libOpenCL_adreno.so 的版本需求（verneed），该库在两个副本中均存在。在 lib64 副本中，libvndksupport.so 需要 LIBVNDKSUPPORT，libc.so 需要 LIBC，libdl.so 需要 LIBC。在 lib 副本中，libvndksupport.so 需要 LIBVNDKSUPPORT，libc.so 需要 LIBC 与 LIBC_N，libdl.so 需要 LIBC。 |
| F17 | observed | needs_change / needs_change | 其他目标的版本需求（同一解析方法）：libgsl.so：liblog.so 需要 LIBLOG；libsync.so 需要 LIBSYNC；libc.so 需要 LIBC、LIBC_N；libm.so 需要 LIBC；libdl.so 需要 LIBC。libgsl.so 的 lib64 与 lib 副本具有相同的版本需求。 |
| F18 | verified | confirmed / needs_change | 所检查的四个 lib64 副本（libgsl.so、libEGL_adreno.so、libGLESv2_adreno.so、libOpenCL_adreno.so）均不含 .gnu.version_d 节，因此它们本身不定义符号版本。每个副本中的每个已定义动态符号的版本索引均为 1（全局），因此它们都不携带版本名。 |
| F19 | verified | confirmed / needs_change | 动态加载器与属性导入（未定义的动态符号，附版本与提供库）。libgsl.so：__android_log_print@LIBLOG、dlopen@LIBC、dlclose@LIBC、dlerror@LIBC 和 dlsym@LIBC（所需版本 LIBC 来自 libdl.so），以及 property_get（无版本标签，因此无提供库）。lib 副本导入这些已命名符号时使用相同的版本名，但两个副本在其他未定义符号上有所不同，例如 ARM 副本导入的 __aeabi_mem* 辅助函数。 |
| F21 | verified | needs_change / needs_change | 供应商和调试属性名称以字面字符串形式出现（property_get 由 libgsl、libEGL、libGLESv2、libOpenCL 和 libq3dtools 导入；在所涵盖的 Adreno 库中，只有 libGLESv2 导入 property_set，lib64 与 lib 副本相同）。所涵盖的集合为 lib/ 与 lib64/ 中直接存在的 Adreno 库及其 egl/ 子目录，不含 hw/ 子目录。在该集合中，property_get 在 lib64 与 lib 副本中由 libgsl.so、libEGL_adreno.so、libGLESv2_adreno.so、libOpenCL_adreno.so、libq3dtools_adreno.so、libq3dtools_esx.so 和 eglSubDriverAndroid.so 导入，其中 libq3dtools_esx.so 与 eglSubDriverAndroid.so 仅位于 egl/ 中；libGLESv1_CM_adreno.so、libOpenCL.so、libadreno_utils.so、libadreno_app_profiles.so 和 libar-gsl.so 不导入它。libq3dtools_esx.so 文件在两个副本中还导入 __system_property_set（一个不同的符号名）。lib/hw/vulkan.adreno.so 与 lib64/hw/vulkan.adreno.so 两个驱动文件导入 property_get、property_list 与 property_set。 |
| F22 | verified | needs_change / needs_change | libgsl.so 中以字面字符串出现的文件路径，lib64 与 lib 副本相同（18 个完整字符串）：/dev/kgsl-3d0、/dev/kgsl-2d0、/dev/kgsl-2d1、/dev/dri、/dev/dri/%s、/dev/dri/renderD129 和 /dev/ion；/data/misc/gpu/、/data/vendor/gpu/、/data/vendor/gpu/glslfs.txt、/data/vendor/gpu/glslvs.txt、/data/vendor/gpu/perfCounterDump.txt、/data/vendor/gpu/resolve_log.txt 和 /data/vendor/gpu/scdevdump.txt；/mnt/sdcard/profiler_records；以及 /proc/%d/maps、/proc/%d/smaps 和 /proc/meminfo。 |
| F23 | verified | confirmed / needs_change | libGLESv2_adreno.so（lib64）中以字面字符串出现的子驱动及相关库名称：eglSubDriverAndroid.so、eglSubDriverDummy.so、eglSubDriverLE.so、libeglSubDriverFbdev.so、libeglSubDriverFreon.so、libeglSubDriverQnx.so、libeglSubDriverWayland.so、libeglSubDriverX11.so 和 eglSubDriverWin32.dll。 |
| F24 | verified | confirmed / needs_change | GLES 版本字符串：OpenGL ES 1.1、OpenGL ES 2.0、OpenGL ES 3.0（也存在带尾随空格的形式）、OpenGL ES 3.1、OpenGL ES 3.2，以及 GLSL ES 版本 OpenGL ES GLSL ES 3.00、OpenGL ES GLSL ES 3.10 和 OpenGL ES GLSL ES 3.20，它们均以字面字符串形式出现在 egl/libGLESv2_adreno.so 的 lib64 与 lib 两个副本中。 |
| F25 | verified | confirmed / needs_change | 两个 libEGL_adreno.so 副本中均不存在 EGL 版本字面量：没有字符串匹配 EGL 后接版本号、EGL_VERSION 或 'EGL N.N'。以 EGL 开头的导出名称为 78 个匹配 ^egl[A-Z] 的已定义动态符号，两个副本导出相同的集合。 |
| F26 | verified | needs_change / needs_change | 八个目标中与 Vulkan 相关的字面量：裸标记 Vulkan 在 libadreno_app_profiles.so 的两个副本中均出现。libgsl.so 的两个副本均包含字符串 'Error %d attempting to get the VK_DEVICE_ID property' 与 'KGSL_PROP_VK_DEVICE_ID not available at run time'。八个目标中的其他目标均不含 Vulkan 标记或 VK_ 名称。 |
| F27 | observed | needs_change / needs_change | 该应用配置库定义了两个导出函数 ApplyApplicationProfile 与 AppProfDestroyStruct，以及四个数据对象：它们是 std::__1 命名空间中的 C++ 标准库模板实例化，均针对 char：basic_ostringstream 的虚表与 VTT、basic_stringbuf 的虚表，以及 basic_ostringstream 内 basic_ostream 的构造虚表。 |
| F28 | verified | confirmed / needs_change | libar-gsl.so（lib64）定义 189 个动态符号，且不导入 dl*、property 或 android_* 符号。libadreno_utils.so（lib64）定义 109 个动态符号，导入 18 个未定义动态符号，其中 7 个带有来自 libc.so 的 LIBC 版本需求，且它们均不是 dl*、property 或 android_* 符号。 |
