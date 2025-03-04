# 调参绘画

>此页面等待精修

>Todo 
>修复混乱的逻辑
>
>Upscaler 2 的预期用途 https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/3749
>


这节会介绍 参数 和 相关的 WebUi(SD)  网页应用资源。如果你会画画，那么效果会更加稳定可观。

标签参考应该使用 [Danbooru wiki Tag](https://danbooru.donmai.us/wiki_pages/tag_groups) 和 [MidJourney-Styles-and-Keywords-Reference](https://github.com/willwulfken/MidJourney-Styles-and-Keywords-Reference) ，因为它们都比较原生。 

如果需要中文的标签参考，请看[调参魔法书](https://docs.google.com/spreadsheets/d/e/2PACX-1vRa2HjzocajlsPLH1e5QsJumnEShfooDdeHqcAuxjPKBIVVTHbOYWASAQyfmrQhUtoZAKPri2s_tGxx/pubhtml)和 [手抄魔法本](https://docs.google.com/spreadsheets/d/14Gg1kIGWdZGXyCC8AgYVT0lqI6IivLzZOdIT3QMWwVI/)

如果你习惯阅读日语的标签参考和技巧，可以在 [日语Wiki](https://seesaawiki.jp/nai_ch/d/%be%ec%bd%ea%a1%a6%c7%d8%b7%ca) 中进行检索。

如果你觉得查表很麻烦，可以打开 [Danbooru 标签超市](https://tags.novelai.dev/),[项目地址](https://github.com/wfjsw/danbooru-diffusion-prompt-builder) 或者[AI绘画tag生成器](https://aitag.top/)(后者不支持负面Tag)

如果你不想读文字，可以打开 [推荐的视频教程](https://space.bilibili.com/35723238/channel/collectiondetail?sid=779851)，但需要了解 **通过 AI 模仿画风，特定镜头，增加特效，微修微调，PS嫁接出图，通过3D特定姿势，重画，迭代** 等等操作的话，需要**通读**下面的内容。


>提前告知：WebUi 的设置页面需要按下 `Apply setting` 才能保存设置。


## 基本流程

![WorkFlow](https://user-images.githubusercontent.com/75739606/197821744-99b18fbc-1ae4-4e37-b7e7-9a47adaa6ae5.svg)
<!--
![WorkFlow](https://raw.githubusercontent.com/sudoskys/StableDiffusionBook/main/resource/draw_workflow.svg)
-->
这幅图演示了循环迭代的流程
迭代方式，有循环迭代和线性迭代两种，线性迭代适用于多样性测试，而 **循环迭代** 是优化的更好选择。

来回改提示+固定种子并不是好选择。

目前研究基本方向是

- 提示词 + PS/Inpaint(微修/嫁接)

- 提示词 + 3D 参考


## 魔法入门

请先在前面了解一下 WebUi(SD) 网页应用的参数。


### 提示词语法简介

对于NAI用户，请阅读[官网Docs](https://docs.novelai.net/image/promptmixing.html)

!!! tip
    两方的提示词语法**不通用**
    用 `()`在WebUi做增强，而在NAi用 `{}` 做增强，注意NAI不支持单个词语指定权重

    如果你想便捷转换权重，可以使用[转换服务](https://t.me/M2NM2NBot)

**表示方法**

分隔，英文半角 `,` 做分隔符，在英文逗号前后存在一个或数个空格并不影响实际使用。

混合，WebUi 使用 `|` 分隔多个关键词以混合多个要素，字面意义上的混合，可以多个使用。

增强，用 `(提示词:权重)` 在 WebUi 权重增强，增强的范围是 `0.1 ~:100`，允许小数。而NAI使用 `{}`

!!! tip
    `a ((((farm))), daytime` 的语法可能会吃掉逗号

渐变，使用 `[some1:some2:num]`

`[fantasy:cyberpunk:16]` 代表从第 16 step 后，使用 `cyberpunk` 标签替换 `fantasy`

`[to:when]` 在固定数量的step后添加 `to` 到提示 ( when)

`[from::when]` 在固定数量的step后从提示中删除 `from`( when)

转义，WebUi 对于带括号提示词比如`a (word)`  请使用 `\` 字符转义为 `a \(word\)`，这适用于带括号的Tag,防止出现本来不想要的增强效果。

降低权重，`[]` 或者 `(word:0.952)`。NAI 仅能使用 `[]`

交替（alternate prompt）[^7]，混合机。这使您可以创建动物、人或风格的混合体，每一个 step 切换一项(一步一换)，轮回渲染，`[alison brie|emma stone|elizabeth olsen|scarlett johansson|anne hathaway|emma roberts], still film` 这是WebUi 语法，在 NAI 中代表平均权重混合(前半部分和后半部分)。

!!! tip "NAI"

    NAI中不允许单独指定权重，但支持混合权重 `cat:1|happy:-0.2|cute:-0:3` 这样的语法。
    
    因为NAI使用的是 WebUi 2022 年 9 月 29 日之前的实现，所以权重增强语法是旧的 `{}` ，新的 WebUi 更改为 `()`。
    
    **换算关系**
    NAI的 [word] = WebUi(word:0.952)(0.952 = 1/1.05)

    NAI的{word} = WeUi的(word:1.05)

    NAI花括号权重为1.05/个，WebUi圆权重为1.1/个
    


### 如何书写提示词(提示)

结合法术书编写它。

- 自然语言

可以直接使用自然语言，WebUi(SD) 有自然语言处理能力(英文句子),也可以使用 颜文字 和 emoji

- 参数[^6]

将你想要的相似的提示词组合在一起，并将这些按从最重要到最不重要的顺序排列。

可以使用 艺术家，工作室，摄影术语，角色名字，风格，特效等等。这些内容可以在 上面提到的法术书(示例集)查看，这些项目在关于中可以看到。

```
图片的质量
图片的主题
他们的外表
他们的情绪
他们的衣服
他们的姿势
图片的背景
```

[为文字转图像Ai提示编写指南：A Guide to Writing Prompts for Text-to-image AI](https://docs.google.com/document/d/1XUT2G9LmkZataHFzmuOtRXnuWBfhvXDAo8DkS--8tec/edit#)

**书写长度**

- 提示不要太长，超过 100 就有失败风险。

根据 手抄本的Tip，由于GPT-3模型限制，prompt 并不是无限的，positive token 在 75-80 之间，negative 大概65，加太多会提示你 xxx of xxx are truncated，所以别人那边的圣经不要照抄，太长的咒语后半都没有意义了，所以用简易反咒就足够，除非你有特定想屏蔽的东西。

当提示超过75个`token`（比如150个`token`）时，WebUi 将分组提示词，提交多组75个 `token`。标记只具有同一集合中其他内容的上下文。这意味着您可能在第一组和第二组之间的边界处有`bule hair`，标记`blue`将在第一组中，`hair`将在第二组中。这导致了结果的不准确，因为这两个词是分开的。

新版本增加了一个选项 `Increase coherency by padding from the last comma within n tokens when using more than 75 tokens`

这个设置让程序试图通过查找最后N个标记中是否有最后一个逗号来缓解这种情况，如果有，则将所有经过该逗号的内容一起移动到下一个集合中。

!!! tip "比如"

    有词条为 `...,Comma,blue hair,PADDING,...`

    第75个词为 `blue`

    使用该选项前

    集合1:{[74]=Comma，[75]=blue}，集合2:{[76]=hair, [77]=PADDING}

    使用该选项后

    集合1:{[074]=Comma，[75]=PADDING}，集合2:{[76]=blue，[77]=hair}

    如果您的提示小于等于75个标记，不会发生分组。

**书写格式**

为了方便书写改动，多行书写的建议格式如下

```
masterpiece,
1girl,  hatsune miku, 
look at viewer,turning back,
blow wind, cyan hair, two side up, cyan eyes,eardrop,dress,
caustics, masterpiece, high resolution,
```

第一行指定作品风格类型，第二行指定人物，第三行指定动作，第四行指定场景和着装，第五放指定其他参数

以上顺序已经比较合理，但是你也可以调整提示词的顺序以产生不同的结果。 你可以手动调整，也可以 [使用 X/Y 图自动生成各种顺序](https://github.com/AUTOMATIC1111/stable-diffusion-webui/pull/1607)

Ai 难以解析下划线，请少用。

**风格化**

Nai 出图默认是一种风格，你可以通过 训练风格模型，指定风格关键词等方法来创作带有特效或指定画风的图片。

[NovelAI使用教程和魔咒课堂](https://space.bilibili.com/8612008/channel/collectiondetail?sid=787691)

[人偶教室的测试记录](https://www.yuque.com/longyuye/lmgcwy)

[风格化: 32种](https://www.bilibili.com/video/BV1TP411N71t/)

更全面的指南在页尾。

### 调序编译

提示词放入的顺序就是优先级~

webui 突破 tag 75 个限制的方式是把 75 个分为一组。

出于提高可读性的目地，推荐主体往前放，接着描述装扮的词，画质提升词穿插在这些描述词之间，一般为了提高成品率要把动作、nsfw 词等改变构图的词往后放，或者手动调低权重。

以上排序是每组tag都要遵守的，所以如果后面的tag超过 75 了就应该把前面的分一部分过来。


### Batch count&batch size

`batch count` 指定训练几批次图像。

`batchsize` 中文翻译为批大小（批尺寸）

简单点说，批量大小将决定我们一次训练的样本数目。 batch_size将影响到模型的优化程度和速度。

还要注意 `batch size` 是为了在`内存效率`和`内存容量`之间寻找最佳平衡，因为单个 512x512 图像生成中，GPU 资源没有被充分利用。更大的图像从中获得的收益更少。

!!! info
    迭代是重复反馈的动作，神经网络中我们希望通过迭代进行多次的训练以到达所需的目标或结果。
    每一次迭代得到的结果都会被作为下一次迭代的初始值。
    一个迭代 = 一个正向通过+一个反向通过


### (提示词)影响因子[^6]
 
"调参魔法" 的一个基本技能是设置权重。

[Wiki:魔导师介绍](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Features#attentionemphasis)

具体规则如下

```
a (word) - 将权重提高 1.1 倍

a ((word)) - 将权重提高 1.21 倍（= 1.1 * 1.1），乘法的关系。

a [word] - 将权重减少 1.1 倍

a (word:1.5) - 将权重提高 1.5 倍

a (word:0.25) - 将权重减少 4 倍 (= 1 / 0.25)

a \(word\) - 在提示中使用文字 () 字符
```

必须使用 `()`指定，可以像这样指定权重：(text:1.4)

如果未指定权重，则假定为 `1.1`

指定权重仅适用于 `()` 而不是 `[]`，注意 `[]` 削减语法，指定单个权重仅适用于WebUi 且使用 `()`

```
> ( n ) = ( n : 1.1 )  
> (( n )) = ( n : 1.21 )  
> ((( n ))) = ( n : 1.331 )  
> (((( n )))) = ( n : 1.4641 )  
> ((((( n )))) = ( n : 1.61051 )  
> (((((( n )))))) = ( n : 1.771561 )  
```

`(cat :2 | dog)` 也就是更像猫的狗

在 WebUi 中需要使用 `()`指定权重！可以像这样指定权重：(text:1.4)。如果未指定权重，则假定为 1.1。



!!! info
    权重增加通常会占一个提示词位。在token紧张的情况下没有必要加特别多括号。
    
    过多圆括号会导致 字符 被程序吃掉，`a ((((farm))), daytime`会变成 `a farm daytime` 而没有逗号。
    
!!! tip "NAI"

    NAI中不允许单独指定权重，但支持混合权重 `cat:1|happy:-0.2|cute:-0:3` 这样的语法。
    
    因为 NAI 使用的是 WebUi 2022 年 9 月 29 日之前的实现，所以权重增强语法是旧的 `{}` ，新的 WebUi 更改为 `()`。
    
    **换算关系**
    NAI的 [word] = WebUi(word:0.952)(0.952 = 1/1.05)

    NAI的{word} = WeUi的(word:1.05)

    NAI花括号权重为1.05/个，WebUi圆权重为1.1/个

??? tip "How It Work"
    每个单词都有一个768个值的相关向量，该向量“指向”概念的方向（在768维空间中）。
    
    如果你缩放这个向量，这个概念会变得更强或更弱。

    From [Here](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/2905)

### 否定提示词

WebUi(SD)网页应用会在生成时**拒绝否定提示词有关的内容**。

否定提示是一种使用稳定扩散的方式，允许用户指定他不想看到的内容，而不对模型做额外的要求。

通过指定 `unconditional_conditioning` 参数，在生成中采样器会查看去噪后符合提示的图像（城堡） 和 去噪后看起来符合负面提示的图像（颗粒状、雾状）之间的差异，并尝试将最终结果远离否定提示词。

```python
# prompts = ["a castle in a forest"]
# negative_prompts = ["grainy, fog"]

c = model.get_learned_conditioning(prompts)
uc = model.get_learned_conditioning(negative_prompts)

samples_ddim, _ = sampler.sample(conditioning=c, unconditional_conditioning=uc, [...])
```


比如使用以下提示词拒绝水印和文字内容

```
lowres, bad anatomy, bad hands, text, error, missing fingers, 
extra digit, fewer digits, cropped, worst quality, low quality, 
normal quality, jpeg artifacts, signature, watermark, username, blurry
```

还如这个例子

```
ugly, fat, obese, chubby, (((deformed))), [blurry], bad anatomy, 
disfigured, poorly drawn face, mutation, mutated, (extra_limb), 
(ugly), (poorly drawn hands fingers), messy drawing, morbid, 
mutilated, tranny, trans, trannsexual, [out of frame], (bad proportions), 
(poorly drawn body), (poorly drawn legs), worst quality, low quality, 
normal quality, text, censored, gown, latex, pencil
```

[官方Wiki](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Negative-prompt#examples)

### 渐变标签

渐变标签，指示 WebUi 在训练中替换 Token，语法使用 `[some1:some2:num]`

`[fantasy:cyberpunk:16]` 代表从第 16 step 后，使用 `cyberpunk` 标签替换 `fantasy`

`[to:when]` 在固定数量的step后添加 `to` 到提示 ( when)

`[from::when]` 在固定数量的step后从提示中删除 `from`( when)

![sample_Gradient](https://user-images.githubusercontent.com/75739606/197822841-f7323afa-8c6a-46a2-a8e2-a1c457bb31d5.jpg)
<!--
![sample_Gradient](https://raw.githubusercontent.com/sudoskys/StableDiffusionBook/main/resource/sample_Gradient.jpg)
-->
### 重现提示词

对于没有压缩的原图，我们可以将文件拖入 `PNG Info` 选项卡，进行提示词(Token)查看。

或者使用 [网页工具](https://spell.novelai.dev/)


### 逆向提示词

这里有一些 **Image 逆向参数服务**，可以从图片中提取相关参数，不一定准确。

[LenKiMo_Bot](https://t.me/LenKiMo_Bot)

[DeepDanbooru](https://github.com/KichangKim/DeepDanbooru)


### 已知提示词(Token)组合 / 搜索引擎

你可以访问以下传送门获取一些优秀的参数实例！（当然，中文社区的测试群有**大量素材**，一分钟20次）

[StableDiffusion_Show](https://t.me/StableDiffusion_Show)

[cyan_ai_sese](https://t.me/cyan_ai_sese)

[LEXICA搜索引擎](https://lexica.art/?q=Miku)


### Prompt matrix 参数矩阵/要素混合

使用 | 分隔多个Tag，程序将为它们的每个组合生成一个图像。 例如，如果使用 `a busy city street in a modern city|illustration|cinematic lighting` ，则可能有四种组合（始终保留提示的第一部分）：

- a busy city street in a modern city
- a busy city street in a modern city, illustration
- a busy city street in a modern city, cinematic lighting
- a busy city street in a modern city, illustration, cinematic lighting

可以进一步在关键词后添加 :x 来指定单个关键词的权重，x 的取值范围是 0.1~100，默认为 1。

`cat :2 | dog` 也就是更像猫的狗


### 提示词原理

咒语的科学原理。

![prompt_draw](https://user-images.githubusercontent.com/75739606/198675128-c2c849d0-d024-468b-80c4-374f13e933e3.png)
<!--
![prompt_draw](https://raw.githubusercontent.com/sudoskys/StableDiffusionBook/main/resource/prompt_draw_fix.png)
-->
>By RcINS

在程序中，提示词的解析由 CLIP 处理

[WebUi的prompt_parser](https://github.com/AUTOMATIC1111/stable-diffusion-webui/blob/master/modules/prompt_parser.py) 实现了渐变等功能

CLIP 无法处理中文，中文字符会被分解。

**关于权重的实现**：权重增加通常会占一个提示词位。


**关于渐变的实现**：到了指定 Step ，WebUi 程序会替换对应 提示词，达到渐变效果。

其他以此类推。

WebUi prompt 语法会转换为相应时间的 prompt,然后通过 embedding 交给 Ai 处理。


### 参数冲突(提示词)

比如 `sex` 包含较多姿势体位，在使用者想要特定姿势时，法术内单一的 `sex` tag就应该被删除。

同样地，`loli` Tag 附带了强画风属性，会很大地影响结果！改成 `female child` 会好一点。



### Step 迭代步数

更多的迭代步数可能会有更好的生成效果，更多细节和锐化，但是会导致生成时间变长。而在实际应用中，30 步和 50 步之间的差异几乎无法区分。

太多的迭代步数也可能适得其反，几乎不会有提高。

进行图生图的时候，正常情况下更弱的去噪需要更少的迭代步数(这是工作原理决定的)。你可以在设置里更改设置，让程序确切执行滑块指定的迭代步数。

### Samplers 采样器

目前好用的有 `eular`，`eular a`，更细腻，和`Ddim`。

推荐 `eular a` 和 `Ddim`，**新手推荐使用 `eular a`**

`eular a` 富有创造力，不同步数可以生产出不同的图片。
PS：调太高步数(>30)效果不会更好

`DDIM` 收敛快，但效率相对较低，因为需要很多 step 才能获得好的结果，**适合在重绘时候使用**

`LMS`和`PLMS`是`eular`的衍生，它们使用一种相关但稍有不同的方法（平均过去的几个步骤以提高准确性）。大概 30 step 可以得到稳定结果

`PLMS`是一种有效的LMS（经典方法），可以更好地处理神经网络结构中的奇异性


`DPM2`是一种神奇的方法，它旨在改进DDIM，减少步骤以获得良好的结果。它需要每一步运行两次去噪，它的速度大约是 DDIM 的两倍。但是如果你在进行调试提示词的实验，这个采样器效果不怎么样

`Euler` 是最简单的，因此也是最快的之一

[英文Wiki介绍](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Features#attentionemphasis)

[英文论坛介绍](https://www.reddit.com/r/StableDiffusion/comments/xbeyw3/can_anyone_offer_a_little_guidance_on_the/)

- 举个例子


|预览一|预览二|
|--|--|
|<img src="https://user-images.githubusercontent.com/22421310/187063145-3d4f16d7-7bd6-4804-be1c-acf228ed2507.jpg" width="400" alt="效果">|<img src="https://user-images.githubusercontent.com/75739606/197824518-f68188a3-0572-4b52-8fe7-289b6d7b640b.jpg" width="400" alt="效果">|

>不同 step 和 采样器 的不同效果


### 种子调试

实际的种子整数并不重要。它只是初始化一个定义扩散起点的随机数生成器，是一个随机初始值。

一个好的种子可以在各种提示、采样器和CFG中执行成分和颜色等内容。但是它现在作用有限。

在相同 Step ，cfg ，Seed,参数（prompts） 的情况下，生产的图片基本相同！

在同一模型和后端实现中，保持所有参数一致的情况下，相同的种子会产生同样的图片。取决于其他参数。

但是注意，**不同显卡可能会造成预料之外的不同结果**（比如精度这样的东西）

10xx 系列看起来与其他所有卡如此不同,见[这里](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/2017#discussioncomment-3873467)

### 绘画插件

安装完毕重启程序。

#### 随机艺术家插件

[项目地址](https://github.com/yfszzx/stable-diffusion-webui-inspiration)

```bash
cd extensions
git clone https://github.com/yfszzx/stable-diffusion-webui-inspiration
```


#### 美学权重插件

[项目地址](https://github.com/AUTOMATIC1111/stable-diffusion-webui-aesthetic-gradients)

```bash
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui-aesthetic-gradients extensions/aesthetic-gradients
```


#### 历史记录画廊

[项目地址](https://github.com/yfszzx/stable-diffusion-webui-images-browser)

```bash
cd extensions
git clone https://github.com/yfszzx/stable-diffusion-webui-images-browser 
```


#### Wildcards 通配符

[项目地址](https://github.com/AUTOMATIC1111/stable-diffusion-webui-wildcards)

允许使用类似 `__name__` 提示的语法，以从名为 `name.txt` 的文件中获取随机一行。 


#### Deforum

[项目地址](https://github.com/deforum-art/deforum-for-automatic1111-webui)

Deforum 的官方API，一个用于 2D 和 3D 动画的扩展脚本，supporting keyframable sequences, dynamic math parameters (even inside the prompts), dynamic masking, depth estimation and warping.

#### Stable-Diffusion-Video2Video

将 视频 投入 Img2Img,输出带有关键帧的视频。

https://github.com/Leonm99/Stable-Diffusion-Video2Video

#### Img2img Video

https://github.com/memes-forever/Stable-diffusion-webui-video

#### Artists To Study

把 https://artiststostudy.pages.dev 添加到 WebUi

https://github.com/camenduru/stable-diffusion-webui-artists-to-study

#### 美学权重评分器

计算生成图像的美学分数

https://github.com/tsngo/stable-diffusion-webui-aesthetic-image-scorer


##  实战指南

这个简短的实战指南，可以让你快速了解如何合理调整参数达成目的效果。

### 场景表

竖着看

|人物|场景|数据限定|事件|
|----|------|-----|-----|
|表情| | | |
|头发|广狭选择|绘画类型|缩写词|
|眼睛|光影选择|评价限定|sfw/nsfw|
|衣着|背景主体|联想元素||
|状态|人物事件地|||
|姿势位||||
|镜头位||||


### 图转图的秘籍

无论是 3D(DAZ这样的3D模型) 还是线稿，AI只识别 **色彩** ，而不是线条，色彩直接决定图转图的效果。


### 尺寸选择

不应该将其与画质挂钩，尺寸一定程度上影响了主题，因为它潜在代表选择的类别(比如竖屏人物，横屏风景，小分辨率表情包居多)。

画质可以使用 超分指南 进行操作。

### 视角

[推荐使用 Danbooru含有的术语](https://danbooru.donmai.us/wiki_pages/tag_group%3Aimage_composition)

![shot](https://user-images.githubusercontent.com/75739606/198682933-25c1b2df-c573-44ea-aba7-2df4609299e5.png)
<!--
![shot](https://raw.githubusercontent.com/sudoskys/StableDiffusionBook/main/resource/shot.png)
-->
>引用来自日本 Wiki 的图片，作者不明

### 跨领域术语/奇门遁甲

跨领域术语的核心逻辑就是**缩小指定的数据范围**，从画面内容之外的**平台，领域，事件**上入手来提升效果。

是的！你可以在提示中使用 [Film Glossary](https://www.owlnet.rice.edu/~engl377/film.html) [FILM GLOSSARY摄影术语](http://userhome.brooklyn.cuny.edu/anthro/jbeatty/COURSES/glossary.htm)， [Cinematic techniques摄影技术](https://en.wikipedia.org/wiki/Category:Cinematic_techniques)，以及绘画术语(类型) 来控制基本情况。

跨领域！你甚至可以使用各种惊险运动的名词来生产一些特效....比如空降

比如，景深，光圈，构图，拍摄机位，运动元素，[艺术摄影术语表中文介绍](https://gallerix.asia/pedia/photography-glossary/)

**但是**这种效果可能会带来附加作用:引入你不希望见到的风格(如实景**而不是**而二次元)数据进入图片。把握好量度。适当增加 Step 和 风格提示 来改善.

你还可以使用**平台名**来限定数据集的范围，比如 pixiv 之类的词汇。

**扩展阅读**

[推荐使用 Danbooru含有的术语](https://danbooru.donmai.us/wiki_pages/tag_group%3Aimage_composition)

[Danbooru tag组](https://danbooru.donmai.us/wiki_pages/tag_groups)

有用的电影术语 https://en.wikipedia.org/wiki/Category:Cinematic_techniques

镜头类型 https://www.bhphotovideo.com/explora/video/tips-and-solutions/filmmaking-101-camera-shot-types

电视术语  https://en.wikipedia.org/wiki/Category:Television_terminology

摄影类型 https://en.wikipedia.org/wiki/Category:Photography_by_genre

摄影术语 https://zh.wikipedia.org/zh-cn/%E9%AB%98%E9%80%9F%E6%91%84%E5%BD%B1

极限运动 https://en.wikipedia.org/wiki/Extreme_sport https://en.wikipedia.org/wiki/Category:Sports_by_type

构图艺术 https://en.wikipedia.org/wiki/Composition_(visual_arts)

### 迭代草图[^8]

这里讨论一下如何将**手绘草图**通过 Ai 绘画优化，*注意不是二次元*。

在第一次迭代中，您不需要太多 Steps，CFG 可以非常低（以获得更好的多样化结果），如果不想完全丢失草图，Denoising 应该在 0.3-0.4 左右。

在最后的迭代中，增加 Steps 和 Denoising 强度（但不超过 0.8，否则图像将被破坏，尤其是在大于 512*512 时）请参见[这里](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/2213#issuecomment-1274137775)，同时根据需要提高 CFG 和尺寸。

你可以随时修复提示（添加或删除出现的细节）并尝试不同的采样器。

另外，你不应该在初次生成使用一个固定不变的种子？

如果你提供一个种子（而不是随机的 -1），你的图像很快就会变得过饱和、过度锐化、过度像素化..... 当然如果想微调，可以使用固定种子。

### 表情控制

我们可以使用 颜文字 来控制出图的表情！

:-) 微笑 :-( 不悦 ;-) 使眼色 :-D 开心 :-P 吐舌头 :-C 很悲伤 :-O 惊讶 张大口 :-/ 怀疑

仅支持西方颜文字，详细内容请见 [Danbooru颜文字部分](https://danbooru.donmai.us/wiki_pages/tag_group%3Aface_tags) 或 [维基百科](https://zh.wikipedia.org/wiki/%E8%A1%A8%E6%83%85%E7%AC%A6%E8%99%9F%E5%88%97%E8%A1%A8?oldformat=true)


### 多人物/单人物

打草图+IMG2IMG,这就秘诀～

宽幅画作单人物生成最好打草图，进行色彩涂抹，确定画面主体。

多人物确定人物数量，最好使用草稿/有色3d 排列 + 图生图。

人数超过三个就难以控制效果，人数大于6的图像模型里估计没有...

### 进行手掌修复

将图片送入 inpaint，使用大致相同的提示词，将关于 `手` 的提示放在前面，根据你希望它变动多少来设置降噪（如果只是希望手更完整，调至0.25以下)，然后保留步骤和 CFG 与 txt2img gen 相同。

或者仅遮住手部，以全分辨率修复，大大降低填充（它使用周围的像素来创建上下文，但只是在重新制作手部）并仅提示手部问题（详细的手部描写等）

CFG 越高，越符合提示词，降噪越高越偏离原图。


#### 同人物&差分

需要用到进阶的 Img2Img 相关内容，最好的方法是准备一个带有色彩的 3D 母本模型，然后这样就可以保证基本一致。

也可以用很多提示词来限制角色内容，出很多张，挑能用的作品。

如果是表情或者是背景，可以采用进阶教程中的 重绘画 技巧。

如果你想了解一些差分的实例，[5CH日语Wiki](https://seesaawiki.jp/nai_ch/d/%c7%ed%a4%ae%a5%b3%a5%e9%a5%c6%a5%af) 提供了一个实例。


### 使用 Ai 进行立绘设计

<iframe src="//player.bilibili.com/player.html?aid=559362671&bvid=BV14e4y1U7r9&cid=869144379&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width="100%" height="600"> </iframe>
>BV14e4y1U7r9


## 魔法进阶

调参基本原理模糊的说是：限定好的数据范围内相似样本越多，越稳定。

!!! tip
    请阅读前面章节的模型进阶1,了解具体的 Img2Img 和 inpaint 介绍操作。


### Img2Txt

生成按钮下有一个 `Interrogate CLIP`，点击后会下载 `CLIP`，用于生成当前图片框内图片的 Tag 并填充到提示词。

CLIP 询问器有两个部分：一个是 BLIP 模型，它从图片中创建文本描述。另一种是 CLIP 模型，它会从列表中挑选出与图片相关的几行

!!! tip 
    本文件为 [model_base_caption_capfilt_large.pth](https://storage.googleapis.com/sfr-vision-language-research/BLIP/models/model_base_caption_capfilt_large.pth)
    
    大小为855MB


### Img2Img 介绍

一般我们有两种途径对图像进行修复：**PS 和 InPaint**，使用方法也十分多样。

WebUi 使用 `--gradio-img2img-tool color-sketch` 启动会带入一个插件对图片进行颜色涂抹(这里不是 Inpaint)

!!! tip "不同之处"
    PS 重新绘画投入 Img2Img 的话，会导致画风的变动，而 Inpaint 就不会。


#### 横条参数

**CFG Scale**

`cfg scale` 就是符合 prompt 的程度,Scale越高，程序对提示词越忠诚，越符合。

**Denoising strength 降低噪声**

`Denoising strength` 决定算法对图像内容的保留程度,可以减少对画风的变得，但也会弱化img2img能力。值越高 AI 对原图的参考程度就越低 (同时增加迭代次数)。

对于以图做图来说，低 `denoising` 意味着修正原图，高 `denoising` 就和原图就没有大的相关性了。一般来讲阈值是 0.7 左右，超过 0.7 和原图基本上无关，0.3 以下就是稍微改一些。


#### 图形参数

Just resize : 将图像调整为目标分辨率。除非高度和宽度完全匹配，否则图片会被挤压

Crop and resize：调整图像大小，使整个目标分辨率都被图像填充。裁剪多余部分。

Resize and fill：调整图像大小，使整个图像在目标分辨率内。用图像的颜色填充空白区域。


###  Img2Img 三渲二

调整3D模型骨架比寻找样图更容易。

可以结合 **3D建模** 摆 Pose,可以使用 MMD 相关软件。

如果是真人图片，需要适当提高 `CFG Scale` 相似度，结合提示词一起生成。降噪 `denoising` 越高，相关性越低。

推荐使用 [DAZ](https://www.daz3d.com/get_studio) 或者 [blender](https://www.blender.org/) 或者 Unity ，在对 3D 模型的测试中，**色彩主要影响 AI 的绘画效果**，所以你的模型需要有纹理。

如果你使用 blender ，你可以使用 [这个视频](https://youtu.be/MClbPwu-75o) 分享的 [模型娃娃](https://www.artstation.com/marketplace/p/VOAyv/stable-diffusion-3d-posable-manekin-doll?utm_source=artstation&utm_medium=referral&utm_campaign=homepage&utm_term=marketplace)


### Img2Img PS重绘画/嫁接修复/躺姿补全

**尺寸上应该尽量靠近原图尺寸，然后选择 `Crop and resize` 也就是裁切后调整大小**

使用PS软件增删元素，然后重新生产。这可以解决画手的问题。

Ai 也接受其他成图进行嫁接(解决躺姿没有下半身的问题)

比如我们可以通过 PS 给角色移植一个手让Ai来润色它，或者为没有下半身的半身像嫁接其他作品的下半身让 AI 润色它。

或者涂鸦特定部位指定形状动作(比如衣料的覆盖率或者形状)，打上色块（为了防止出现阴影，**请取亮色皮肤**）。

![test_woman](https://user-images.githubusercontent.com/75739606/197823480-5de77d69-46d5-4817-948f-4e514e1f8204.jpg)
<!--
![info](https://raw.githubusercontent.com/sudoskys/StableDiffusionBook/main/resource/00119_136826557_masterpiece%2C_best_quality%2C_1girl%2C_black_hair%2C_hat1.jpg)
-->
>一张图片[^5]展现WebUI下img2img中不同参数下效果的详细对比图（prompt、steps、scale、各种seed等参数均保持一致）

纵轴是Denoising strength（线上版的strength），横轴是Variation strength



#### 重绘画技巧/去除/替换

- 首先要对人物描线，然后打上色块（如果有阴影，取**亮色皮肤**）。

- 变动强度，选择较低的 0.3 左右的去噪。

- 然后使用 Img2Img Inpaint + 相关提示词修复，不满意可以再改，直到满意。

- 然后对图像进行超分，降噪(去除图像纹理)。


### Img2Img **Outpainting 外部修补**

Outpainting 扩展原始图像并修复创建的空白空间。
您可以在底部的 img2img 选项卡中找到该功能，在 Script -> Poor man's outpainting 下。

```
Outpainting, unlike normal image generation, seems to profit very much from large step count. A recipe for a good outpainting is a good prompt that matches the picture, sliders for denoising and CFG scale set to max, and step count of 50 to 100 with Euler ancestral or DPM2 ancestral samplers.
```

### Img2Img **Inpainting 修补**

在 img2img 选项卡中，在图像的一部分上绘制蒙版，该部分将被重画。

对于 `Masked content` 设置，遮罩内容字段确定内容在修复之前放置到遮罩区域中。

一般选 `original`，可以保持潛空间一致性。

它们的效果如下:

| mask  | fill  | original   | latent noise      | latent nothing       |
|---------------------------|----------------|-----------------------|-------------------------|-----------------------|
| ![](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/images/inpainting-initial-content-mask.png) | ![](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/images/inpainting-initial-content-fill.png) | ![](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/images/inpainting-initial-content-original.png) | ![](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/images/inpainting-initial-content-latent-noise.png) | ![](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/images/inpainting-initial-content-latent-nothing.png) |

`mask` 横条决定了模糊程度，original 是`原图`，fill 是`填充底色`

Tip: `fill` 要更多 step 才能消除不自然感.

`Inpaint at full resolution` 即全分辨率修复。
通常，Inpaint 会将图像大小调整为*UI中指定的目标分辨率*。启用完全分辨率的Inpaint后，仅调整遮罩区域的大小，并在处理后将其**粘贴回**原始图片。这允许您处理大图片，并允许您以更大的分辨率渲染修复对象。

有几种方法进行重绘制:

- 在网络编辑器中自己绘制蒙版（`Inpaint masked `指重画涂鸦区域，`Inpaint not masked` 指重画涂鸦之外的区域）

- 在外部编辑器中擦除部分图片并上传透明图片。 任何稍微透明的区域都将成为蒙版的一部分。 请注意，某些编辑器默认将完全透明的区域保存为黑色。

- 将模式（图片右下角）更改为"Upload mask"并为蒙版选择单独的黑白图像(white=inpaint)。

如果 `inpaint at full resolution` 出现黑块，可能是RAM不足，尝试卸载 vae.

![result](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/images/inpainting.png)

[开源调研-AI绘画全参数讲解-002img2img图像到图像](https://www.bilibili.com/video/BV1HK411Q7uk/?zw)
<!--
<iframe src="//player.bilibili.com/player.html?aid=474043788&bvid=BV1HK411Q7uk&cid=860273094&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width="100%" height="600"> </iframe>
-->

通过这种方法，我们可以更改角色衣物风格或者其他任何细节。

[如何教会Ai画手](https://www.bilibili.com/video/av559044202/?zw)
<!--
<iframe src="//player.bilibili.com/player.html?aid=559044202&cid=859852841&page=1&danmaku=0" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width="100%" height="600"> </iframe>
-->

### Img2Img Loopback 回环生成

在 img2img 中设置loopback脚本，它允许自动将输出图像作为下一批的Batch提供，相当于保存输出图像，并用它替换输入图像。

Batch 数设置控制获得多少次迭代

通常，在执行此操作时，您会自己为下一次迭代选择许多图像中的一个，因此此功能的有用性可能值得怀疑，但反正我已经设法获得了一些我无法获得的非常好的输出。


### Img2Img 让低显存生成大分辨率图片

在前面提到了，如果遇到生成鬼图或者 低显存生产高分辨率图片 可以采用的 Img2Img 画质提升脚本。

其实我**强烈推荐**你使用 Extras 的功能对低分辨率进行重放，效果不错的，且体验良好！


#### 脚本

但是如果你想使用脚本提供的分辨率增强，这里有 Img2Img 的具体流程

1. 使用 `--medvram` 或者 `--lowvram` 参数启动webui

2. 选择较小分辨率生成图片。记住你生成图片的分辨率。生成完毕之后，复制图片的 `Seed`

3. 生成完毕后，先查看图片效果是否满意。如果满意，直接将图片送进Img2img。（点击 `Send to img2img`）

4. 在img2img界面底部，有一个 `Script` 选项。将 `Script` 选为 `SD Upscale`，里面的 Tile overlap 尽量调小

5. 一般送入 Img2img 的图，输入框自动填充原提示词。如果你发现prompt有变动，请手动填充

6. 选择合适的 `Sampling Steps` 和 `Sampling method`

7. 确认你的 `Width` 和 `Height` 与**原图**一致

8. 将第 2 步复制的 Seed 填入img2img的 Seed 里并生成

这里的 Width 和 Height 是超分时 img2img 的图片大小，如果不等会导致出现重叠问题

SD Upscale 选项在 Img2Img 的 Script 栏目中，主要作用是提升分辨率。


[脚本解决方案来源于此](https://gist.github.com/crosstyan/f912612f4c26e298feec4a2924c41d99#%E9%AB%98%E5%88%86%E8%BE%A8%E7%8E%87%E4%B8%8B%E5%87%BA%E6%80%AA%E5%9B%BE)

#### 超分图像 extras

`webui` extras 页有一个自带的超分功能，可以使用 `ESRGAN_4x`模型

当然 `realesrgan` 或者 `realcugan` 也可以。

!!! tip "相关模型"

    文件统一下载到 `SDwebUI文件夹\models` 下

    [LDSR](https://heibox.uni-heidelberg.de/f/578df07c8fc04ffbadf3/?dl=1)，文件大小为1.9GB

    [BSGRAN 4x](https://github.com/cszn/KAIR/releases/download/v1.0/BSRGAN.pth) ，文件大小为63.9M

    [ESRGAN_4x](https://github.com/cszn/KAIR/releases/download/v1.0/ESRGAN.pth)，文件大小为63.8MB

    [ScuNET GAN/PSNR](https://github.com/cszn/KAIR/releases/download/v1.0/scunet_color_real_gan.pth" to D:\stable-diffusio\models\ScuNET\ScuNET.pth)，文件大小为68.6MB

    [SwinIR 4x](https://github.com/JingyunLiang/SwinIR/releases/download/v0.0/003_realSR_BSRGAN_DFOWMFC_s64w8_SwinIR-L_x4_GAN.pth)，文件大小为136MB

**Highres Fix/超分应该使用什么 Upscaler？**

`SD Upscaler` 在注重细节的同时还提升分辨率。

曾经有段时间，`LSDR` 被认为是最好的。有些人喜欢 swinir，有些喜欢`esrgan4x`，`ymmv`，推荐使用 `ESRGAN_4x`

如果你要搞二次元，推荐使用[realcugan](https://github.com/bilibili/ailab/tree/main/Real-CUGAN)


### 图像去噪

推荐使用 [Real-ESRGAN](https://github.com/xinntao/Real-ESRGAN) 降噪。

![效果](https://raw.githubusercontent.com/xinntao/Real-ESRGAN/master/assets/teaser.jpg)
>效果图




### **渐变提示词**

https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Features#prompt-editing

允许您开始对一张图片进行采样，但在中间切换到其他图片。基本语法是：

```
[from:to:when]
```

其中from和to是任意文本，并且when是一个数字，用于定义应在采样周期多长时间内进行切换。越晚，模型绘制to文本代替from文本的能力就越小。如果when是介于 0 和 1 之间的数字，则它是进行切换之后的步数的一小部分。如果它是一个大于零的整数，那么这只是进行切换的步骤。

将一个提示编辑嵌套在另一个提示中不起作用。

**使用方法**

[to:when] 在固定数量的step后添加to到提示 ( when)

[from::when] 在固定数量的step后从提示中删除from( when)

例子： a [fantasy:cyberpunk:16] landscape

开始时，模型将绘制a fantasy landscape。

在第 16 步之后，它将切换到绘图a `cyberpunk landscape`，从幻想停止的地方继续。

比如 [male:female:0.0], 意味着你开始时就要求画一个女性。

### 扩展模型/DLC

见下一节的详细内容。

### 注意 `尺寸`

比如出图尺寸宽了人可能会多

!!! tip
    要匹配好姿势，镜头和人物才不畸形，有时候需要限定量词，多人物时要处理空间关系和 prompt 遮挡优先级。人数->人物样貌->环境样式->人物状态
    


### 提示词速查

**[手抄本法术书](https://docs.google.com/spreadsheets/d/14Gg1kIGWdZGXyCC8AgYVT0lqI6IivLzZOdIT3QMWwVI/edit)**

**[Danbooru全部Tag列表](https://gelbooru.com/index.php?page=tags&s=list)**

**[参数法术全典](https://docs.google.com/spreadsheets/d/e/2PACX-1vRa2HjzocajlsPLH1e5QsJumnEShfooDdeHqcAuxjPKBIVVTHbOYWASAQyfmrQhUtoZAKPri2s_tGxx/pubhtml#)**

**[Tag在线协作](https://docs.google.com/spreadsheets/d/1zij5OzCZIaQuhAbiSayhFznjgJ3rwbaNwnUnaUMxyTQ/edit?usp=drivesdk)**

**[NSFWTag](https://github.com/scooderic/exhentai-tags-chinese-translation)**

**[Ai艺术家文档](https://docs.google.com/spreadsheets/d/1SRqJ7F_6yHVSOeCi3U82aA448TqEGrUlRrLLZ51abLg/htmlview#)**

**[Novelai 关键词组合器](https://www.bilibili.com/read/cv19023021)**


### 调参工程师

[https://github.com/Maks-s/sd-akashic](https://github.com/Maks-s/sd-akashic)

[https://github.com/willwulfken/MidJourney-Styles-and-Keywords-Reference](https://github.com/willwulfken/MidJourney-Styles-and-Keywords-Reference)


## 市场应用情况调查

这里是稳定扩散（非 NAI 模型）的应用情况。

- 3D

在 blender 上，Ai 有 [渲染插件](https://blendermarket.com/products/ai-render/?ref=110)

- 设计

[Microsoft 365 工具套件](https://www.xda-developers.com/microsoft-designer-image-creator-ai-dall-e-2/)

[为 Age of Empires 3 Definitive edition 的游戏模组生成肖像](https://github.com/matrix4767)

[产品和架构设计/素描](https://github.com/horribleCodes)

- 专辑图

[用稳定扩散生成歌手的图像放入视频](https://github.com/Chilluminati91)

- 配图

[制作怪图作为记忆工具](https://github.com/ClashSAN)

[稳定扩散的真实用例](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/3219)

[稳定扩散的应用结合](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/3572)

- 自媒体

哔哩哔哩已经一大堆了，抖音也不少，可以做的话题很多

- 套壳

无良某些公司套壳开源项目，比如 某某画廊，某某版图。

(题外：韭菜采收容易，可用于园艺，在世界广泛种植)

- 插图/背景

小说插图，AI画背景(据说原版模型也很好用)

- NFT

？

### 谈论

Ai 处理不了细节，对漫画创作(包含其他需要连续情节的作品)其实很鸡肋，因为 Ai 无法保证是同一角色。

同时 Ai 会添加莫名其妙的结构光影，造成你二修困难。

这些特点导致 Ai 根本不能替代人工的精细需求，只能替代低端需求(细节没Ai多的那种)

有点实力的人都不用担心，Ai 的限制就在那里。

所以主要应该在于 背景(星轨)，给甲方参考图。

>他的未来中作用应该是完成预制框架中的绝大部分重复性作业


## 自定义脚本

此主题的内容可能不会即时更新到此处，[源地址](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Custom-Scripts#prompt-interpolation)

### 自动补全

这里有一个项目，可以使SD做到类似 NAI 的补全效果。

https://github.com/DominikDoom/a1111-sd-webui-tagcomplete

### Advanced prompt matrix

https://github.com/GRMrGecko/stable-diffusion-webui-automatic/blob/advanced_matrix/scripts/advanced_prompt_matrix.py

安装后可以使用以下语法
```
<cyber|cyborg|> cat <photo|image|artistic photo|oil painting> in a <car|boat|cyber city>
```

### XYZ 绘图脚本 

生成一个 .html 文件以交互浏览图像集。 使用滚轮或箭头键在 Z 维度中移动。

https://github.com/xrpgame/xyz_plot_script

### Embedding to PNG

将一个已经存在的 embeddings 转换为图片。

https://github.com/dfaker/embedding-to-png-script

### Random Steps and CFG

https://github.com/lilly1987/AI-WEBUI-scripts-Random


## 参数

为什么不去 [这里](https://danbooru.donmai.us/wiki_pages/howto%3Atag_checklist) 看[原始数据站点](https://danbooru.donmai.us/wiki_pages/tag_groups)的参数呢？

[E 站标签翻译项目](https://github.com/EhTagTranslation)

### NAI 在使用的出图参数

- 使用全量模型(官方的GPU云特别强悍)

- CLIP layer = 2

- 使用 ema 权重加载，将yaml 配置其中的 `use_ema` 设置为 true

- 将 ` sigma noise/strength` 重置为默认值 1

- 设定 `eta noise seed delta` 为 31337（使 ` sigma noise/strength` 无需使用 0.69 / 0.67）

- 如果 prompt 有权重，转换权重（ WebUi 占比 1.1 ，NAI 占比 1.05）

- 使用 `--no-half` 参数启动程序（次要）


**NAI 默认的模型设置**

```
steps": 28, "sampler": "[sampler]", "seed": [seed], "strength": 0.69, "noise": 0.667, "scale": 11.0,

Strength ， noise 是 eta 和 sigma

scale 就是 CFG scale
```

**NAI 默认的 `SFW` 消极提示词为**
```
lowres, bad anatomy, bad hands, text, error, missing fingers, extra digit, fewer digits, cropped, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, username, blurry
```


**其他**

将所有提示词前面加入 `masterpiece, best quality`


Clip 跳过 0，其他一切都很好（afaik 不要使用超网络、v2、yaml、VAE）


### 转换——NAI和WebUi(SD)的增强语法不同

**Prompts 参数括号转换**

在 NAI 和 Webui 之间转换加强参数，相关的机器人服务 [M2NM2NBot](https://t.me/M2NM2NBot)

相关的 [网页JS](https://github.com/naisd5ch/novel-ai-5ch-wiki-js)

权重增强标识：NAI 是 `{}` ，WebUi(SD) 是 `()`


### 关于元素法典

元素法典提供了一个即查即用的模板库(类比作文大全)，里面有Tag的调试记录，可以作为灵感参考。

[元素法典一卷](https://docs.qq.com/doc/DWHl3am5Zb05QbGVs)

[元素法典第一点五卷](https://docs.qq.com/doc/DWGh4QnZBVlJYRkly)


### 良好参数(风格趋向插画)[^4]

```
{an extremely delicate and beautiful}
```

[动漫人物/艺术家/风格化列表/Pt文件](https://rentry.org/anime_and_titties)

[风格化：人偶教室](https://www.yuque.com/longyuye/lmgcwy)

[风格化，日语Wiki](https://seesaawiki.jp/nai_ch/d/%b2%e8%c9%f7%a1%a6%b9%bd%bf%de)

[风格化: 32种](https://www.bilibili.com/video/BV1TP411N71t/)

[艺术家列表/SD1.4](https://rentry.org/artists_sd-v1-4)

[艺术家列表/SD1.4/1,833 位艺术家](https://www.urania.ai/top-sd-artists)

[艺术家博物馆](https://gallerix.asia/storeroom/)

#### 草图风格

| 词                                                            | 描述                                               |
|---------------------------------------------------------------|----------------------------------------------------|
| sketch                                                        | 可以让图片看起来像随手画的草稿                     |
| {{lineart}}                                                   | 可以让线条变得很粗                                 |
| {{{posing sketch}}}, {{monochrome}}                           | 黑白草图                                           |
| {rough sketch}                                                | 上了颜色的草图                                     |
| monochrome+lineart                                            | 情况下一般只会让眼睛上色，强调发色后头发也可以上色 |
| {{{monochrome}}}, {{{gray scale}}}, {{{pencil sketch lines}}} | 做出的铅笔速写的感觉                               |


利用sketch，pastel color，lineart的tag模拟一张图的绘画过程


#### 艺术风格

| 词                                                                  | 描述                                   |
|---------------------------------------------------------------------|----------------------------------------|
| chibi                                                               | 可以画出低头身比的效果(二头身, 三头身) |
| {{watercolor pencil}}                                               | 可以生成彩铅画                         |
| {{faux traditional media}}                                          | 可以做出签绘的风格                     |
| anime screeshot，                                                   | 可以让画面变成动画风格                 |
| {{{retro artstyle}}}                                                | 赛璐璐风                               |
| {photorealistic}, {painting}, {realistic}, {sketch}, {oil painting} | 厚涂                                   |
| pastel color和sketch                                                | 搭配会有速涂的质感                     |


#### 杂志/设定集 风格

| 词                                                             | 描述                                                           |
|----------------------------------------------------------------|----------------------------------------------------------------|
| official art                                                   | 变得更加官方一点                                               |
| three views from front, back and side和costume setup materials | 可以用来生成设定图                                             |
| multiple views                                                 | 会出现类似设定图                                               |
| {character sheet}                                              | 会出现设定图                                                   |
| magazine cover                                                 | 会把背景换成杂志封面, 配合office art更像真实杂志(虽然字没法看) |
| magazine scan                                                  | 类似杂志内页的风格                                             |
| posing                                                         | 会强调有一个动作, 不至于出现混乱的动作(露出有六个手指头的手)   |
| caustics                                                       | 画面向主题聚焦, 类似海报                                       |


### 常用参数:SFW

| 人物数量 | 描述                                                                  |
| ----------- | ----------------------------------------------------------------------- |
| 数量      | , one boy , one girl , two boy ,two girl,one_boy_one_girl(这是错误的) |


| 人物画风                                         | 描述 |
| -------------------------------------------------- | ------ |
| 质量提升参数 |   , masterpiece, best quality   |
| 原神                             |   , Genshin Impact   |
| 萝莉                  |    , female child , loli画风差  |

| 人物样貌                           | 描述                                                           |
| ------------------------------------ | ---------------------------------------------------------------- |
| 头发                               | hair                                                           |
| 长发                               | longhair                                                       |
| 短发                               | shorthair                                                      |
| 眼睛                               | eyes                                                           |
| 渐变颜色长发                       | gradient pink longhair                                         |
| 渐变颜色眼睛                       | gradient pink eyes                                             |
| 粗眉毛                             | thick eyebrows                                                 |
| 猫尾巴                             | cat tail                                                       |
| 猫耳朵                             | cat ears                                                       |
| 动物耳朵                           | animal ears                                                    |
| 毛茸茸的动物耳朵                   | animal ear fluff                                               |
| 刘海                               | bangs                                                          |
| 两眼之间的头发                     | hair between eyes                                              |
| 眉毛后面的头发                     | eyebrows behind hair                                           |
| 锁骨                               | collarbone                                                     |
| 斗篷(要在很前面才有效)             | cape                                                           |
| 乳房尺寸                           | small breasts                                                  |
| 出汗                               | sweating                                                       |
| 颜色丝袜(和长丝袜冲突)             | white stockings , black stockings                              |
| 长丝袜                             | thighhighs                                                     |
| 女仆                               | maid                                                           |
| 发带                               | ribbon                                                         |
| 爱心眼                             | heart-shaped pupils                                            |
| 御姐/JK/辣妹?                      | gyaru                                                          |
| 肌肉发达                           | muscular                                                       |
| 天使翅膀(要是形容人的第一个才正常) | angel wings                                                    |
| 颜色内裤(赠内衣)                   | pink underpants                                                |
| 肚脐                               | navel                                                          |
| 颈部颜色项圈                       | white collar                                                   |
| 黑色皮肤                           | dark skin                                                      |
| 撕裂的衣服                         | torn clothes                                                   |
| 撕裂的裤子                         | torn legwear                                                   |
| 开襟夹克(配合叉开腿特色)           | open jacket                                                    |
| 异色瞳                             | heterochromia_blue_red                                         |
| 吊袜带(会和内衣冲突)               | garter straps                                                  |
| 靴子                               | boots                                                          |
| 眼罩                               | blindfold                                                      |
| 流泪                               | tears                                                          |
| 项链                               | necklace                                                       |
| 眼镜                               | glasses                                                        |
| 比基尼                             | bikini                                                         |
| 湿衣服                             | wet clothes                                                    |
| 透明衣物                           | transparent raincoat , transparent jacket , transparent tshirt |
| 唾液(自动伸舌头)                   | saliva                                                         |
| 流口水(和唾液冲突)                 | drooling                                                       |
| 水手服                             | sailor dress                                                   |

| 环境样式                                                                 | 描述                           |
| -------------------------------------------------------------------------- | -------------------------------- |
| 在床上                                                                   | on bed                         |
| 光线反射                                                                 | reflection light               |
| 赛博朋克                                                                 | cyberpunk, city, kowloon, rain |
| 在地毯上                                                                 | on carpet                      |
| 在瑜伽垫上(它分不清什么是瑜伽垫，只知道色块比较大，所以要配合one girl用) | on_yoga_mats                   |

| 人物视角     | 描述        |
| -------------- | ------------- |
| 正面视角     | from viewer |
| 从上到下视角 | from below  |
| 全身         | full body   |

| 人物状态           | 描述                                                 |
| -------------------- | ------------------------------------------------------ |
| 叉开腿             | spread leg                                           |
| 露出腋下           | armpits                                              |
| 举起手             | hands up , arms up                                   |
| 爪子手             | paw pose                                             |
| 站立               | standing                                             |
| 行走               | walking                                              |
| 吐舌头             | tongue out                                           |
| 抬起腿             | legs up                                              |
| 手放背后           | arms behind back , hidden hands                      |
| 衬衫               | shirt                                                |
| 长袖               | long sleeves                                         |
| 连帽衫             | hoodie                                               |
| 褶边               | frills                                               |
| 喇叭裤             | bloomers                                             |
| 白色连衣裙         | white dress                                          |
| 捆绑               | bondage , bondage body , bondage foot , bondage hand |
| 蹲下               | crouch , squatting                                   |
| 真画风           | photorealistic                                       |
| 跪下               | kneel down                                           |
| 湿身               | wet body                                             |


[^4]:[Paper朱整理优化方法](https://pan.baidu.com/s/1VWr7OLvAbu1KIoTPEs2wwQ?pwd=y8lk)

[^5]:[参数图](https://m.weibo.cn/status/4823585938735546)

[^6]:[SD金矿](https://rentry.org/sdupdates#hall-of-fame)

[^7]:[风格模型训练](https://www.bilibili.com/video/BV1ae4y1S7v9/)

[^8]:[迭代草图](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/2473)

[^9]:[交替单词](https://github.com/AUTOMATIC1111/stable-diffusion-webui/pull/1733)

[^10]:[角色与画风tag训练十问](https://www.bilibili.com/video/BV1xt4y1F7Y2/)

[^11]:[WebUI即将引入重磅更新，大幅提升图像品质](https://www.bilibili.com/read/cv19102552)
