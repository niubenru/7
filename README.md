# 实验5 目标跟踪 —— 代码运行说明

用 OpenCV 封装的跟踪算法实现**单目标跟踪**（KCF / CSRT）和**多目标跟踪**（MultiTracker）。

## 1. 环境准备

需要 Python 3.8+，安装依赖（`opencv-contrib-python` 才带 KCF/CSRT/MultiTracker）：

```bash
pip install opencv-contrib-python
```

> 注意：装 `opencv-python`（不带 contrib）时部分算法会缺失，请务必装 contrib 版本。
> 如果摄像头不可用或想快速看效果，先运行 `python make_test_video.py` 生成一个测试视频。

## 2. 单目标跟踪（single_tracker.py）

```bash
# 摄像头 + KCF（最快）
python single_tracker.py --source 0 --algo KCF

# 视频文件 + CSRT（精度更高，速度较慢）
python single_tracker.py --source test_video.mp4 --algo CSRT

# 免交互：直接指定初始框 x,y,w,h（无显示环境/自动测试用）
python single_tracker.py --source test_video.mp4 --algo KCF --bbox 60,80,50,70

# 无界面模式（不弹窗，服务器/批量测试用，需配合 --bbox）
python single_tracker.py --source test_video.mp4 --algo KCF --bbox 60,80,50,70 --headless
```

**操作**：启动后程序停在第一帧，鼠标框选目标 → 回车/空格确认；
运行中按 `q` 退出，按 `r` 重新框选目标（相当于重新初始化）。

窗口左上角实时显示算法名、处理帧率（FPS）和每帧跟踪耗时，可用于对比不同算法的速度。

## 3. 多目标跟踪（multi_tracker.py）

```bash
# 摄像头 + KCF，依次框选 3 个目标
python multi_tracker.py --source 0 --algo KCF --num 3

# 视频文件 + CSRT，2 个目标
python multi_tracker.py --source test_video.mp4 --algo CSRT --num 2

# 免交互：直接指定多个初始框
python multi_tracker.py --source test_video.mp4 --algo KCF --bbox 60,80,50,70 --bbox 520,150,55,85
```

**操作**：启动后逐个弹出框选窗口，每个目标框选后回车/空格确认；
运行中按 `q` 退出，按 `r` 重新框选所有目标。
每个目标用不同颜色标注（target1、target2…）。

## 4. 算法选择建议（写入报告的对比结论）

| 算法 | 速度 | 精度/鲁棒性 | 说明 |
|------|------|------------|------|
| MOSSE | 极快（100+ FPS） | 低 | 最早的 相关滤波，适合速度优先演示 |
| KCF | 快（50~100 FPS） | 中 | 核相关滤波；不抗遮挡、不抗形变 |
| CSRT | 慢（10~30 FPS） | 高 | 通道+空间可靠性，抗形变效果好，适合精度优先 |
| MIL | 中 | 中 | 多实例学习，抗遮挡稍好 |

**典型结论**：KCF 速度快但目标出画面/遮挡后易跟丢且无法自恢复；CSRT 慢但对旋转、形变更稳；
MultiTracker 本质是"多个独立单目标跟踪器"，目标间不会做数据关联，因此目标交叉重叠后可能互换 ID——
这正是基于检测的多目标跟踪算法（如 SORT/ByteTrack）要解决的问题。

## 5. 常见问题

- **cv2.error: TrackerKCF / MultiTracker not found** → 装的是非 contrib 版：`pip uninstall opencv-python` 后重装 `opencv-contrib-python`。
- **OpenCV 5.x 报错找不到 legacy 跟踪器** → 本实验代码需 OpenCV 4.x contrib：`pip install "opencv-contrib-python==4.12.0.88"`。
- **摄像头黑屏/打不开** → 换 `--source 1` 或使用视频文件。
- **selectROI 窗口在后台** → 程序可能被窗口焦点挡住，点击任务栏即可。

## 6. 版本兼容说明（重要）

不同 OpenCV 版本的跟踪器 API 有差异，代码已做兼容处理：

- **4.5~4.8**：`cv2.MultiTracker_create()` + `add(tracker)` + `init(frame, boxes)`；KCF 在 `cv2.TrackerKCF_create`
- **4.9+（如 4.12）**：跟踪器移入 `cv2.legacy.*`；MultiTracker 用 `cv2.legacy.MultiTracker_create()`，且 `add(tracker, frame, bbox)` 需要带图和框；注意 4.12 中 KCF/CSRT 同时有现代版与 legacy 版，但 **legacy MultiTracker 只接受 legacy 跟踪器**
- **5.0+**：legacy 跟踪器被移除，本实验请固定使用 4.x 版本

> `test_trackers.py` 是开发用的自动化回归测试（生成合成视频、以 IoU 评估跟踪精度），交付运行时可忽略。
