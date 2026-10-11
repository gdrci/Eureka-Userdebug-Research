# 07 - SELinux 策略与文件上下文

[English](07-selinux-policy.md)

## 摘要

selinux-policy 组是本镜像中携带的 Android SELinux 策略集：共 51 个文件，分布在 system_a（22 个）、system_ext_a（15 个）、vendor_a（11 个）和 odm_a（3 个）中。它包含：以 CIL 格式保存的平台策略源码（plat_sepolicy.cil，28,911 行，mls true，27 个 sid，4 个 policycap）；vendor 与 system_ext 的策略源码（vendor_sepolicy.cil、system_ext_sepolicy.cil）；vendor 策略所依赖的带版本号的平台公共策略（plat_pub_versioned.cil，2,077 个以 _32_0 结尾的名称）；odm 上一个预编译的二进制策略（文件头报告 SE Linux policy v30）；按版本划分的映射文件（26.0 至 32.0）及兼容桩文件；标签与查找表（file_contexts、seapp_contexts、property_contexts、service_contexts、hwservice_contexts、keystore2_key_contexts）；签名到 seinfo 的 XML 表；一份拒绝元数据表；以及一个内容为 32.0 的平台版本文件。每个分区有两条校验和记录，用于将预编译策略绑定到其输入；平台和 system_ext 的记录已重新计算，并与策略源码加 mapping/32.0.cil 的拼接结果相符。Init 与 libselinux 按名称引用这些路径。
该角色表为本组的 51 个文件各提供一行（46 个 verified，2 个 observed，3 个 inferred，0 个 unverified）。它位于 `data/subsystems/selinux-policy/roles.tsv`。

## 已验证的声明

