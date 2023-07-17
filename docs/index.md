# Stable Diffusion AIACC加速版部署

>**免责声明：**本服务由第三方提供，我们尽力确保其安全性、准确性和可靠性，但无法保证其完全免于故障、中断、错误或攻击。因此，本公司在此声明：对于本服务的内容、准确性、完整性、可靠性、适用性以及及时性不作任何陈述、保证或承诺，不对您使用本服务所产生的任何直接或间接的损失或损害承担任何责任；对于您通过本服务访问的第三方网站、应用程序、产品和服务，不对其内容、准确性、完整性、可靠性、适用性以及及时性承担任何责任，您应自行承担使用后果产生的风险和责任；对于因您使用本服务而产生的任何损失、损害，包括但不限于直接损失、间接损失、利润损失、商誉损失、数据损失或其他经济损失，不承担任何责任，即使本公司事先已被告知可能存在此类损失或损害的可能性；我们保留不时修改本声明的权利，因此请您在使用本服务前定期检查本声明。如果您对本声明或本服务存在任何问题或疑问，请联系我们。

## 概述
stable diffusion可以通过使用文字生成图片，在整个pipeline中，包含CLIP或其他模型从文字中提取隐变量；通过使用UNET或其他生成器模型进行图片生成。通过逐步扩散(Diffusion)，逐步处理图像，使得图像的生成质量更高。**通过本文，客户可以搭建一个stable diffusion的webui框架，并使用aiacctorch加速图片生成速度。在512x512分辨率下，AIACC加速能将推理性能从1.91s降低至0.88s，性能提升至2.17倍，同时AIACC支持任意多LORA权重加载，且不影响性能。**
aiacctorch支持优化基于Torch框架搭建的模型，通过对模型的计算图进行切割，执行层间融合，以及高性能OP实现，大幅度提升PyTorch的推理性能。您无需指定精度和输入尺寸，即可通过JIT编译的方式对PyTorch框架下的深度学习模型进行推理优化。具体详见[手动安装AIACC-Inference（AIACC推理加速）Torch版](https://help.aliyun.com/document_detail/317822.html)。


![image.png](1.png)
# 计算巢实例创建
## 创建实例
访问[计算巢实例](https://computenest.console.aliyun.com/user/cn-hangzhou/recommendService)，点击创建
**Stable diffusion AIACC加速社区版。**
![image.png](2.png)
选择所需地域:
![image.png](3.png)
勾选实例类型，并填写实例密码：
![image.png](4.png)
填写登录用户名和密码，用户名和密码在后续登录webui时使用:
![image.png](5.png)
点击下一步:确认订单，并勾选服务条款，点击创建：
![image.png](6.png)
稍等片刻，等待部署完成。
# 执行图片生成测试
## 登录页面
点击计算巢控制台中的实例:
![image.png](7.png)
点击其中的endpoint网址：
![image.png](8.png)
## 测试
输入登录信息中的用户名和密码登录:
![image.png](9.png)
进入测试页面:
![image.png](10.png)
输入中文prompt，例如:
```bash
铁马冰河入梦来，概念画，科幻，玄幻，3D
```
点击生成，即可生成如下图片：
![image.png](11.png)

此处我们使用了aiacctorch加速了模型，可以看到单张图片推理时间为**0.88s**。**(注意，首次应用aiacctorch进行图片生成，或者切换模型后的首次图片生成，会多占用30s时间，以进行aiacctorch模型加载)。**
## 模型切换
在上述的模型下载过程中，我们下载了3个模型:

- Taiyi-Stable-Diffusion-1B-Chinese-v0.1：中文Stable Diffusion模型
- Taiyi-Stable-Diffusion-1B-Anime-Chinese-v0.1：中文Stable Diffusion动漫模型
- v1-5-pruned-emaonly.safetensors：Stable Diffusion v1.5模型

我们可以根据输入文字以及生成图片风格，切换模型进行模型推理，例如我们可以通过左上侧的选项卡，选择Taiyi-Stable-Diffusion-1B-Anime-Chinese-v0.1模型：
![image.png](12.png)
而后输入提示词和反向提示词:
```bash
1个女孩,绿眼,棒球帽,金色头发,闭嘴,帽子,看向阅图者,短发,简单背景,单人,上半身,T恤
Negative prompt: 水彩,漫画,扫描件,简朴的画作,动画截图,3D,像素风,原画,草图,手绘,铅笔
```
则可生成如下图所示的动漫风格图像:
![image.png](13.png)
## LORA功能试用
aiacctorch支持LORA加速，可以在prompt中加入LORA支持文本:  <lora:iuV35.uv1P:1>
![image.png](14.png)
可见在aiacctorch加速优化下，LORA加载后性能与加载前相同。

## Controlnet功能试用
Controlnet可以帮助我们生成与原始图相似风格或相似布局的图片，这里以canny为例介绍controlnet功能。
打开Controlnet选项卡，选择controlType为Canny，并上传一张图片，以LENA图为例，进行如下设置：

![image.png](15.png)

点击生成：
![image.png](16.png)
可见性能为2.36s。

## 禁用AIACC加速
我们也可以在settings中禁用aiacctorch。点击settings选项卡，选中aiacctorch，去掉"Apply Aiacctorch in Unet to speedup the whole network inference when loading models"前方的勾，而后点击应用设置：
![image.png](17.png)

重复图片生成操作，可见推理时间为**1.97s**：
![image.png](18.png)

禁用AIACC后，我们也可增加LORA权重进行推理，得到耗时为**2.35s**，大幅高于AIACC优化后的延迟，AIACC加速可达**2.67倍**:
![image.png](19.png)

# F&Q
### 如何重启服务
- 可使用如下命令停止服务:
```shell
systemctl stop sdwebui 
```
- 可使用如下命令打开服务:
```shell
systemctl start sdwebui
```
### 如何查看log
```shell
cat /var/log/sdwebui.log
```
