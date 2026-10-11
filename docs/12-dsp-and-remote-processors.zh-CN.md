# 12 - DSP 与远程处理器

[English](12-dsp-and-remote-processors.md)

## 摘要

该组共有 243 行（237 个普通文件和 6 个符号链接）：odm_a 中 20 行，vendor_a 中 223 行。它包含远程处理器的固件和运行时集合：用于 ADSP 和 CDSP 的 Hexagon DSP6 根镜像（拆分的 .mdt 加 .bNN 段）、FastRPC 用户态守护进程（adsprpcd、cdsprpcd）以及 DSP 侧的 shell 和 skel 库；XRA 音频管线的 CAPI 模块，附带 JSON 配置和调优数据块；一个 EVA Xtensa 镜像和一个 SPSS ARM 镜像；QCA6490 WLAN 固件，附带 board-data ELF、消息目录和 ini 配置；带有 servreg JSON 配置的 AArch64 安全应用镜像；ODM 微控制器镜像（SyncBoss、SantanaManager，以及附带 unwind ELF 的 MCU tar 包）、PDFU 更新包和一个 EFM8 镜像。已挂载的 ueventd.rc、vendor_file_contexts、vendor_sepolicy.cil 以及 init .rc 文件确认了固件搜索路径、/vendor/bin/adsprpcd 和 /vendor/bin/cdsprpcd 的标签、三个 remoteproc 实例（spss、adsp、cdsp），以及 adsprpcd 和 cdsprpcd 的 init 服务。大多数固件镜像名称未被任何已挂载的 init 或 SELinux 文本文件按名称引用。

角色表为本组中的 243 个文件各提供一行（153 verified、79 observed、11 inferred、0 unverified）。该表位于 `data/subsystems/dsp-and-remote-processors/roles.tsv`。

## 已验证的论断

