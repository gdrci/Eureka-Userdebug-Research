# 22 - 字体、区域设置、输入与资源

[English](22-fonts-locale-input-resources.md)

## 摘要

该组包含 448 个输入行：system_a 中 434 个，odm_a 中 8 个，product_a 中 6 个。它分为三个部分。字体：210 个系统字体条目（204 个 TrueType 或 OpenType 文件、两个 CJK TrueType 集合、四个符号链接），以及六个产品字体。系统字体由 system_a/system/etc/fonts.xml 索引（familyset 版本 23，157 个字体族，24 个别名），默认无衬线字体为 Roboto；产品字体通过 product_a/etc/fonts_customization.xml 添加。区域设置数据：一个带有 tz_version 字符串的 tzdata2021a 包、37 个断字模式（.hyb）文件、两个由 libFFTEm.so 读取的 BMD 二进制表，以及一个指向 com.android.i18n APEX 的 usr/icu 符号链接。输入配置：176 个 Linux 按键布局（.kl）文件、六个按键字符映射（.kcm）和八个输入设备配置（.idc）文件，主要是游戏手柄和遥控器设备，另有模拟器和 VR 条目。所有 443 个普通文件的大小和 sha256 都与记录一致，五个符号链接均可解析。

角色表：`data/subsystems/fonts-locale-input-resources/roles.tsv`（448 个文件：258 个 verified，187 个 observed，3 个 inferred，0 个 unverified）。

## 已验证的声明

