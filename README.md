# 小红书视频解析流程

以下是解析小红书视频的具体步骤与所需的关键链接：

## 流程步骤

### 1. 输入链接解析
- 用户输入小红书链接，可能是以下类型：
  - 短链接，例如：`https://xhslink.com/...`
  - 普通链接，例如：`https://www.xiaohongshu.com/explore/...`
  - 分享链接，例如：`https://www.xiaohongshu.com/discovery/item/...`

88 诡秘之主发布了一篇小红书笔记，快来看吧！ 😆 Ebsjwey8PSgXlMl 😆 http://xhslink.com/a/K8Jc0b，复制本条信息，打开【小红书】App查看精彩内容！

，复制打开会跳转到如下链接：  
https://www.xiaohongshu.com/discovery/item/67c8332b000000000d014493?app_platform=android&ignoreEngage=true&

15 少女乐队的呐喊官方发布了一篇小红书笔记，快来看吧！ 😆 ae2HsBnGKGZhJAs 😆 http://xhslink.com/a/dWXWmYAab，复制本条信息，打开【小红书】App查看精彩内容！

https://www.xiaohongshu.com/discovery/item/67cb94b1000000002803ff8b?app_platform=android&ignoreEngage=true


17 东映动画发布了一篇小红书笔记，快来看吧！ 😆 2CPKUyWzQJkhOI0 😆 http://xhslink.com/a/ewBnezKQvGBab，复制本条信息，打开【小红书】App查看精彩内容！
网页源码中直接就有视频链接，

https://www.xiaohongshu.com/discovery/item/67f4cf74000000001d023988?app_platform=android&ignoreEngage=true&app_version=
查看exiftool可知

“全球 Web 图标
xiaozhongpai.com
https://www.xiaozhongpai.com
剪映APP导出的视频文件会携带识别符信息 - 小众派
/images/search?view=detailV2&ccid=ecPu0dtW&id=000109F2777AB0B3EFFC67E6668ACCFCEC920102&thid=OIP.ecPu0dtWYbHSEq_ejjva8QHaJ8&mediaurl=https://www.xiaozhongpai.com/wp-content/uploads/2020/08/jianying02.jpg&q=douyin_beauty_me&ck=E2140F64864B1A69CDA4D9468FDCED12&idpp=rc&idpview=singleimage&form=rc2idp
本文介绍了剪映APP导出的视频文件在属性备注里会添加douyin_beauty_me”

其中
html中：

```
<meta name="og:video" content="这里是有水印视频链接">
```
无水印原链接：
直接在html中的"originVideoKey":"无水印视频id"

```
r = requests.get(url)
key = re.findall(r'{\"originVideoKey\":\".*?\"}', r.text)


```

然后 http://sns-video-bd.xhscdn.com/{无水印视频id} 就是无水印视频链接

```

import requests
import re
import json

link = 'https://www.xiaohongshu.com/explore/65e2c4fb00000000030367bd'

def work(url: str) -> dict:
    r = requests.get(url)
    if r.status_code == 200:

        url_with_watermark = re.findall(r'<meta name="og:video" content="(.*?)">', r.text)
        if url_with_watermark:
            url_with_watermark = url_with_watermark[0]
        else:
            url_with_watermark = None
        
        key = re.findall(r'{\"originVideoKey\":\".*?\"}', r.text)
        if key:
            url_without_watermark = "http://sns-video-bd.xhscdn.com/" + json.loads(key[0])["originVideoKey"]

```
#### 操作

- 检测链接类型：
  - 若为短链接，使用短链接扩展工具将其转换为普通链接。
  - 如果是普通链接或分享链接，直接用正则表达式提取链接。

---

### 2. 提取笔记 ID
- 从解析后的链接中提取笔记 ID。

#### 操作
- 分析链接路径，提取路径中的最后一个非空部分作为笔记 ID。

---

### 3. 获取 HTML 页面内容
- 根据链接抓取对应的 HTML 页面。

#### 操作
- 使用 HTTP 请求工具（如 `HttpUtil.getHtml`）获取页面 HTML。

---

### 4. 提取页面中的 JavaScript 数据
- 从 HTML 页面中找到包含笔记详细信息的脚本。

#### 操作
- 使用正则表达式定位 `<script>window.__INITIAL_STATE__=...</script>`。
- 提取其中的 JSON 数据。

---

### 5. 解析笔记详细信息
- 将提取的 JSON 数据解析为结构化的笔记信息。

#### 操作
- 定位笔记信息：
  ```
  json['note']['noteDetailMap'][linkId]['note']
  ```
- 提取字段：
  - `noteId`: 笔记 ID
  - `title`: 笔记标题
  - `desc`: 笔记描述
  - `type`: 笔记类型（`normal` 表示图文，`video` 表示视频）

---

### 6. 提取视频信息
- 如果笔记类型为视频，提取视频资源。

#### 操作
1. 找到 `originVideoKey`：
   ```
   noteInfo['video']['consumer']['originVideoKey']
   ```
2. 构建视频 URL：
   ```
   https://sns-video-bd.xhscdn.com/{originVideoKey}
   ```
3. 获取视频时长：
   ```
   noteInfo['video']['capa']['duration']
   ```
4. 获取文件大小：
   - 使用 HTTP 请求工具，获取视频文件的大小。

---

## 结果
最终，您可以获得以下信息：
- 视频 URL: `https://sns-video-bd.xhscdn.com/{originVideoKey}`
- 视频时长
- 视频文件大小




参考：解析小红书无水印视频直链 | 回声https://iecho.cc/2024/03/03/decode-xiaohongshu-video-url/

https://github.com/nilaoda/xhs_app/blob/main/lib/service/file_downloader.dart


```
import re
import requests

class XiaohongshuVideoExtractor:
    def __init__(self):
        self.short_link_regex = re.compile(r'https?://xhslink\.com/\S+')
        self.script_regex = re.compile(r'<script>window.__INITIAL_STATE__=(.*?)</script>')

    def expand_short_url(self, url):
        return requests.head(url, allow_redirects=True).url

    def extract_video_url(self, input_url):
        # Expand short links
        if self.short_link_regex.match(input_url):
            input_url = self.expand_short_url(input_url)

        # Fetch HTML content
        html = requests.get(input_url).text

        # Extract JSON script
        initial_state = self.script_regex.search(html).group(1)
        json_data = eval(initial_state)  # Use a safer JSON parser in production

        # Extract video key and construct video URL
        video_key = json_data['note']['noteDetailMap'].popitem()[1]['note']['video']['consumer']['originVideoKey']
        return f"https://sns-video-bd.xhscdn.com/{video_key}"

# Example usage
extractor = XiaohongshuVideoExtractor()
video_url = extractor.extract_video_url("https://xhslink.com/example")
print("Video URL:", video_url)
```
