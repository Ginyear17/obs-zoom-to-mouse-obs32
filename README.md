# obs-zoom-to-mouse (OBS 32 修复版备份)

> **非官方版本，仅作个人备份用途。**
>
> - 原作者：**BlankSourceCode** — https://github.com/BlankSourceCode/obs-zoom-to-mouse
> - 本仓库只做了一处改动：让脚本能在 OBS Studio 32 上正常加载。
> - 脚本版权归原作者所有，本仓库不主张任何权利。
> - 如果上游已发布修复版本，请优先使用官方版本。

本仓库存在的唯一目的：保存一份在 OBS 32 下可用的脚本副本，方便换电脑或日后重装时直接取用。

## 为什么有这个仓库

上游脚本基于 OBS 29 开发，在 **OBS Studio 32** 上加载会报错：

```
[obs-zoom-to-mouse.lua] Failed to call script_update for obs_lua_script_update:
[string "..."]:556: attempt to call field 'obs_sceneitem_get_info' (a nil value)
```

原因：`obs_sceneitem_get_info()` / `obs_sceneitem_set_info()` 是已废弃接口，
OBS 32 已把它们从 Lua 绑定中移除，只保留 `*_info2` 版本。
（核对 `bin/64bit/obs.dll`：`obs_sceneitem_get_info`、`obs_sceneitem_set_info` 不存在，
`obs_sceneitem_get_info2`、`obs_sceneitem_set_info2` 存在。）

两个函数操作的是同一个 `obs_transform_info` 结构（`get_info2` 额外填充
`crop_to_bounds`），因此可以直接等价替换。

## 修复内容

见 `fix.patch`：在脚本开头加一层兼容别名，OBS 32+ 与旧版本都能用，
不改动脚本其余逻辑。

```lua
if obs.obs_sceneitem_get_info == nil and obs.obs_sceneitem_get_info2 ~= nil then
    obs.obs_sceneitem_get_info = obs.obs_sceneitem_get_info2
end
if obs.obs_sceneitem_set_info == nil and obs.obs_sceneitem_set_info2 ~= nil then
    obs.obs_sceneitem_set_info = obs.obs_sceneitem_set_info2
end
```

## 安装

1. 下载本仓库的 `obs-zoom-to-mouse.lua`
2. OBS → 工具 → 脚本 → `+` 添加该文件
3. 显示器采集源建议设置：
   - 位置对齐：左上
   - 包围盒类型：缩放到内边界
   - 边界框内对齐：左上
   - 裁剪：全 0
4. 设置 → 快捷键 → 绑定 `Toggle zoom to mouse`

## 主要参数

| 参数 | 作用 |
|---|---|
| Zoom Factor | 放大倍数 |
| Zoom Speed | 变焦动画速度（每帧进度增量） |
| Auto follow mouse | 放大后自动跟随鼠标 |
| Follow outside bounds | 鼠标移出源范围时仍跟随 |
| Follow Speed | 跟随的插值速度，越大越跟手 |
| Follow Border | 距边缘多少 % 时开始跟随；0% = 到最边缘才跟，50% = 一直跟 |
| Lock sensitivity | 距目标多少像素内算"锁定"，越大越容易停 |
| Auto Lock on reverse direction | 反向移动鼠标立即停止跟随（摄像机手感） |
| Allow any zoom source | 允许选择非显示器采集源（需手动填位置） |
| Set manual source position | 手动指定源的 X/Y/宽/高/缩放/显示器尺寸 |

## 与上游的关系

上游 issue #72 与 PR #97 已有人报告并提交同类修复。
若上游已合并，请优先使用官方版本，本仓库可弃用。