- **C01** (verified)：该组有 448 行：434 个位于 system_a，8 个位于 odm_a，6 个位于 product_a；443 个为普通文件，5 个为符号链接。
- **C02** (verified)：每个普通文件行都存在于挂载的镜像中，且大小和 sha256 与记录一致；没有缺失的行。
- **C03** (verified)：五个符号链接为：system_a/system/fonts/DroidSans.ttf 和 DroidSans-Bold.ttf 指向 Roboto-Regular.ttf；NotoColorEmoji.ttf 和 NotoColorEmoji.lnk 指向 FacebookEmoji64.ttf；system_a/system/usr/icu 指向 /apex/com.android.i18n/etc/icu。
- **C04** (verified)：角色类别覆盖全部 448 行：210 个系统字体条目（204 个 TTF/OTF，2 个 TTC，4 个符号链接）、6 个产品字体、1 个 ICU 符号链接、176 个按键布局（171 个以厂商命名，包括 odm_a 中的 6 个；odm_a 中的 1 个 gpio-keys；4 个其他）、8 个 IDC 文件（7 个系统，1 个 odm）、6 个按键字符映射、37 个 hyb 文件、2 个 bmd 文件、2 个 zoneinfo 文件。
- **C05** (verified)：roles.tsv 包含一个表头，以及每个输入行对应的恰好一行（448 个唯一的分区/路径键，无重复，每行四列）。置信度计数：verified 258，observed 187，inferred 3。
- **C06** (verified)：Roboto-Regular.ttf 是一个 TrueType 可变字体，有 18 个表，字体族名为 Roboto，版本为 'Version 3.002; 2020'，轴为 wght 100-900（默认 400）、wdth 75-100、ital 0-1，unitsPerEm 为 2048，共 3362 个字形。
- **C07** (verified)：fonts.xml 首先声明默认字体族：无衬线字体族对字重 100-300 列出 Roboto-Regular.ttf，对 400 列出 RobotoStatic-Regular.ttf，而 sans-serif-condensed 也使用 Roboto。该文件说明第一个字体为默认字体。
- **C08** (verified)：FacebookEmoji64.ttf 是一个 TrueType 彩色位图字体（CBDT/CBLC 表），名称为 'Facebook Emoji'，共 3902 个字形；NotoColorEmoji.ttf 和 NotoColorEmoji.lnk 是指向它的符号链接；fonts.xml 将 NotoColorEmoji.ttf 注册在语言标签 und-Zsye 下。
- **C11** (verified)：product_a/etc/fonts_customization.xml 添加了三个命名字体族：ms-neo-gothic（MSNeoGothic.ttf，字重 400）、optimistic（OptimisticText 静态文件，字重 300/400/500/700/900）以及 optimistic display（OptimisticDisplay-VF.ttf，字重 300/400/500/700/900）。display 字体族为一个 wght 轴只到 800 的文件列出了最高 900 的字重。
- **C12** (verified)：AndroidClock.ttf 是一个 TrueType 字体（sfnt 版本 0x00010000，12 个表），名称为 AndroidClock Regular，版本 1.00000（2012），共 16 个字形。输入中的 MIME 标签 'application/SIMH-tape-data' 以及 file(1) 的结果 'SIMH tape data' 对该文件都不正确。该字体未列在 fonts.xml 中；其用途仅根据名称推断。
- **C13** (verified)：MSNeoGothic.ttf 是一个名为 Microsoft NeoGothic 的 TrueType 字体（版本 2.00，14606 个字形，755716 字节）。它不在 fonts.xml 中，只能通过产品定制字体族 ms-neo-gothic 访问。
- **C14** (verified)：fonts.xml 的版本为 23，包含 157 个 family 元素、24 个 alias 元素，以及 366 个引用 204 个不同文件的 font 元素；全部 204 个文件都存在于 system_a/system/fonts 中，使用的轴标签为 ital、wdth 和 wght。组中有六个字体条目未被引用：AndroidClock.ttf、DroidSans.ttf、DroidSans-Bold.ttf、FacebookEmoji64.ttf、MSNeoGothic.ttf 和 NotoColorEmoji.lnk。
- **C16** (observed)：libandroid.so 包含字面路径 /system/etc/fonts.xml、/system/fonts/、/product/etc/fonts_customization.xml 和 /product/fonts/，与该组字体文件及两个配置文件的位置一致。
- **C17** (verified)：Generic.kl（美国英语 PC 键盘）是一个 Linux 按键布局：包含形如 'key <scancode> <KEYCODE> [flags]' 的按键行（254 行）、形如 'axis <code> <AXIS>' 的轴行（13 行）以及形如 'led <code> <LED>' 的 LED 行（11 行）。扫描码 1 映射到 ESCAPE，2 映射到 1，14 映射到 DEL，28 映射到 ENTER。
- **C19** (observed)：大多数按键布局文件命名为 Vendor_<VID>_Product_<PID>.kl（USB 厂商和产品 ID，可选 _Version_<hex> 后缀），并在第一条注释中描述设备（游戏手柄、键盘、遥控器、蓝牙 AVRCP）。与 USB ID 关联的厂商名称（例如 045e 对应 Microsoft，054c 对应 Sony）是根据一般的 USB ID 知识推断的，并非来自文件本身。
- **C20** (verified)：odm_a/usr/keylayout/gpio-keys.kl 映射六个按键：115 VOLUME_UP、114 VOLUME_DOWN、102 HOME、227 TV_INPUT、528 FOCUS、766 CAMERA。其文件头是 Linux Foundation 许可证块，因此该描述来自按键名称。
- **C21** (verified)：按键字符映射：qwerty.kcm 为 ALPHA 类型，包含 54 个键块；qwerty2.kcm 为 ALPHA 类型；Generic.kcm 为 FULL 类型，包含 107 个键块；Virtual.kcm、Vendor_18d1_Product_0200.kcm 和 Vendor_18d1_Product_5018.kcm 均为 FULL 类型。每个键块按修饰键列出 label、number、base、shift、capslock、alt 以及 shift+alt 条目。
- **C22** (verified)：qwerty.idc（模拟器键盘 #1）设置 touch.deviceType=touchScreen、touch.orientationAware=1、keyboard.layout=qwerty、keyboard.characterMap=qwerty、keyboard.orientationAware=1、keyboard.builtIn=1、cursor.mode=navigation 和 cursor.orientationAware=1。
- **C23** (verified)：整个组中使用的 IDC 属性键包括 keyboard.layout、keyboard.characterMap、keyboard.builtIn、keyboard.doNotWakeByDefault、device.internal、device.type、touch.deviceType、cursor.mode、audio.mic 以及 sensor.<accelerometer|gyroscope>.<reportingMode|maxDelay|minDelay|power>。Vendor_054c_Product_09cc.idc（Sony DS4 运动传感器）将加速度计 power 设为 1.5，陀螺仪 power 设为 0.8，minDelay 为 5000，maxDelay 为 100000。
- **C24** (verified)：odm_a/usr/idc/Vendor_2833_Product_0160.idc（113 字节）选择布局和字符映射 oculus-device，设置 device.internal=1 和 device.type=vrPeripheral。其引用的 odm_a/usr/keylayout/oculus-device.kl（按键 172 HOME，按键 159 FORWARD）和 odm_a/usr/keychars/oculus-device.kcm（FULL 类型；按键 FORWARD 的 base 回退为 HOME）不在本组中；它们属于 sensors-mcu-meta-hal。
- **C25** (verified)：SELinux 文件上下文为 odm/vendor 按键布局、按键字符映射和 IDC 路径添加了标签（vendor_keylayout_file、vendor_keychars_file、vendor_idc_file）。zoneinfo 路径的标签为 system_zoneinfo_file（/system/usr/share/zoneinfo）和 zoneinfo_data_file（/data/misc/zoneinfo）。唯一与字体相关的上下文是 /data/fonts/files（font_data_file）；没有条目匹配 /system/fonts 或 /product/fonts。
- **C26** (verified)：对于 C25 中的标签以及 tzdatacheck_exec，SELinux 类型声明均存在于 plat_sepolicy.cil 中。
- **C27** (verified)：init 相关引用：init.rc 创建 /data/fonts（root，需要加密）、/data/fonts/files 和 /data/fonts/config（system），以及 /data/misc/zoneinfo；init.rc 在 post-fs-data 阶段运行 /system/bin/tzdatacheck /apex/com.android.tzdata/etc/tz /data/misc/zoneinfo inside；init.environ.rc 导出 ANDROID_I18N_ROOT=/apex/com.android.i18n 和 ANDROID_TZDATA_ROOT=/apex/com.android.tzdata。
- **C28** (verified)：没有任何 init .rc 文件或 ueventd 文件引用 usr/keylayout、usr/keychars、usr/idc、hyphen-data、share/bmd 或 usr/icu。init 引用的唯一字体路径是 /data/fonts 目录设置。
- **C29** (verified)：system_a/system/usr/share/zoneinfo/tzdata 是一个 bionic 格式的 tzdata 包：魔数 'tzdata'，版本 2021a，索引偏移 24，数据偏移 30860，末尾偏移 491837 与文件大小相等，共 593 条 52 字节的索引记录；第一条记录是 Africa/Abidjan（起始 0，长度 156）。
- **C30** (verified)：tz_version 保存字符串 005.001|2021a|001（三个以竖线分隔的字段；字段含义是推断出的：tzcode 版本 | tzdata 版本 | 修订号）。
- **C31** (observed)：libc.so 包含 tzdata 搜索路径 /apex/com.android.tzdata/etc/tz/tzdata、/data/misc/zoneinfo/current/tzdata 和 /system/usr/share/zoneinfo/tzdata，以及错误文本 'couldn't find any tzdata when looking for %s!'。仅凭这些字符串无法确定搜索顺序。
- **C32** (verified)：system_a/system/usr/hyphen-data 中的全部 37 个断字模式文件都以字节 68 79 ad 62 开头（小端 u32 为 0x62ad7968），并且在所有情况下，偏移 0x14 处的 u32 都等于文件大小。hyph-en-us.hyb（59802 字节）的头部字为 0x08=24、0x0C=96、0x10=53276、0x14=59802；文件名给出了语言标签。
- **C35** (verified)：usr/icu 是指向 /apex/com.android.i18n/etc/icu 的符号链接，init.environ.rc 导出 ANDROID_I18N_ROOT=/apex/com.android.i18n。提供此路径的 APEX 是 system_a/system/apex/com.android.i18n.apex（36649148 字节，包含 36581376 字节的 apex_payload.img）；其载荷未被解包，因此 ICU 数据布局未经验证。