- **C01** (verified): selinux-policy 组有 51 行：system_a 22、system_ext_a 15、vendor_a 11、odm_a 3。全部 51 项的 kind 均为 file：41 个 text/plain、6 个 inode/x-empty、3 个 text/xml、1 个 application/octet-stream。
- **C02** (verified): 所有 51 个映射文件都存在于挂载点下，且重新计算的每个文件的 SHA-256 与字节大小，均与清单行的第 7 列和第 5 列一致。
- **C03** (verified): 平台校验和记录等于 plat_sepolicy.cil 加 mapping/32.0.cil 拼接后的 SHA-256。仅 plat CIL 不匹配。
- **C04** (verified): system_ext 校验和记录等于 system_ext_sepolicy.cil 加上 system_ext 副本的 mapping/32.0.cil 拼接后的 SHA-256。system_a 副本的 mapping/32.0.cil 得到不同的摘要，且两个 32.0.cil 文件内容不同。
- **C05** (verified): 两个名为 precompiled_sepolicy.<name> 的 odm 校验和文件，分别与平台和 system_ext 记录逐字节相同。
- **C06** (verified): odm 的 precompiled_sepolicy 是一个 1,006,142 字节的二进制 SELinux 策略。其前 64 字节给出：魔数 0xf97cff8c，带长度前缀的字符串 'SE Linux'，版本 30，配置字 0x80000001，8 个符号表和 7 条 ocontext 记录。字段名根据常见策略头部顺序推断。
- **C07** (observed): 二进制中嵌入了以上下文族命名的标识符（例如 file_contexts_file、seapp_contexts_file、service_contexts_file、hwservice_contexts_file、property_contexts_file、keystore2_key_contexts_file、vendor_service_contexts_file、vndservice_contexts_file、nonplat_service_contexts_file）。其用途为推断。
- **C09** (verified): plat_sepolicy.cil 中顶层形式的计数：9,891 个 allow，3,773 个 neverallow，1,483 个 roletype，1,482 个 type，1,212 个 typeattribute，1,068 个 typeattributeset，1,024 个 category，562 个 dontaudit，365 个 genfscon，261 个 typetransition，102 个 class，89 个 mlsconstrain，74 个 classcommon，27 个 sid，27 个 sidcontext，20 个 fsuse，5 个 common，4 个 policycap。
- **C10** (verified): mapping/32.0.cil（system_a）为 189,038 字节、3,714 行。其中 typeattribute、typeattributeset 和 expandtypeattribute 三种形式各 1,238 个，每个属性都带 _32_0 后缀。system_ext 副本为 49,528 字节，包含 299 组这样的三元组。
- **C11** (verified): 版本 26.0 至 32.0 的映射集：.cil 文件大小依次为 44,069 / 80,908 / 95,919 / 111,009 / 124,230 / 140,511 / 189,038 字节（行数 754 / 1476 / 1726 / 1977 / 2188 / 2470 / 3714）。26.0 至 30.0 的兼容文件分别为 987、987、987、846、902 字节，31.0 的兼容文件为 29 字节。31.0 的兼容文件内容为 ;; This file can\'t be empty.。在 system_ext 中，26.0 至 31.0 的每个兼容文件都是 0 字节。未读取 26.0、28.0 和 29.0 兼容文件的内容。
- **C12** (verified): plat_pub_versioned.cil（1,028,727 字节）是平台公共策略的带版本副本。它包含 2,077 个不同的、以 _32_0 后缀结尾的名称，4,695 条 allow 规则，1,675 条 neverallow 规则和 2,463 个 typeattribute 形式。其最初的声明包括 typeattribute dev_type、bdev_type 和 domain。
- **C13** (verified): vendor_sepolicy.cil 引用了 1,589 个不同的 _32_0 名称。其中 54 个未在 plat_pub_versioned.cil 中定义。这 54 个全部是生成形式 base_typeattr_N_32_0，并在 vendor_sepolicy.cil 自身中以 typeattribute 形式声明。没有其他未解析的引用。
- **C15** (verified): system_ext_sepolicy.cil 为 400,885 字节、4,724 行，包含 3,159 个 allow、357 个 type、230 个 neverallow、228 个 typeattributeset、222 个 typeattribute 和 93 个 typetransition 形式。
- **C18** (verified): plat_file_contexts 是一个两列表格，包含路径正则和 SELinux 上下文。627 条只有两个字段，17 条有第三个字段 '--'，表示仅限普通文件。模式使用 (vendor|system/vendor) 交替和 (/.*)? 后缀。
- **C19** (verified): vendor_file_contexts（492 条）和 system_ext_file_contexts（86 条）都是两列的路径正则到上下文表，没有文件类型字段。
- **C20** (verified): plat_seapp_contexts（34 行）将应用身份映射到域和数据类型。选择器键包括 user、seinfo、name、isPrivApp、isSystemServer、isEphemeralApp、minTargetSdkVersion 和 fromRunAs。输出键为 domain、type 和 levelFrom。第一条规则将 isSystemServer=true 映射到 system_server_startup。
- **C24** (observed): services 框架代码通过路径引用 mac_permissions 文件（plat、odm 和 nonplat 变体）。services.jar 压缩包本身是压缩的，因此这些路径字符串是从编译后的 oat 镜像中读取的。
- **C26** (verified): plat_service_contexts 将 binder 服务名称映射到标签：312 个两字段条目，涵盖 272 个不同标签，全部在策略中声明。最后一条是 '*' 默认映射，指向 default_android_service。
- **C27** (verified): hwservice 上下文将 'package::Interface' 名称映射到标签：plat 为 85 条，涉及 74 个不同标签；vendor 为 50 条（31 个标签）；system_ext 为 3 条。所有标签均可解析。
- **C29** (verified): plat_keystore2_key_contexts 将整数 Keystore2 命名空间映射到密钥标签：7 条。命名空间必须是 [0, 2^31) 范围内的整数。例如 0 对应 su_key，1 对应 shell_key，100 对应 vold_key。
- **C31** (verified): plat_sepolicy_vers.txt 包含平台策略版本 '32.0'（5 字节）。这与平台校验和链所使用的 mapping/32.0.cil 一致。
- **C33** (verified): vendor_service_contexts 映射三个服务（两个 drm 条目和一个 display config 条目）。标签 hal_drm_service 未在已挂载分区的任何 CIL 文件中声明，因此该标签在此集合中无法解析。
- **C34** (verified): 清单中唯一的预编译策略文件是三个 odm_a 文件。vendor_a 分区没有预编译条目，且挂载中的 product_a 没有 etc/selinux 目录。
- **C35** (observed): init 二进制文件中包含策略输入的路径字符串：平台、system_ext、vendor、odm 和 product 的策略 CIL 文件，平台属性上下文，odm 和 vendor 的预编译策略路径，映射目录，平台版本文件，以及校验和后缀。该二进制文件已去除符号，因此加载顺序尚未确定。
- **C36** (verified): 有七个上下文文件以 '#line 1' 来源指令开头，给出构建树中的源路径。例如 system/sepolicy/private/property_contexts、vendor/meta/sepolicy/sepolicy-common/private/property_contexts 和 device/meta/anorak-common/sepolicy/vendor/service_contexts。

