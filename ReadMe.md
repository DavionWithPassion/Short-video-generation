
项目用途：
    1.用于快速批量生成短视频

项目实现原理（依赖moviepy）：
    1.对文案进行分段分词
    2.用文件关键词去图片网下载相应图片
    3.用分段后的文案生辰音频
    4.将第二部下载的图片转化为视频（简单轮播）
    5.将视频与音频结合
    6.为视频生成字幕
    7.生成视频
 
 现有用户及功能：
    用法：
    1.在网络中寻找合适文案，将其拷贝于表格的文案字段
    2.执行video_deal文件
    功能：
    1.可以批量生成视频
    2.可以生成字幕并选择字幕颜色尺寸位置等
    3.可以为视频生成声音并选择语言
    
 
项目环境：
    Python 3.9.10
    
 