- **G1**（verified）：该组共有 243 个输入行：237 个普通文件和 6 个符号链接，其中 odm_a 中 20 行，vendor_a 中 223 行。
- **G2**（verified）：全部 237 个普通输入文件均存在于挂载根目录下，且与其记录的 sha256 值一致。
- **G3**（verified）：六个符号链接指向挂载集合之外（odm_a 中四个指向 /persist 校准路径，vendor_a 中两个为 WLAN 链接）。WLAN 配置的目标存在于 vendor_a/etc/wifi 内；/persist 的目标不在挂载根目录下。
- **G4**（verified）：九个固件主干名为拆分镜像：一个 ELF 程序头文件（.mdt）加上编号的段文件（.bNN）。对每个主干，每个存在的段文件大小都等于其程序头的 p_filesz，但 adsp.b31 除外（见 D3）。
- **G5**（observed）：本组中出现的镜像标识字符串：DSP 根镜像、EVA、视频固件和 QCA6490 WLAN 均带有 QC_IMAGE_VERSION_STRING 和 IMAGE_VARIANT_STRING。
- **G6**（observed）：已挂载的配置文件指明了本组镜像所用的固件搜索路径和三个 remoteproc 实例。
- **G7**（verified）：角色表对全部 243 行各覆盖一次，置信度计数为 153 verified、79 observed 和 11 inferred。
- **D1**（verified）：bin/adsprpcd 是面向 ADSP 的 AArch64 PIE FastRPC 用户侧守护进程，动态链接 liblog、libc++、libc、libm 和 libdl。
- **D1b**（observed）：adsprpcd 的日志字符串描述了其循环：启动 'adsp daemon'，通过监听器启动函数加载 libadsp_default_listener.so，出错时在 25 ms 后重启。
- **D1c**（observed）：adsprpcd 由一个 main 类 init 服务启动，以用户 system、组 media 运行；其可执行文件标签为 vendor_adsprpcd_exec，并定义了 init 转换和设备规则。
- **D2**（verified）：bin/cdsprpcd 与 adsprpcd 具有相同的 ELF 类型和 NEEDED 列表，在 1226 个字节位置上与之不同；其日志字符串描述了一个 CDSP 监听器循环，重启延迟为 100 ms。
- **D2b**（observed）：cdsprpcd init 服务默认禁用，在列出的 SKU 和设备上开机时启动，并可通过属性 vendor.fastrpc.disable.cdsprpcd.daemon=1 停止。其可执行文件标签为 vendor_cdsprpcd_exec。
- **D3**（verified）：adsp.mdt 是一个 ELF32 Hexagon DSP6 EXEC，入口为 0x87a00000，有 47 个程序头；存在 46 个段文件。段 31 的 p_filesz 为 0，但文件大小为 7528 字节；段 45 的 p_filesz 为 0，且没有对应文件。各段大小合计 13,393,180 字节。
- **D3b**（observed）：配置中的 ADSP 镜像标签：/vendor/firmware 标记为 firmware_file；ADSP remoteproc1 SSR sysfs 节点标记为 vendor_sysfs_ssr；adsp 镜像携带 DSP.XT.1.1-00646-HALLIDAY-1，变体为 halliday.adsp.prodQ。
- **D4**（verified）：cdsp.mdt 是一个 ELF32 Hexagon DSP6 EXEC，入口为 0x8b500000，有 16 个程序头；存在 15 个段文件（索引 14 没有文件，其 p_filesz 为 0），且所有存在的文件大小均匹配。
- **D4b**（observed）：CDSP 镜像标识为 DSP.XT.1.1-00646-HALLIDAY-1（变体 halliday.cdsp.prodQ），并与 servreg 域 cdsp/root_pd（QMI 实例 76）配对。
- **D5**（verified）：evass.mdt 是一个 ELF32 Tensilica Xtensa 镜像，入口为 0xf500000，有 20 个程序头；存在 20 个段文件，其中 8 个长度为零（b10 至 b17）。evass.mbn 是一个独立的单文件 Xtensa ELF，大小为 2,333,480 字节，且其开头不是 .mdt 的字节。
- **D5b**（observed）：EVA 镜像标识为 EVA.FIRMWARE.3.6-11（变体 PROD）。厂商属性前缀 vendor.runtime.eva. 标记为 vendor_adsprpc_prop，将 EVA 运行时属性与 FastRPC 属性标签关联起来。
- **D6**（verified）：spss1p 和 spss1t 是 ARM EABI5 拆分镜像（machine 0x28，入口 0x1000），各有三个程序头，每个都有三个对应的段文件。它们的单文件 .mbn 形式也是 ARM ELF 镜像。
- **D7**（verified）：hdcp1.mdt、hdcp2p2.mdt、hdcpsrm.mdt 和 loadalgota64.mdt 是 AArch64 ELF64 拆分镜像，各有九个程序头和对应的段文件。它们的字符串中包含根证书 'Eureka_FW_Root1' 以及主体 'Meta Platforms, Inc.'。widevine.mbn 链接了 libcmnlib.so。
- **D7b**（observed）：本组中的其他可信应用镜像带有识别字符串：eyetz64.mbn 含有 tz_app_init 和 'calling command test'，sp_license.mbn 会打印 'SP license app Init'。已挂载的 init 或 SELinux 文本文件中均未提及这些 .mbn 或 .mdt 镜像（grep 计数为 0），但 Widevine HAL 服务条目启动的是一个独立的 HAL 二进制文件，这一条除外。
- **D8**（verified）：amss.bin 是 QCA6490 WLAN 固件镜像：ARM ELF32 EXEC，入口 0x143a800，大小 5,523,568 字节。amss20.bin 是同一 WLAN.HSP.2.0 构建（eval_v2）的第二个版本，大小为 5,603,328 字节。m3.bin 是一个 WLAN ucode 镜像，其 ELF32 头的 e_machine 字段为 3（EM_386）。
- **D10**（verified）：Data.msc 和 Data20.msc 是文本格式的 WLAN 固件消息目录。每个文件都有一个 VERSION:536936572 头部，以及形如 id,args,[ file : line ] text 的带 id 编码记录：Data.msc 有 10,298 条记录，Data20.msc 有 10,579 条。
- **D10b**（inferred）：regdb.bin 是二进制形式的 WLAN 监管数据库；其记录布局未被解码。
- **D11**（verified）：xra_config_render 和 xra_config_capture 是 XRA 音频管线的 JSON 配置。它们指向 /vendor/lib/rfsa/adsp/xra_resources 下的资源文件，给出引脚映射和采样参数（48 kHz，采样类型），以及 /data/data_collection 下的调试输出。
- **D12**（observed）：xra_render_capi.so 和 xra_capture_capi.so 是 XRA 的 DSP6 CAPI 模块。其字符串包括 xra_capi_get_static_properties、13 个 capi_utils_props_* 辅助函数名，以及 facebook2xr5audio 命名空间中的 C++ 符号。动态符号表中未列出 xra_capi_get_static_properties。
- **D13**（observed）：DSP 侧的 FastRPC shell 库（fastrpc_shell_0、fastrpc_shell_3、fastrpc_shell_unsigned_3）是 DSP6 DYN 目标，没有 NEEDED 条目，定义了约 1,550 至 1,600 个全局函数。它们的导出包括 HAP 和 qurt API。synx 库导出 synx_* 同步对象 API。
- **D13b**（observed）：若干 DSP skel 库带有 QAIC 生成的 invoke 入口点（例如 loadalgo_skel_handle_invoke_qaic_version、sysmonquery_skel_handle_invoke_qaic_version），表明采用了 FastRPC IDL skel 模式。
- **D14**（verified）：五个 servreg JSON 文件（adspr、cdspr、adsps、adspua、battmgr）声明了一个 sr_domain 以及一到两个 sr_service 条目，QMI 实例 ID 分别为 74（adsp）和 76（cdsp）。adspua.jsn 另外添加了一个 avs 音频服务条目。
- **D15b**（observed）：syncboss-app.bin 镜像具有 dAeH 前缀的头部，包含一个 12 位十六进制构建标识符；MCU tar 包按设备保存 app 与 SPL-updater 镜像，其版本集合的日期为 2020 至 2024 年。
- **D15c**（observed）：unwind ELF 镜像（unwind-elk-app.bin、unwind-coulee-app.bin、syncboss-unwind.bin）只包含 .debug_frame 和符号段，没有代码段。
- **D15d**（observed）：ODM odm_dlkm 模块列表包含 syncboss 内核模块（syncboss_spi.ko、syncboss_miscfifo.ko、syncboss_powerstate.ko、syncboss_direct_channel.ko、syncboss_nsync.ko），厂商 init 设置了 syncboss 的 sysfs 权限，并等待固件更新节点。
- **D16**（verified）：PDFU 更新包以 46 字节的 ASCII 十六进制头开始，该头在偏移 0x2e 处以 CRLF 结束。负载以字 0x20003000 和一个复位向量开始，这是 Cortex-M 向量表的布局。
- **D16b**（observed）：PDFU 包通过 Meta PDFU 路径分发：ueventd 外部处理程序匹配 2833-*.pdfu（Meta VID 0x2833），PDFU HAL 标签为 hal_pdfu_default_exec，/odm/bin/pdfu_depot 标签为 pdfu_depot_exec。存储的文件名以产品名称（Lehua-、Moku-）开头，而不是 2833-。
- **D17**（observed）：EFM8 镜像 Eureka_Dev1.efm8 是原始二进制（非 ELF），包含设备标签和 LED 标签字符串。没有已挂载的 init 或 SELinux 文本提及它。
- **D19**（observed）：ODM fw_init.cfg 的 [blu] 段指向 /vendor/firmware/blu.bin 和 blu.toml，但本组的 vendor_a/firmware 中没有这些文件。