## 已修正措辞的声明

下方结论以数据所支持的措辞重新表述。

- **C08** (verified): plat_sepolicy.cil 为 28,911 行、1,897,754 字节的 CIL。它以三条 role 声明和两行 cil_gen_require 开头，随后是 (handleunknown deny) 和 (mls true)、四个 policycap，以及 27 个 sid 及其 sidcontext 形式。用户 u 的范围从 s0（无类别）到 s0 加类别范围 c0 至 c1023。
- **C16** (verified): plat_file_contexts（644 行）、vendor_file_contexts（492）和 system_ext_file_contexts（86）中的标签均可解析为五个 CIL 文件之一中声明的类型。这五个文件共声明 2,275 个不同的类型名称。各文件的不同标签数为 423（plat）、248（vendor）和 80（system_ext），没有无法解析的标签。
- **C17** (verified): plat_file_contexts 有 11 条规则使用 sepolicy_file 为策略文件加标签（行 36、66、333、371、445、495、496、623、624、642、644）。平台校验和名称仅匹配第 88 行（system_file）。odm 校验和名称匹配第 23 行（vendor_file）和第 28 行（vendor_configs_file），且不匹配第 644 行。
- **C22** (observed): system_ext 的 seapp 规则根据 seinfo 值 oculus_apps、oculus_apps_v2 和 fbandroid_debug 进行选择，并映射到 asr_app、mrds_service_app 和 spatial_persistence 等域。这些值与 Oculus 和 Facebook 的归属关系是根据名称推断的。
- **C23** (verified): mac_permissions XML 文件是签名到 seinfo 的表。plat 有 3 个 signer 元素（seinfo 为 platform、media、network_stack），vendor 有 1 个（platform），system_ext 有 11 个（echovr、fbandroid_debug、oculus_apps、oculus_apps_v2、oculus_core、oculus_inhouse、oculus_loneechoavalanche、oculus_privapps、oculus_prod、oculus_workrooms、oculus_workrooms_dev）。每个签名都是以 3082 开头的十六进制，即 DER SEQUENCE 头。在 plat 中，两个以 308204a8 开头，一个以 308205dc 开头。
- **C25** (verified): plat_property_contexts 有 932 行数据：197 行为两字段（名称和标签），735 行带 match 关键字和类型（exact 726，prefix 9）。类型分布为 string 376、int 164、bool 177、enum 11、uint 6 和 double 1。108 个名称以点结尾（其中 99 个位于两字段行），824 个不以点结尾。将带点后缀的名称描述为旧式写法，属于推断。
- **C28** (observed): service manager 二进制文件包含上下文文件名：hwservicemanager 包含错误文本 'No match for interface %s in hwservice_contexts'，servicemanager 包含 ' in service_contexts.'。hwservicemanager 没有单独的 service_contexts 引用；其匹配项是 hwservice_contexts 的子串。这些文件会被读取，是根据字符串推断的。
- **C30** (verified): selinux_denial_metadata 是一个 35 行的注释表，包含四个以空白分隔的字段：源域、目标类型、对象类别，以及形如 b/ 加 8 或 9 位数字的缺陷引用（8 位数字的 27 行，9 位数字的 8 行）。logd 二进制文件引用该路径。
- **C32** (observed): 预编译二进制报告策略版本为 30，而平台版本文件读作 32.0，且映射目录包含 mapping/32.0.cil（该目录覆盖 26.0 至 32.0）。这一差异的原因无法仅根据这些文件确定。

