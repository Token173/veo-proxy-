# Veo 中转 API：国内直连 Veo 视频生成，附豆包 Seedance 与 SD 视频模型对比

Veo 是国内做短视频素材的人最常提的视频模型之一：文本直接出片、支持首尾帧控制、原生带音频。问题还是老一套——Google 的视频端点在境内连不上，异步任务一轮轮轮询，端口一断就白等。

Token173（https://token173.net）把 Veo 和豆包 Seedance、SD 2.5 视频放在同一套接口后面，国内网络直接提交任务，轮询拿结果。

## 可用视频模型

| 模型 ID | 特点 |
|---|---|
| `veo_3_1` | 文本/图生视频，原生音频，支持首尾帧 |
| `doubao-seedance-2-0-fast-260128` | 豆包快速版，出片快，价格低 |

## 提交任务

视频是异步的：先提交，拿任务 ID，再轮询。

```bash
curl -X POST https://token173.net/v1/videos/generations \
  -H "Authorization: Bearer sk-你的令牌" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "veo_3_1",
    "prompt": "清晨的海边公路，镜头跟随一辆自行车缓慢前进，侧逆光",
    "aspect_ratio": "16:9",
    "duration": 8
  }'
```

返回里会带 `task_id`，接着查：

```bash
curl https://token173.net/v1/videos/generations/TASK_ID \
  -H "Authorization: Bearer sk-你的令牌"
```

Python 轮询：

```python
import time, requests

H = {"Authorization": "Bearer sk-你的令牌"}
r = requests.post(
    "https://token173.net/v1/videos/generations",
    headers=H,
    json={
        "model": "veo_3_1",
        "prompt": "清晨的海边公路，镜头跟随一辆自行车缓慢前进，侧逆光",
        "aspect_ratio": "16:9",
        "duration": 8,
    },
).json()

tid = r["task_id"]
while True:
    s = requests.get(f"https://token173.net/v1/videos/generations/{tid}", headers=H).json()
    if s["status"] in ("succeeded", "failed"):
        print(s)
        break
    time.sleep(10)
```

轮询间隔 10 秒够用，太频繁没意义，视频生成本来就要几十秒到几分钟。

## 图生视频

把一张图作为首帧，让它动起来：

```json
{
  "model": "veo_3_1",
  "prompt": "镜头缓慢推近，衣服轻微摆动",
  "image_url": "https://你的图床/cover.jpg",
  "aspect_ratio": "9:16",
  "duration": 6
}
```

电商主图转短视频、海报转动态素材，这个用法最省事。

## 选型建议

| 需求 | 模型 |
|---|---|
| 高质量品牌短片、要音频 | `veo_3_1` |
| 一天几十条社媒短视频 | `doubao-seedance-2-0-fast-260128` |
| 图生视频做商品展示 | `veo_3_1`，9:16 竖版 |

先低价模型跑通脚本，确定提示词有效再换 Veo 出成品，这条路径最省钱。

## 提示词要点

视频 prompt 和图像不一样，重点在**镜头怎么动**：

- 景别：近景、中景、大全景
- 运镜：推、拉、摇、跟随、固定机位
- 节奏：缓慢、快速切换
- 光线：侧逆光、顶光、夜景霓虹

写成"镜头跟随一辆自行车缓慢前进"比"一辆自行车在路上"有效得多。

## 计费

视频按条数和时长计，不同模型、分辨率、时长单价不同，后台能查到当前价格。账户 2 元 = 1 美元额度，注册送 0.2 美元，够跑几条测试。

## 常见问题

**Q：任务一直排队？**
高峰期会排。别一次性提交几十个，分批提交、拿到结果再交下一批。

**Q：返回失败但扣费了？**
失败任务不计费，后台能看到明细。真扣了截图找客服处理。

**Q：能生成多长？**
按模型支持的最长时长来，常见 5 到 10 秒。更长的片子分段生成再剪。

**Q：音频能单独拿吗？**
Veo 返回的视频带音轨，需要单独提取就用 ffmpeg 拆 `-vn -acodec copy`。

**Q：竖版视频怎么出？**
`aspect_ratio` 填 `9:16`，短视频平台用这个比例。

## 相关

- 文档站：https://docs.token173.net
- 图像生成：[Nano-Banana-api](https://github.com/Token173/Nano-Banana-api)
- GPT 图像：[gpt-image-2](https://github.com/Token173/gpt-image-2)
- 多模型聚合：[2026-Token173-Claude-ChatGPT-Grok-Gemini-AI-API](https://github.com/Token173/2026-Token173-Claude-ChatGPT-Grok-Gemini-AI-API)