## 措辞已修正的论断

下方结论以数据所支持的措辞重新表述。

- **D6b**（observed）：变体后缀 p 或 t 也出现在签名容器 crypt1p.sig 和 crypt1t.sig 上。两者的 organizationalUnit 值均为 'Qualcomm Cryptographic Operations'。它们的 commonName 值不同：'SPU MBNv7 Image Signing Root CA 6 SubCA 1'（crypt1p.sig）和 'SPU MBNv7 Image Signing Test Root CA 6 SubCA 1'（crypt1t.sig）。组织单位之后出现的数字串 1200 和 1705 是 DER 头字节，并不属于主体字符串。
- **D8b**（observed）：QCA6490 WLAN 配置 WCNSS_qcom_cfg.ini 共有 126 行：99 行 key=value、14 行注释、12 行空行，以及结尾的 END 行。amss.bin 内嵌了其子镜像的名称，WLAN 配置和标签引用也出现在已挂载的 SELinux file contexts 中。
- **D9**（verified）：十四个 qca6490 eureka_*.elf 和 bdwlan*.elf 文件是 board-data ELF 封装，每个 60,028 字节，分为四个内容组。每组内嵌其自身的 58,180 字节 .data 负载，因此四组共包含四种不同的负载。
- **D9b**（observed）：board-data ELF 通过名为 _binary_tempDir_d1Xday120bdwlan_bin_start、_end 和 _size（bdwlan 组）或 _binary_tempDir_d1Xday120bdwlang_bin_start、_end 和 _size（bdwlang 组）的链接器符号嵌入其负载，其中 X 取决于分组：bdwlan.elf 和 bdwlang.elf 为 d10，eureka_evt2.1_bdwlan.elf 和 eureka_pvt1.1_bdwlang.elf 为 d11。
- **D11b**（observed）：XRA 调优资源数据块带有文本配置头，分别标明 Stinson_EVT2v1.4（resources_render.dat）、stinsonWola（resources_capture.dat）和 stinsonExperimentalWola（resources_capture_ex.dat）。
- **D15**（verified）：ODM 的 syncboss.bin 和 santana_manager.bin 镜像是带向量表的 ARM Cortex-M 固件（SyncBoss SP 0x2001e000，复位 0x000006e8；SantanaManager SP 0x20008000，复位 0x080005e0）。fw_init.cfg 给出了它们的路径，catty.cfg 给出了它们的串口设备（/dev/ttyHS14 和 /dev/ttyHS12）。fw_init 二进制引用了 /odm/etc/fw_init.cfg；读取 catty.cfg 的组件尚未确定，因此启动关系为推断。
- **D18**（observed）：/persist/calibration/prox/min_ipd/ 下的 PROX_PS 校准链接目标仅被通用的 /persist/calibration 标签覆盖（vendor_file_contexts 第 252 行，persist_cal_file）。/persist/PROX[A-Z_]+ 标签（第 224 行）与这些目标不匹配。没有已挂载的 init 或 ODM 配置提及 PROX_PS 或 min_ipd（grep 计数为 0）。

