# 粒子圣诞树 · Merry Christmas 2026

单文件 Three.js 粒子圣诞树。分层的枝条 + 金色针叶 + 缠绕的螺旋灯带 + 树顶金星 + 装饰球 +
树根礼物盒 + 飘雪 + 星空,带摄像头手势控制。BGM 是 WebAudio 现场合成的八音盒版 Jingle Bells,
烟花和爆炸音效同样是现场合成,不需要任何音频文件。

---

## 快速开始

### 方式一:双击 `start.bat`(推荐,手势最稳)

需要本机装了 Python 3。脚本会在 8080 端口起一个静态服务器并自动打开浏览器。

```
http://localhost:8080/
```

### 方式二:直接双击 `index.html`

树、雪花、星空、音乐、鼠标拖拽全部正常。

摄像头方面:Chrome / Edge 在 `file://` 下**会弹出权限请求**,点"允许"后手势一般可以用。
但 `file://` 不是能按站点记住的 origin,每次打开可能都要重新授权;某些环境(远程桌面、自动化)会直接拒绝。
所以想稳定用摄像头,还是走 `start.bat` 的 localhost。

> 发给别人时,把整个 `christmas-tree` 文件夹压缩发过去,让对方跑 `start.bat`。
> 对方机器也要有 Python;没有的话装一个,或者用任何静态服务器指向这个目录也行。

### 方式三:`mobile.html`(手机 / 微信)

`mobile.html` 是给手机看的独立版本,单文件、离线可用,Three.js 已经内联进去,
不依赖任何 CDN,丢到任何静态服务器上打开就能看。

- 双击 `mobile.html` 也能直接看,但没有声音(浏览器禁止 `file://` 自动播放音乐)
- 想发微信,需要一个 **HTTPS 网址**(微信内置浏览器打不开 `file://` 和 `http://局域网IP`)

和电脑版的区别:

| | 电脑版 `index.html` | 手机版 `mobile.html` |
|---|---|---|
| 体积 | 75KB(依赖 CDN) | 618KB(全部内联) |
| 依赖网络 | 是 | 否 |
| 摄像头手势 | 有 | 关掉(手机上不好用) |
| 交互 | 鼠标拖拽 / 滚轮 | 单指拖拽 / 双指缩放 / 单击烟花 / 双击星云 |
| 粒子量 | 22000 | 10000 |
| 取景 | 按高度 | 竖屏按宽度反算距离,整棵树不会被裁 |

手机版是从电脑版自动生成的:`_mk_mobile.py` 读 `index.html`,做定点替换后输出,
再用 esbuild 把 Three.js 打成 IIFE 内联进去。改完 `index.html` 要重新跑一遍生成。

**首次打开会比较慢**:Three.js 从 CDN 拉,手势模块要下载约 18MB 的 wasm 和模型文件(之后走缓存)。
不点"允许摄像头"就不会下载手势那部分。

---

## 操作

| 操作 | 效果 |
|---|---|
| 张开五指 | 整棵树爆散成金色星云 |
| 握拳 | 所有粒子聚回树形,树顶金星闪一下并"叮"一声 |
| 手掌左右平移 | 旋转视角 |
| 拇指 + 食指捏合/张开 | 拉近 / 拉远 |
| 鼠标拖拽 | 旋转视角 |
| 滚轮 | 缩放 |
| 点击画面 | 朝那个位置打一发烟花,火箭升空后爆开 |
| 右上角 ♪ | 静音 / 取消静音 |
| 右上角 ✋ | 开 / 关摄像头手势 |

不做任何操作时,树会缓慢自转;爆散后 2 秒没有手势输入,会自动聚回树形。

---

## 想改的话

所有可调参数都在 `index.html` 里 `CONFIG` 这个对象中:

```js
const CONFIG = {
  particles: 22000,      // 树粒子数
  treeHeight: 8.4,       // 树高
  treeRadius: 4.2,       // 树底半径
  ornaments: 64,         // 装饰球数量
  snowfall: 1000,        // 雪花数量
  stars: 700,            // 背景星点
  gifts: 5,              // 树根礼物盒
  autoSpin: 0.055,       // 自动旋转速度
  text: {
    main: 'Merry Christmas',
    sub: '2026 · To xlr',
  },
  camera: { radius: 17, polar: 1.44, minRadius: 9, maxRadius: 30 },
  gesture: {
    enabled: true,
    fps: 24,               // 检测频率
    openThreshold: 0.60,   // 越大越难触发爆散
    fistThreshold: 0.38,   // 越大越容易收回树形
    debounceFrames: 2,
    autoReturnMs: 2600,    // 松手后多久自动回树
  },
  music: { enabled: true, bpm: 92, volume: 0.42 },
};
```

### 螺旋灯的密度

在 `CONFIG` 下面几行:

