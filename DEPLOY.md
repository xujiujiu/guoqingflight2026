# 假期机票价格报告 · 静态部署包

纯静态，无构建、无后端、无 fetch。任意静态资源服务器直接托管本目录即可
（Nginx / OSS / COS / CDN / `python3 -m http.server` 均可）。

## 文件清单
| 路径 | 作用 | 改动频率 |
|---|---|---|
| `index.html` | 默认文档跳转，避免复制两份内容造成漂移 | 几乎不改 |
| `flights.html` | 报告页面与全部样式 | 改版式时 |
| `flights-app.js` | 报告逻辑：矩阵/曲线/走势/表格 | 改交互时 |
| `assets/flight-data.js` | 机票数据归一层（按城市合并、机场码→城市码、地图渲染） | 改口径时 |
| `assets/echarts.min.js` | 图表引擎（已本地化，6.1.0，Apache-2.0） | 升级时 |
| `assets/fonts.css` + `assets/fonts/*.woff2` | 本地字体（拉丁子集 6 档，约 135KB） | 换字体时 |
| `assets/cities-geo.js` | 370 个地级单元边界 GeoJSON（含机场坐标落位所需质心） | 基本不变 |
| `data/flight-all.js` | **主数据**：全部统计数字均由 flights-app.js 的 head() 从数据实算后填入与 15 天日历 | 每次重采 |
| `data/flight-mtop.js` | 首批数据（含 flyai 可售往返打包价，用于对照） | 每次重采 |

## 外部依赖：已归零
字体已本地化到 `assets/fonts/`（Archivo 3 档 + IBM Plex Mono 3 档，共 6 个 woff2、约 135KB），
经 `assets/fonts.css` 引用。**全包不含任何外部 URL**，内网/离线环境可直接托管，
`file://` 双击也能完整渲染。

> 为什么没有中文字体：原先引用的 Google CSS 里 `Noto Sans SC` **只有 latin / latin-ext /
> cyrillic / vietnamese 分片，不含任何中文块**——页面里的中文一直是系统字体
> （macOS PingFang SC、Windows 微软雅黑）在渲染。完整中文字库单档 10MB+，
> 为一份静态报告拉进来不划算，故中文继续交给系统字体栈，字形与之前无变化。

## 缓存建议
两个 `data/*.js` 是**取数时刻的快照**（页脚 `#meta` 会显示采集时间）。重采后必须让客户端拿到新版：

- 首选**文件名带版本**：`flight-all.v20260924.js`，同步改 `flights.html` 里的 `<script src>`；
- 或对 `data/` 设 `Cache-Control: no-cache`，静态资源 `assets/` 可长缓存。

## 数据口径（部署说明里建议保留）
- 价格是「去程该日单程最低价 + 返程该日单程最低价」的**理论组合**，含基建燃油，可能跨承运人，不保证能作为一张往返票按此总价购买。
- 与飞猪同商家可售往返打包价（`data/flight-mtop.js`）通常相差 5–24%，两者不可混比。
- 机场码 `PEK/XIY/PVG` 一类返回 0 而城市码 `BJS/SIA/SHA` 有价，已按城市码补查；同城多机场在展示层合并为一个城市、取最低价。
- 无报价即留空，不以邻日价格顶替。

## 验证
```
cd web && python3 -m http.server 8080   # 打开 http://localhost:8080
```
