# Stable Diffusion AIACC加速版部署

>**免责声明：**本服务由第三方提供，我们尽力确保其安全性、准确性和可靠性，但无法保证其完全免于故障、中断、错误或攻击。因此，本公司在此声明：对于本服务的内容、准确性、完整性、可靠性、适用性以及及时性不作任何陈述、保证或承诺，不对您使用本服务所产生的任何直接或间接的损失或损害承担任何责任；对于您通过本服务访问的第三方网站、应用程序、产品和服务，不对其内容、准确性、完整性、可靠性、适用性以及及时性承担任何责任，您应自行承担使用后果产生的风险和责任；对于因您使用本服务而产生的任何损失、损害，包括但不限于直接损失、间接损失、利润损失、商誉损失、数据损失或其他经济损失，不承担任何责任，即使本公司事先已被告知可能存在此类损失或损害的可能性；我们保留不时修改本声明的权利，因此请您在使用本服务前定期检查本声明。如果您对本声明或本服务存在任何问题或疑问，请联系我们。

## 概述
stable diffusion可以通过使用文字生成图片，在整个pipeline中，包含CLIP或其他模型从文字中提取隐变量；通过使用UNET或其他生成器模型进行图片生成。通过逐步扩散(Diffusion)，逐步处理图像，使得图像的生成质量更高。**通过本文，客户可以搭建一个stable diffusion的webui框架，并使用aiacctorch加速图片生成速度。在512x512分辨率下，AIACC加速能将推理性能从1.91s降低至0.88s，性能提升至2.17倍，同时AIACC支持任意多LORA权重加载，且不影响性能。**
aiacctorch支持优化基于Torch框架搭建的模型，通过对模型的计算图进行切割，执行层间融合，以及高性能OP实现，大幅度提升PyTorch的推理性能。您无需指定精度和输入尺寸，即可通过JIT编译的方式对PyTorch框架下的深度学习模型进行推理优化。具体详见[手动安装AIACC-Inference（AIACC推理加速）Torch版](https://help.aliyun.com/document_detail/317822.html)。


![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/125679/1670483445654-9ef3f9af-bfb5-4324-9b68-6ca0cc211e24.png#clientId=ub5129ed7-e9e9-4&from=paste&height=277&id=Khxxe&originHeight=277&originWidth=720&originalType=binary&ratio=1&rotation=0&showTitle=false&size=196355&status=done&style=none&taskId=u80ac7f0f-1708-4f2e-8adb-c20191e80e9&title=&width=720)
# 计算巢实例创建
## 创建实例
访问[计算巢实例](https://computenest.console.aliyun.com/user/cn-hangzhou/recommendService)，点击创建
**Stable diffusion AIACC加速社区版。**
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685066580791-055c0f07-8401-4c4c-820b-98900b77ae72.png#clientId=u98f890e2-bf6b-4&from=paste&height=1736&id=u1017be6c&originHeight=1736&originWidth=2446&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1332479&status=done&style=none&taskId=uff518c70-a800-4af8-97d0-cce52405479&title=&width=2446)
选择所需地域:
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685066681428-c9ad983f-eba7-4705-a508-1e7e79d5809f.png#clientId=u98f890e2-bf6b-4&from=paste&height=632&id=uc1466379&originHeight=632&originWidth=2022&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1060858&status=done&style=none&taskId=u1222a295-5b65-4e09-b52c-35be36b94f7&title=&width=2022)
勾选实例类型，并填写实例密码：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685066734916-f5216e58-a314-4bb4-a4c6-3c07efe15ca1.png#clientId=u98f890e2-bf6b-4&from=paste&height=986&id=u4cfcc128&originHeight=986&originWidth=2168&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1877143&status=done&style=none&taskId=u641f88d6-6a62-4d88-ba0f-224238d2460&title=&width=2168)
填写登录用户名和密码，用户名和密码在后续登录webui时使用:
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685066772866-82dbbcfd-a808-4800-84aa-a43596df8239.png#clientId=u98f890e2-bf6b-4&from=paste&height=414&id=ua326f875&originHeight=414&originWidth=2158&originalType=binary&ratio=1&rotation=0&showTitle=false&size=748549&status=done&style=none&taskId=uc9af6f1e-5916-4100-8449-ef9f6361b69&title=&width=2158)
点击下一步:确认订单，并勾选服务条款，点击创建：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685066899702-b24b6a24-32f4-49be-a797-47961dd02202.png#clientId=u98f890e2-bf6b-4&from=paste&height=260&id=ub402146f&originHeight=448&originWidth=970&originalType=binary&ratio=1&rotation=0&showTitle=false&size=409082&status=done&style=none&taskId=u93db4c88-6d08-4357-8d40-1aac1c578f7&title=&width=564)
稍等片刻，等待部署完成。
# 执行图片生成测试
## 登录页面
点击计算巢控制台中的实例:
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685067196374-bffd7d93-e725-49b6-9fb0-7af6aa45aaea.png#clientId=u98f890e2-bf6b-4&from=paste&height=812&id=t9g0Z&originHeight=812&originWidth=2324&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1637587&status=done&style=none&taskId=ua11931df-4ef5-469d-9ab7-8188d45435f&title=&width=2324)
点击其中的endpoint网址：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685067348654-6ef6bc5e-b291-4a37-98d0-fc2e40c30b2a.png#clientId=u98f890e2-bf6b-4&from=paste&height=1190&id=u76383091&originHeight=1190&originWidth=2588&originalType=binary&ratio=1&rotation=0&showTitle=false&size=2541533&status=done&style=none&taskId=udaf1cb4b-4017-454a-ba32-218624e9c6f&title=&width=2588)
## 测试
输入登录信息中的用户名和密码登录:
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685067431628-1f7b4b83-eda3-4d7b-9522-6959a7c69047.png#clientId=u98f890e2-bf6b-4&from=paste&height=1710&id=u9571d219&originHeight=1710&originWidth=3128&originalType=binary&ratio=1&rotation=0&showTitle=false&size=3856733&status=done&style=none&taskId=u49d8db20-a82b-458d-8843-e0c0a5bc6ce&title=&width=3128)
进入测试页面:
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685067502612-dcd8d4a1-5fc6-487c-91f0-99c5a495e004.png#clientId=u98f890e2-bf6b-4&from=paste&height=1796&id=u9b4ce66b&originHeight=1796&originWidth=3102&originalType=binary&ratio=1&rotation=0&showTitle=false&size=4740052&status=done&style=none&taskId=uab7a4c28-0ced-41e0-b44c-9eefcfaac8a&title=&width=3102)
输入中文prompt，例如:
```bash
铁马冰河入梦来，概念画，科幻，玄幻，3D
```
点击生成，即可生成如下图片：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685067706263-7902a058-a52a-4a5c-875a-36b05242cb28.png#clientId=u98f890e2-bf6b-4&from=paste&height=1073&id=uc717add0&originHeight=1073&originWidth=1517&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1153472&status=done&style=none&taskId=ua208cd42-189f-4f11-b087-bcba46b4103&title=&width=1517)
此处我们使用了aiacctorch加速了模型，可以看到单张图片推理时间为**0.88s**。**(注意，首次应用aiacctorch进行图片生成，或者切换模型后的首次图片生成，会多占用30s时间，以进行aiacctorch模型加载)。**
## 模型切换
在上述的模型下载过程中，我们下载了3个模型:

- Taiyi-Stable-Diffusion-1B-Chinese-v0.1：中文Stable Diffusion模型
- Taiyi-Stable-Diffusion-1B-Anime-Chinese-v0.1：中文Stable Diffusion动漫模型
- v1-5-pruned-emaonly.safetensors：Stable Diffusion v1.5模型

我们可以根据输入文字以及生成图片风格，切换模型进行模型推理，例如我们可以通过左上侧的选项卡，选择Taiyi-Stable-Diffusion-1B-Anime-Chinese-v0.1模型：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685067936263-c60088be-72fc-4a19-9b9b-d1c4c65062be.png#clientId=u98f890e2-bf6b-4&from=paste&height=1167&id=u4d5d817c&originHeight=1167&originWidth=1530&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1485101&status=done&style=none&taskId=u9d9058db-8961-4c96-9db5-ba5da9d0c4e&title=&width=1530)
而后输入提示词和反向提示词:
```bash
1个女孩,绿眼,棒球帽,金色头发,闭嘴,帽子,看向阅图者,短发,简单背景,单人,上半身,T恤
Negative prompt: 水彩,漫画,扫描件,简朴的画作,动画截图,3D,像素风,原画,草图,手绘,铅笔
```
则可生成如下图所示的动漫风格图像:
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685068082354-35714a36-ad9b-417e-a2b3-8c263e9d719a.png#clientId=u98f890e2-bf6b-4&from=paste&height=1098&id=u8862e4cd&originHeight=1098&originWidth=1516&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1187134&status=done&style=none&taskId=uf89d1686-d412-4163-9c82-3b6fb8796fa&title=&width=1516)
## LORA功能试用
aiacctorch支持LORA加速，可以在prompt中加入LORA支持文本:  <lora:iuV35.uv1P:1>
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685071552670-8b89d82c-69e7-4c64-abf0-870dfd9a5c99.png#clientId=u29481c1f-53df-4&from=paste&height=1101&id=u844f82ab&originHeight=1101&originWidth=1516&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1196527&status=done&style=none&taskId=ud3b243fe-81a3-4895-befb-2c9aa7e1596&title=&width=1516)
可见在aiacctorch加速优化下，LORA加载后性能与加载前相同。
## 禁用AIACC加速
我们也可以在settings中禁用aiacctorch。点击settings选项卡，选中aiacctorch，去掉"Apply Aiacctorch in Unet to speedup the whole network inference when loading models"前方的勾，而后点击应用设置：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685068221633-96fc9ca8-f591-49f0-80f7-91ced758806c.png#clientId=u98f890e2-bf6b-4&from=paste&height=1132&id=uf0ac43fa&originHeight=1132&originWidth=1522&originalType=binary&ratio=1&rotation=0&showTitle=false&size=996312&status=done&style=none&taskId=uf27eaf39-e961-4260-88ed-1ee7821c8cc&title=&width=1522)
重复图片生成操作，可见推理时间为**1.97s**：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685070075626-8025d0b9-5744-436a-a193-6c51e653a31e.png#clientId=u98f890e2-bf6b-4&from=paste&height=1096&id=u0f1aa2ec&originHeight=1096&originWidth=1512&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1188283&status=done&style=none&taskId=u524e5f9b-2bfb-4e71-b065-329bed77c95&title=&width=1512)
禁用AIACC后，我们也可增加LORA权重进行推理，得到耗时为**2.35s**，大幅高于AIACC优化后的延迟，AIACC加速可达**2.67倍**:
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/125679/1685071972421-000239fc-f098-4b63-939d-1c073d8626d2.png#clientId=u29481c1f-53df-4&from=paste&height=1098&id=u4417d568&originHeight=1098&originWidth=1508&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1183686&status=done&style=none&taskId=ubc32cb56-3370-4511-be29-6683430e23e&title=&width=1508)