## 已更正措辞的声明

- **C10** (verified)：产品字体：OptimisticDisplay-VF.ttf 是一个可变 TrueType 字体，名称为 'Optimistic Disp VF Cyrl Md'（版本 2.000，wght 200-800，默认值 500，341 个字形，unitsPerEm 2000）。OptimisticText-{Light,Regular,Medium,Bold,Black}.ttf 是静态 TrueType 字体（版本 1.000，430 个字形，unitsPerEm 2000），属于排版字体族 'Optimistic Text App'（name ID 16，Regular 和 Bold 中不存在）。它们的完整名称分别为 'Optimistic Text App Light'、'Optimistic Text App'（Regular）、'Optimistic Text App Medium'、'Optimistic Text App Bold' 和 'Optimistic Text App ExtraBold'（文件名为 Black）。这六个文件均位于 product_a/fonts 中。
- **C15** (observed)：fonts.xml 说明其选择规则：字体依次按包含脚本在内的完整 BCP-47 语言标签匹配，然后按语言匹配，最后按顺序匹配（即第一个包含该字形的字体）。出现顺序也用于在字重匹配时打破平局。
- **C18** (verified)：在组内 176 个按键布局文件中，指令统计为：key 2986，axis 1294，led 38，sensor 96。传感器类型为 ACCELEROMETER 和 GYROSCOPE（各 48 个）。轴名称包括 X、Y、Z、RZ、HAT_X 和 HAT_Y，每个都出现在 176 个文件中的 162 个里（system_a 的 169 个文件中有 156 个，odm_a 的 7 个文件中有 6 个）。
- **C33** (observed)：libandroid_runtime.so 包含字面前缀 /system/usr/hyphen-data/hyph-（observed）。该前缀与按语言标签在运行时查找断字文件的做法一致（inferred；未追踪查找代码）。
- **C34** (verified)：RFFstd_501.bmd（19586 字节）和 RFFspeed_501.bmd（12862 字节）都以一个小端 u32 开头，其值乘以 2 等于文件大小（9793 和 6431）。字符串 usr/share/bmd/RFFstd_501.bmd 存在于 libFFTEm.so 中；字符串 RFFspeed 未出现在 lib、lib64、bin 或 framework 目录的库中。libFFTEm.so 导出 945 个已定义的动态符号，大多数带有前缀 bts_（297）、bbs_（221）、bbf_（211）、bim_（120）、bpi_（56）和 btk_（36，其中 10 个以 btk_SDK_ 开头），另外还有四个无前缀名称，包括 JNI_OnLoad。

