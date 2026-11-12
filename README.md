# convert_negative_film · 负片去色罩校正

纯前端的负片（底片）去色罩工具：自动识别胶片片基色，逐通道扣除色罩并把负像还原为正像。所有计算都在浏览器本地完成，图片不会上传到任何服务器。

处理思路参考 [QingSi Lab 直观胶片](https://lab.qingsiphotograph.com/intuitive-film/)，交互与本仓库实现独立。

## 功能

- **片基色识别**：边缘取样 + 画面内容兜底双策略，自动给出片基 RGB 与置信度；也可手动点击取样（采样半径 4–32px）
- **正负片自动判断**：检测到疑似正片时提示，可一键切换正片直通
- **滑块对比**：载入后默认开启「简单反转 vs 片基校正」对比，拖动分割线查看差异
- **参数调整**：8 项调色滑块，支持参数 JSON 导出 / 导入 / 重置
- **多图队列**：最多 12 张，缩略图切换与删除，批量应用调色/片基到全部或仅未处理图片
- **裁剪**：自由比例与 1:1 / 4:3 / 3:2 / 16:9，随队列持久化
- **导出**：JPEG / PNG / WebP / TIFF / 原图格式；批量支持保存到文件夹（File System Access）、自动分卷 ZIP、单一 ZIP
- **全分辨率查看**：灯箱查看处理结果，Esc 或点击空白关闭

## 支持的导入格式

JPEG、PNG、WebP、TIFF、BMP、AVIF。浏览器原生不支持的 TIFF 由 UTIF.js 兜底解码；RAW 格式不支持，会给出提示。

## 本地运行

页面为纯静态资源，无需构建，但不能直接双击打开（`file://` 下 canvas 取像素会被安全策略限制）。用任意静态服务器打开即可：

```bash
python3 -m http.server 8899
# 浏览器访问 http://localhost:8899/
```

## 目录结构

```
index.html      页面结构
app.js          全部业务逻辑（解码、片基识别、渲染管线、队列、导出）
styles.css      样式（含深色模式）
lib/
  UTIF.js       TIFF 解码（MIT）
  pako.min.js   zlib 解压，UTIF 依赖（MIT）
  jszip.min.js  ZIP 打包（MIT）
```

## 渲染管线

1. 读取原始像素，按片基色做逐通道归一化 `(v - black) / (base - black)`
2. 反相并按通道白平衡增益校正偏色
3. 自动色阶拉伸，再应用 8 项调色参数（曝光、对比度、高光、阴影、饱和、色温、色调、Gamma）
4. 输出前按裁剪框裁切并编码

## 浏览器要求

Chrome / Edge / Firefox / Safari 近两年版本。文件夹直写（File System Access API）目前仅 Chromium 系支持，其他浏览器会自动回退为分卷 ZIP。

## 第三方库

- [UTIF.js](https://github.com/photopea/UTIF.js) — MIT
- [pako](https://github.com/nodeca/pako) — MIT
- [JSZip](https://stuk.github.io/jszip/) — MIT