```js
const SPIRAL_ARMS = 6;      // 螺旋灯带条数
const SPIRAL_TURNS = 4.6;   // 每条从底绕到顶的圈数
```

粒子怎么分配在"树叶"和"灯带"之间,在生成循环里:

```js
const onSpiral = Math.random() < 0.30;   // 30% 是灯带,其余长在枝条上
```

调大 → 灯带更粗更明显,树更"线框";调小 → 树更饱满,灯带更含蓄。

枝条数量在更上面一行,改它就能决定树有几层:

```js
const BRANCHES = 26;   // 枝条数,调大树更蓬松,调小更清爽
```

### 换祝福文字

改 `CONFIG.text.main` 和 `CONFIG.text.sub`。

### 换曲子

改 `MELODY` 数组。格式是 `['音名', 拍数]`:

```js
['E5', 1], ['G5', 1], ['C5', 1.5], ['D5', 0.5], ['E5', 4],
```

音名支持 `C4` ~ `B6` 以及 `#` 升号。改 `CONFIG.music.bpm` 调速度。

### 换配色

- 树叶和灯带的颜色:`PALETTE` 数组 + `DEEP_GOLD` / `BRIGHT_GOLD`
- 装饰球:`ORN_COLORS`
- 礼物盒:`GIFT_COLORS`
- 背景和雾:`renderer.setClearColor` 与 `FogExp2`
- 辉光强度:`new UnrealBloomPass(v2, strength, radius, threshold)`

### 关掉手势

`CONFIG.gesture.enabled = false`,就不会再请求摄像头。

---

## 常见问题

**打开是黑屏 / 树没出来**
1. 检查网络 —— Three.js 从 `cdn.jsdelivr.net` 加载,断网就打不开。F12 → Console 看有没有 404。
2. `start.bat` 的窗口不能关,关了服务器就停了。

**手势没反应**
1. 优先用 `http://localhost:8080/` 打开。
2. 浏览器地址栏左侧的摄像头图标要允许。
3. 摄像头被其他软件占用(微信视频、腾讯会议)也会失败,先关掉。
4. 手离摄像头太远或光线太暗,MediaPipe 认不出。手距镜头 40~80cm,正面光。
5. 第一次要等右下角小窗出现,模型加载大概几秒。

**音乐不出声**
浏览器禁止自动播放,必须先点一下页面(开场那个"点击任意处开始"就是干这个的)。
右上角 ♪ 发暗 = 静音中,再点一下。

**掉帧**
把 `CONFIG.particles` 从 15000 降到 8000,`snowfall` 降到 500。
Blur 太重就把 bloom 的 strength 从 0.58 调到 0.4。

---

## 文件说明

```
christmas-tree/
├── index.html     电脑版主程序(单文件,所有逻辑都在这,依赖 CDN)
├── mobile.html    手机 / 微信版(单文件自包含,618KB,不用网络)
├── start.bat      一键起本地服务器(摄像头用)
├── _mk_mobile.py  从 index.html 生成 mobile.html 的脚本
└── README.md      本文件
```

---

## 技术实现摘要

- **树形**:26 根枝条从树身向外伸出、末端下垂,粒子沿枝条按 `s^0.78` 分布,枝尖散开并毛糙化;
  另有 30% 粒子吸附在 6 条黄金螺旋灯带上。枝条长短、下垂量、弯曲方向各不相同,外轮廓因此参差
- **渲染**:`THREE.Points` + 自定义 `ShaderMaterial`(软光斑、逐粒子闪烁、大粒子十字星芒、微漂浮、风摆、
  按视深压暗做体积)+ `UnrealBloomPass` 辉光
- **烟花**:3600 粒对象池。一发烟花是完整的三段过程——火箭从地面带火星尾升空 → 爆裂(闪光 + 冲击波环
  + 球面/圆环/双环三种形状)→ 每颗火花拖一条 5 点光链尾巴,颜色从白热到本色再到暗红余烬。
  每 3.4~6.4 秒一朵,按相机朝向投放到画面左右两侧;点击画面则朝点击位置打一发。
  发射"咻"和爆炸"砰"是 `OscillatorNode` + 带通噪声现场合成
- **爆散**:vertex shader 里 `mix(treePos, explodePos, uProgress)`,JS 端用缓动补间驱动 `uProgress`
- **手势**:`@mediapipe/tasks-vision` 的 `HandLandmarker`。张开度用**三维**"指尖到手腕 / 掌长"判定
  (摊开约 2.2 倍、握拳约 1.1 倍),只算 2D 投影的话手掌正对镜头时握拳和摊开几乎一样;
  再叠 EMA 平滑 + 2 帧防抖,左右手都适用
- **BGM**:`OscillatorNode` 正弦 + 二/三次谐波 + 指数衰减包络 + 程序生成混响卷积,lookahead 调度
- **依赖**:Three.js r187、MediaPipe Tasks Vision 0.10.21(均从 jsDelivr 加载,固定版本号)