## 已排除的声明

| ID | 声明 | 原因 |
|---|---|---|
| C09 | NotoSansCJK-Regular.ttc 是包含 10 个字体面的 TrueType/CFF 集合（Noto Sans CJK JP、KR、SC、TC、HK 以及四个 Noto Sans Mono CJK 字体面）；NotoSerifCJK-Regular.ttc 有 4 个字体面（JP、KR、SC、TC）。fonts.xml 按索引选择字体面：ja=0、ko=1、zh-Hans=2、zh-Hant,zh-Bopo=3。 |  |

## 缺口

- 挂载的镜像中不存在 product_a 和 odm_a 的 SELinux 文件上下文；仅找到 system_a/system/etc/selinux/plat_file_contexts 和 vendor_a/etc/selinux/vendor_file_contexts，因此产品字体以及超出 vendor 模式的 odm 路径标签尚未确定。
- APEX 载荷（com.android.i18n apex_payload.img 和 com.android.tzdata apex_payload.img）未被解包；未检查 ICU 数据文件以及 /apex 下处于活动状态的 tzdata 副本。系统中的 tzdata 副本为 tzdata2021a；尚不能确定 APEX 中的副本是否更新。
- BMD 表（RFFstd_501.bmd、RFFspeed_501.bmd）除大小头关系之外的用途和文件格式尚未确定，libFFTEm.so 的功能作用（符号前缀 bts_、bbs_、bbf_、bpi_、btk_SDK_）也未确定。在所搜索的二进制文件中未找到对 RFFspeed_501.bmd 的字符串引用。
- 断字（.hyb）格式除魔数、0x14 处的大小字段以及 C32 中列出的头部字之外未被解码；未提取模式数量和语言覆盖范围。
- 未计算 Noto 字体各文字系统的字形覆盖率；仅解析了字形数量（maxp）和名称表。未模拟 fonts.xml 隐含的回退顺序。
- Vendor_<VID>_Product_<PID> 按键布局的厂商名称是根据一般的 USB ID 知识推断的，未与 USB ID 数据库核对；171 个以厂商命名的文件是根据其第一行注释描述的，仅完整阅读了一个样本。
- AndroidClock.ttf（时钟数字）以及 DroidSans 和 DroidSans-Bold（旧版别名）的用途仅根据名称推断；在 fonts.xml、init 或 SELinux 文件中未找到对它们的引用。
- AndroidClock.ttf 的 MIME 标签（application/SIMH-tape-data），以及 40 个字体、断字、BMD 和 tzdata 文件的 MIME 标签（application/octet-stream 或标签错误的 'data' 类型）是输入分类，仅在已验证的情况下才更正；输入分类未被重写。
- Vendor_2833 IDC 引用的 oculus-device.kl 和 oculus-device.kcm 文件属于 sensors-mcu-meta-hal 组；其完整行为不在本组范围内。
- 产品 fonts_customization.xml 在运行时的效果（libandroid.so 如何应用定制字体族并解析字重）未被追踪；仅验证了配置内容和路径字符串。
- 按键布局、按键字符映射和 IDC 文件的运行时加载路径（哪个进程读取 /system/usr/keylayout 等）未被追踪；仅找到的静态引用是 SELinux 标签和路径的模式。
