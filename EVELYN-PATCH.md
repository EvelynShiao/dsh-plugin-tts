# 大肥鱼补丁版说明（Evelyn 的 fork）

基于上游 `@memef1f1y/dsh-plugin-tts` **0.6.7**（仓库 1624318455/dsh-plugin-tts）的本地补丁副本。
**这份代码 = 本机正在运行的原样版本**（不是重新构建的，`lib/*.mjs` 即运行时源码）。
建立于 2026-11-03，目的：防止 `pnpm update` / 插件重装被上游原版冲掉补丁；也供手机端直接安装。

## 补了什么（全部带 `2026-11-03` 注释标记，好在源码里搜）

1. **断句修复**（`lib/index.mjs` 的 `splitText()` → `safeEnd()`）
   空格仅当**后一个字符非 CJK** 时才算切点；边界集扩充 `，、。！？；,!?;：:—…·`。
   修掉「DSH|里」被劈开的抽象断句。
2. **emoji 剥离**（`lib/client.js` 的 `plainText()`）
   朗读前剥离 `\p{Extended_Pictographic}` + 肤色修饰符 + FE0E/FE0F + ZWJ + 20E3（u flag）；
   `rawMarkdown` 分支保持原样不动。
3. **CosyVoice 服务三开关**（设置 → 插件 → 语音 → CosyVoice 配置）
   - 使用时自动拉起（默认开）：朗读 / 刷新音色前探 `/health`，不通则 spawn 桥接，轮询最多 30 秒；
     仅本机地址（127.0.0.1 / localhost / ::1）触发。
   - 开机自动拉起（默认关）：勾选写 `Startup\大肥鱼声线桥接.bat`，取消即删；宿主启动时同步一次。
   - 空闲自动卸载分钟（默认 10，0 = 常驻）：桥接在跑则经 `POST /config` 即时下发，否则下次拉起带环境变量生效。
4. **配置落盘健壮性**（`saveHostCosySettings`）
   写入失败重试一次并记录结果，路由以 `persisted` 回执（修「UI 改了设置、实际没落盘、
   重启后打回默认」的静默失败）。
5. **宿主接口扩展**：`GET /dsh-tts-api/cosy-config` 增返 `autoStart` / `startAtBoot` / `idleUnloadSec`；
   `POST /dsh-tts-api/cosy-config-save` 接受这三个键（`null` = 清除回默认）。

## 配置与路径（本机默认值）

- 服务级配置：`~/.dsh/tts-rvc/settings.json` 的 `cosy` 节（`DSH_TTS_HOME` 可覆盖目录）
- 桥接解释器 / 脚本默认路径：`lib/index.mjs` 的 `COSY_SVC_DEFAULTS`
  （默认 `C:\Users\Evelyn\miniconda3\envs\GPTSoVits\python.exe` + `C:\Users\Evelyn\GPT-SoVITS\da_fei_yu_weights\dafeiyu_bridge.py`；
  **换机器要改这里**）
- 桥接本体（`dafeiyu_bridge.py`）不在本仓库，属于 GPT-SoVITS 侧

## 上游升级时的重打流程

1. 拉上游新版到临时目录，对比 `lib/index.mjs`、`lib/client.js`
2. 把上面 5 处改动照搬到新版（源码里搜 `2026-11-03` 标记最快）
3. 覆盖安装目录 `.dsh\profiles\web\node_modules\@memef1f1y\dsh-plugin-tts\`
4. 重启 DSH

## 安装（手机 / 新机器）

```bash
dsh plugin --profile web add https://github.com/EvelynShiao/dsh-plugin-tts
# 国内可换 GitCode 镜像地址
```
