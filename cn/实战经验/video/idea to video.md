# idea to video

## 1.初始化阶段

1. 用户输入prompt(想法描述) + 参数（时常，尺寸，语言等）

2. Controller接受到请求 -> 生成requestId
3. 初始化loading 步骤缓存

## 2. 三个并行任务

### 2.1 生成场景脚本（最核心）

调用GPT/AI模型，根据用户的prompt生成分镜脚本。

将故事拆分多个场景(scenes)

每个场景都包含，文本内容、时常、情绪。

### 2,2 生成视频标签

使用AI根据prompt生成合适的视频标签。并且作为草稿昵称。

### 2.3 选择背景音乐

​	分析 prompt 的内容和情绪

​	自动匹配合适的背景音乐标签

## 3.场景处理和素材匹配

每个场景都会文本转语言（TTS）调用AI语言模型服务生成旁白

然后素材匹配，优先匹配用户上传的素材库，匹配度不足时使用默认素材库（Pexels等），支持图片/视频混合

然后根据语音生成时间轴对齐的字母

## 4. 构造

所有场景转换为一个RenderClip

~~~json
{
  "clipId": "场景ID",
  "duration": "片段时长",
  "media": "素材URL（图片或视频）",
  "audio": "语音URL",
  "subtitle": {
    "text": "字幕文本",
    "timing": "显示时间轴"
  },
  "transition": "转场效果",
  "animation": "动画效果"
}
~~~

## 5. 生产完整渲染配置(RederConfig)

~~~c#
RenderConfig {
  ├─ clips: [所有场景片段]
  ├─ backgroundMusic: {音乐配置}
  ├─ globalSettings: {
  │   ├─ size: "1080x1920"
  │   ├─ fps: 30
  │   └─ duration: 总时长
  │  }
  └─ effects: {全局效果}
}
~~~

## 6. 把json给视频渲染引擎生成

最终获得出来一个视频的url