## 已排除的声明

下方声明已排除；每条都给出了原因。

| ID | 声明 | 原因 |
|---|---|---|
| C14 | vendor_sepolicy.cil 为 388,964 字节、1,190 行 CIL 形式，其中 2,150 个 allow、436 个 type、200 个 genfscon、146 个 dontaudit、130 个 typetransition、86 个 neverallow an | wc -l 给出 3951 行（grep -c '' 同样为 3951；顶层形式为 3950 个）。所声称的 1,190 行不成立。字节数 388964，allow 2150，type 436，genfscon 200，dontaudit 146，typetransition 130，nev |
| C21 | vendor 的 seapp 表有一条规则，system_ext_seapp_contexts（39 行）包含其余规则，共 40 条规则。每条规则都带有 user、seinfo、name、domain、type | 共 40 条规则（vendor 1 条，system_ext 39 条）。user、seinfo、name、domain 和 type 在全部 40 条规则中都存在。levelFrom 出现在 36 条中。没有它的四条规则是 system_ext 中针对 com.meta.internal 的规则 |

## 缺口

- 预编译二进制文件仅解析到其 64 字节头部。其符号表、类别与权限列表、avtab 以及 ocontext 记录均未解码，因此该二进制文件的规则内容尚不清楚。
- 未编译或验证任何策略。secilc（system_a/system/bin/secilc，aarch64 ELF）未被执行，因此构建标志、目标策略版本和编译输出均未复现。
- Init 与 libselinux 引用了未挂载在分区中的路径：/odm/etc/selinux/odm_sepolicy.cil 及 odm_*_contexts、/product/etc/selinux/product_sepolicy.cil 及 product_*_contexts、/vendor/etc/selinux/nonplat_sepolicy.cil 及 nonplat_*_contexts、/vendor/etc/selinux/precompiled_sepolicy、/debug_ramdisk/userdebug_plat_sepolicy.cil 以及 /mfgmode_plat_sepolicy.cil。未检查其内容和是否存在。
- 标签 hal_drm_service（vendor_service_contexts 中的两个条目）未在任何已挂载的 CIL 文件中声明。声明该标签的分区未包含在挂载中。
- file_contexts 中第 642 和 644 行的正则对校验和文件的命名与磁盘上的文件不同。磁盘上的校验和文件由第 88 和 28 行更宽泛的规则加上标签。命名背后的意图尚未确定。
- 未确定 file_contexts 中正则的优先级（当多条匹配条目同时存在时，使用者采用哪一条）。仅列出了匹配条目的集合。
- mac_permissions 签名属性中的证书未解码（主体、颁发者、有效期）。仅确认了十六进制前缀。
- 预编译二进制中嵌入的标识符（例如 vendor_service_contexts_file）仅从字符串中读取。它们在二进制中的作用为推断。
- init 二进制文件已去除符号。路径引用来自字符串，未进行反汇编，因此未确定 init 加载各文件的顺序和条件。
- 未读取（system_a 中）26.0、28.0 和 29.0 兼容桩文件的内容。它们的作用是根据文件名和大小推断的。
- 二进制（v30）与平台版本（32.0）之间版本不一致的来源未确定。
- selinux_denial_metadata 中的缺陷引用（b/NNNNNNNN）未与任何外部来源核对。
- 声明类型数（2,275）来自对五个 CIL 文件中 '(type NAME' 形式的 grep 结果。它们不是策略编译的输出，也不包括 typealias 或生成的名称。
- 仅检查了该组的 51 行。classified.tsv 中引用 SELinux 路径的其他行不在本组内，未作评估。