## 缺口

- 若干 DSP 库的导出表中没有已定义符号：libdspCV_skel.so、libhexagon_skel.so、libdfs_skel.so、libboltEva_skel.so、libQnnBoltnnOpPackageV69.so、rfsa 副本的 libevadsp_3_0.so 以及 XRA CAPI 模块均报告未定义任何符号。它们的角色仅基于名称、soname 和字符串。
- Hexagon、Xtensa、ARM 和 AArch64 镜像的主体未经反汇编。EVA、SPSS、VPU、证明（attestation）、传感器锁、OVR 以及可信应用镜像的功能仅根据名称和字符串推断。
- regdb.bin 的布局未被解码。Eureka_Dev1.efm8 的格式也未被解码。
- adsp.b31 段（程序头中 p_filesz 为 0，文件为 7528 字节）以及各零长度段的主体均未被解释。
- file(1) 将 cdsp.b09 报告为 Matlab v4 mat 文件。由于其大小与程序头 9 相匹配，它被视为一个段，但其内容未被解码。
- 除了表明 .mbn 不以 .mdt 字节开头之外，evass.mbn 相对于 .mdt 和 .bNN 文件的布局并未确定。
- /persist 下的 PROX_PS 符号链接目标以及 WLAN MAC 文件 /persist/wlan_mac.bin 不在挂载集合之内，因此其内容和大小未知。
- 无法根据文本 file_contexts 核对 ODM 固件标签（odm_a/etc/selinux 仅包含 precompiled_sepolicy）。/odm/firmware 和 /odm/etc/firmware 的文件标签未验证。
- 没有已挂载的 init 或 SELinux 文本文件提及 ADSP、CDSP、SPSS、EVA、VPU 或 QCA6490 的镜像文件。它们的加载路径是根据 ueventd firmware_directories、/vendor/firmware 文件标签和 remoteproc sysfs 标签推断的。加载代码位于本组之外的二进制文件中。
- ueventd 的通配模式 2833-*.pdfu 与存储的 PDFU 文件名不匹配，后者以 Lehua- 或 Moku- 开头。能够产生匹配名称的暂存步骤不在已挂载的分区中。
- 消费这些固件和 DSP 文件的 odm_dlkm 与 vendor lib64 主机侧库（例如 libadsprpc.so、libadsp_default_listener.so、libcdsp_default_listener.so）不在本组行之内，也未做分析。
- MCU tar 包中的镜像（elk-app.bin、deerfly-app.bin、jedi.bin、coulee-app.bin、jade-app.bin 以及 SPL updater 镜像）仅通过 file(1) 的输出识别；其内部格式和版本字段未被解码。
- PDFU 负载中前 8 字节之后的部分，以及 dAeH 头中除 magic、长度和构建标识符之外的字段，均未被解码。
- 若干镜像中发现的 OEM_IMAGE_VERSION_STRING 值包含构建主机标识符；未复现这些值，也未分析其含义。
