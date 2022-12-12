> ### **正式代码：**

```python
import re
import os,sys
import jieba
from selenium import webdriver
from urllib.parse import quote
import time
import secrets
from lxml import etree
import requests
from PIL import Image
from io import BytesIO
from os import path
import random
import pyttsx3
from moviepy.editor import *
import librosa
import math
import time
try:
    import xlrd
except:
    os.system('pip install -U xlrd')
    import xlrd
import logging

# log_file = os.path.join(os.getcwd(),'logs/liveappapi.log')
# log_format = '[%(asctime)s] [%(levelname)s] %(message)s'
# logging.basicConfig(format=log_format,filename=log_file,filemode='w',level=logging.DEBUG)
# console = logging.StreamHandler()
# console.setLevel(logging.DEBUG)
# formatter = logging.Formatter(log_format)
# console.setFormatter(formatter)
# logging.getLogger('').addHandler(console)

# 根据关键字获取图片下载信息
def get_image_ulr(base_url,keword):
    driver = webdriver.Chrome(executable_path="chromedriver.exe")
    search_keyword = base_url + quote(quote(keword))
    print("开始访问")
    driver.get(search_keyword)
    time.sleep(10)
    return driver

# 根据关键字获取图片下载地址（关键字不转义）
def get_image_ulr_without_righteousness(base_url,keword):
    driver = webdriver.Chrome(executable_path="chromedriver.exe")
    search_keyword = base_url + keword
    print("开始访问")
    driver.get(search_keyword)
    time.sleep(10)
    return driver

# 无关键字获取图片下载信息
def get_image_ulr_without_keyword(base_url):
    driver = webdriver.Chrome(executable_path="chromedriver.exe")
    search_keyword = base_url
    print("开始访问")
    driver.get(search_keyword)
    time.sleep(10)
    return driver

# 根据链接下载图片,并保存到本地,随机生成文件名
def download_image(url,save_url):
    r = requests.get(url)
    img = Image.open(BytesIO(r.content))
    filename1 = path.basename(url)
    # 下载保存路径
    savelocation = path.join(save_url, filename1)
    img.save(savelocation)
   
# 根据链接下载图片,并保存到本地,生成指定文件名 
def download_image_to_name(url,save_url,file_name,suffix):
    r = requests.get(url)
    img = Image.open(BytesIO(r.content))
    filename1 = file_name
    # 下载保存路径
    savelocation = path.join(save_url, filename1 + suffix)
    img.save(savelocation)

# generator 转化为list
def generator_to_list(generator):
    list = []
    for key in generator: 
        print(key)
        list = list + [''.join(key)]
    return list

# 生成音频，即文字转音频
def generator_audio(part_content,timbre,save_url,rate,audio_suffix):    
    engine = pyttsx3.init()
    voices = engine.getProperty('voices')
    engine.setProperty('voice', voices[timbre].id)
    #设置速度
    engine.setProperty('rate', rate)
    engine.save_to_file(''.join(part_content), save_url + part_content + audio_suffix)
    engine.runAndWait()

# 生成字幕
def generator_subtitle(clip,part_content,position,fontsize,fontcolor,duration,subtitle_font):
    txt_clip = TextClip(part_content,font=subtitle_font,fontsize=fontsize,color=fontcolor)
    txt_clip = txt_clip.set_pos(('center',position),relative=True).set_duration(duration)
    video = CompositeVideoClip([clip, txt_clip])
    return video

# 取倒数
def get_reciprocal(audio_duration):
    #借助除法和小数化取audio_duration的倒数fps
    new_audio_duration = 10000//audio_duration
    fps = float('0.'+ str(new_audio_duration))
    return fps

# 选择图片源获取图片
def select_picture_source_to_get_url(key_word_name,picture_source):
    
    # 百度图库图片源
    if(picture_source == 2): 
        #每句话取一个关键字
        # 测试百度图片源
        driver = get_image_ulr_without_righteousness(base_url2,key_word_name)
        response_text = driver.page_source
        print(response_text)
        html = etree.HTML(response_text)
        image_list = html.xpath('//div[@id="imgid"]//li[contains(@class,"imgitem")]')
        if(len(image_list) == 0):
            driver = get_image_ulr_without_righteousness(base_url2,'表情包')
            response_text = driver.page_source
            print(response_text)
            html = etree.HTML(response_text)
            image_list = html.xpath('//div[@id="imgid"]//li[contains(@class,"imgitem")]')
            #每个关键字取一个图片
            image_key = random.choice(image_list)
            image_url = image_key.xpath('.//@data-objurl')
            print(image_url)
            image_link = ''.join(image_url)
        else:
            #每个关键字取一个图片
            image_key = random.choice(image_list)
            # for image_key in image_list:       
            image_url = image_key.xpath('.//@data-objurl')
            print(image_url)
            image_link = ''.join(image_url)
    # 表情网图片源
    else:  
        #每句话取一个关键字
        driver = get_image_ulr(base_url,key_word_name)
        response_text = driver.page_source
        print(response_text)
        html = etree.HTML(response_text)
        image_list = html.xpath('//div[@id="container"]//img[contains(@class,"image")]')
        if(len(image_list) == 0):
            driver = get_image_ulr_without_keyword(boutique_url)
            response_text = driver.page_source
            print(response_text)
            html = etree.HTML(response_text)
            image_list = html.xpath('//div[@id="container"]//img[contains(@class,"image")]')
            #每个关键字取一个图片
            image_key = random.choice(image_list)
            image_url = image_key.xpath('.//@src')
            print(image_url)
            image_link = ''.join(image_url)
        else:
            #每个关键字取一个图片
            image_key = random.choice(image_list)
            image_url = image_key.xpath('.//@src')
            print(image_url)
            image_link = ''.join(image_url)               
    return image_link

# 指定路径下所有MP4文件拼接
def concatenate_MP4_videoclips(path,result_file_url,result_file_name):
    video_list = []
    for file_name in os.listdir(path):
        suffix = os.path.splitext(file_name)[-1]
        if(suffix == '.mp4'):
            video_list.append(VideoFileClip(path + file_name))
            print(file_name)
    print("拼接视频===============>")
    # Merge video,注意此时分辨率设置为最大即最大片段那个分辨率，否则会花屏
    final_clip = concatenate_videoclips(video_list,method="compose")
    # Save Merged video file
    final_clip.write_videofile(result_file_url + 'result//' +result_file_name + ".mp4")

# 读取文件并执行
def get_file_and_run(file,save_url,content,cutter,timbre,reading_speed,reading_volume,subtitle_font_size,subtitle_color,subtitle_font,subtitle_height,picture_source):
    videoSourceFile = os.path.join(os.getcwd(),file)
    if not os.path.exists(videoSourceFile):
        # logging.error('文件不存在！！！')
        sys.exit()
    testCase = xlrd.open_workbook(videoSourceFile)
    table = testCase.sheet_by_index(0)
    for i in range(1,table.nrows):
        if ((table.cell(i,13).value.replace('\n','').replace('\r','') == 'yes') and (table.cell(i,12).value.replace('\n','').replace('\r','') == 'no')):
            print("No:"+str(int(table.cell(i,0).value)))           
            print("Save URL:"+str(table.cell(i,1).value).replace('\n','').replace('\r',''))
            if len(str(table.cell(i,1).value).replace('\n','').replace('\r','').strip())>0:
                save_url = str(table.cell(i,1).value).replace('\n','').replace('\r','').strip()
            print("Copywriting Content:"+ str(table.cell(i,2).value).replace('\n','').replace('\r',''))
            if len(str(table.cell(i,2).value).replace('\n','').replace('\r','').strip())>0:
                content = str(table.cell(i,2).value).replace('\n','').replace('\r','').strip()
            else:
                continue          
            print("Cutter:"+ str(table.cell(i,3).value).replace('\n','').replace('\r',''))
            if len(str(table.cell(i,3).value).replace('\n','').replace('\r','').strip())>0:
                cutter = str(table.cell(i,3).value).replace('\n','').replace('\r','').strip()                    
            print("Timbre:" + str(table.cell(i,4).value))
            if((len(str(table.cell(i,4).value))>0) and ((int(table.cell(i,4).value))== 0 or  (int(table.cell(i,4).value))==1 or (int(table.cell(i,4).value))== 2 or (int(table.cell(i,4).value))== 3 or (int(table.cell(i,4).value))== 4)):
                timbre = int(table.cell(i,4).value)  
            print("Reading Speed:"+str(table.cell(i,5).value))
            if((len(str(table.cell(i,5).value))>0) and (int(table.cell(i,5).value)>=0 and int(table.cell(i,5).value)<= 500)):
                reading_speed = int(table.cell(i,5).value)   
            print("Reading Volume:"+str(table.cell(i,6).value))
            if((len(str(table.cell(i,6).value))) and (float(table.cell(i,6).value) >=0 and float(table.cell(i,6).value)<= 1)):
                reading_volume = float(table.cell(i,6).value)
            print("Subtitle Font Size:"+str(table.cell(i,7).value))
            if((len(str(table.cell(i,7).value))>0) and (int(table.cell(i,7).value)<=200 and int(table.cell(i,7).value) >= 0)):
                subtitle_font_size = int(table.cell(i,7).value)
            print("Subtitle Color:"+str(table.cell(i,8).value).replace('\n','').replace('\r',''))
            if(len(str(table.cell(i,8).value).replace('\n','').replace('\r','').strip())>0):
                subtitle_color = str(table.cell(i,8).value).replace('\n','').replace('\r','').strip()
            print("Subtitle Font:"+str(table.cell(i,9).value).replace('\n','').replace('\r',''))
            if(len(str(table.cell(i,9).value).replace('\n','').replace('\r','').strip())>0):
                subtitle_font = str(table.cell(i,9).value).replace('\n','').replace('\r','').strip()
            print("Subtitle Height:"+str(table.cell(i,10).value))
            if((len(str(table.cell(i,10).value))>0) and (float(table.cell(i,10).value) >=0 and float(table.cell(i,10).value)<= 1)):
                subtitle_height = float(table.cell(i,10).value)
            # print("Used:")
            # print(str(table.cell(i,11).value).replace('\n','').replace('\r',''))
            print("Picture Source:"+str(table.cell(i,11).value))
            if((len(str(table.cell(i,11).value))>0) and (int(table.cell(i,11).value)==2)):
                picture_source = int(table.cell(i,11).value)
            # 针对每个文件创建一个文件夹用于存放该文案相关文件
            timestampforforlder = int(time.time()*1000)
            os.mkdir((save_url + str(timestampforforlder)))
            save_url = save_url + str(timestampforforlder) + '\\'
        
            # 将文案按照指定分隔符分割
            content_list = content.split(cutter)
            
            for part_content in content_list:
                #获取毫秒级时间戳
                timestamp = int(time.time()*1000)
                
                #将一段话分为几句
                # 分词 每句话选个关键字
                part_content_generator = jieba.cut(part_content)
                part_content_list = generator_to_list(part_content_generator)
                key_word_name = random.choice(part_content_list)
                
                # 针对关键字选择视频源并获取图片下载路径      
                image_link = select_picture_source_to_get_url(key_word_name,picture_source)
                
                #获取文件后缀
                suffix = os.path.splitext(image_link)[-1]
                # if(suffix != '.jpg' and suffix != '.png' and suffix != '.gif' and suffix != '.webp'):
                #    suffix = '.jpg' 
                   
                #下载图片
                download_image_to_name(image_link,save_url,part_content,suffix)
                
                # 生成音频，即文字转音频
                generator_audio(part_content,timbre,save_url,reading_speed,".mp3")
                
                #获取音频文件时长
                duration = librosa.get_duration(filename = (save_url + part_content + ".mp3"))
                
                # 获取到的时长向上取整，单位为秒
                print(duration)
                audio_duration = math.ceil(duration)
                
                # 借助除法和小数化取audio_duration的倒数fps
                fps = get_reciprocal(audio_duration)
                
                # 用图片生成视频，视频时长取决于fps即音频时长倒数
                clip = ImageSequenceClip([save_url + part_content + suffix], fps=(fps))
                
                # 设置视频音量   
                clip = clip.volumex(reading_volume)

                # 为视频生成字幕        
                video = generator_subtitle(clip,part_content,subtitle_height,subtitle_font_size,subtitle_color,audio_duration,subtitle_font)

                # 读取音频并添加进视频
                my_audioclip = AudioFileClip(save_url + part_content + ".mp3")
                video = video.set_audio(my_audioclip)
                
                # 导出视频
                video.write_videofile(save_url + str(timestamp) + part_content +".mp4")
                time.sleep(5)
                
                print("========>>End")    
            result_file_name = content[0:4]    
            # 剪辑完成的视频做拼接
            concatenate_MP4_videoclips(save_url,result_save_url,result_file_name)
            
            # print("Enable:")
            # print(str(table.cell(i,12).value).replace('\n','').replace('\r',''))
            
        else:
            continue

# 设置默认值并读取文件内容
if __name__ == '__main__':  
    
    # # Excel读取
    file = 'E:\\Dealing\\PythonForMovie\\截取视频\\python\\data\\TestCasePre.xls'
    
    #正文内容
    content = ''
    
    #图片资源：
    
    #表情包库
    base_url = "https://fabiaoqing.com/search/bqb/keyword/"
    #表情包精品库
    boutique_url = "https://fabiaoqing.com/biaoqing"
    
    #百度图库
    base_url2 = "https://image.baidu.com/search/index?tn=baiduimage&word="
    # boutique_url2 = "https://image.baidu.com/search/index?tn=baiduimage&word="

    # 中转文件存储路径
    save_url = 'E:\\Dealing\\PythonForMovie\\截取视频\\python\\data\\'
    
    # 最终视频存储路径
    result_save_url = 'E:\\Dealing\\PythonForMovie\\截取视频\\python\\data\\'
    
    # 文案分隔符
    cutter = '，'
    
    # 阅读音音色选择，语音包 0：经典中文（女声），1：英文（美式），2：英文（美式），3：粤语（香港），4：中文（台湾）
    timbre = 3
    
    # 阅读速度 允许范围（0-500）
    reading_speed = 120
    
    # 阅读音量 允许范围：0-1
    reading_volume = 0.8
    
    # 字幕字号选择 允许范围 0-200
    subtitle_font_size = 30
    
    # 字幕颜色选择
    subtitle_color = 'purple'
    
    # 字幕字体选择 默认黑体，支持选择：
    subtitle_font = 'simhei.ttf'
    
    # 字幕高度，0.84表示距离top 84%
    subtitle_height = 0.84
    
    # 图片资源库选择 1.表情网，2.百度图库
    picture_source = 1
    
    # 读取excel文件中的内容
    get_file_and_run(file,save_url,content,cutter,timbre,reading_speed,reading_volume,subtitle_font_size,subtitle_color,subtitle_font,subtitle_height,picture_source)

```



> ### **测试代码：**

```python
import re
import os,sys
import jieba
import jieba.analyse
from selenium import webdriver
from urllib.parse import quote
import time
import secrets
from lxml import etree
import requests
from PIL import Image
from io import BytesIO
from os import path
import random
import pyttsx3
from moviepy.editor import *
import librosa
import math
import time
try:
    import xlrd
except:
    os.system('pip install -U xlrd')
    import xlrd
import logging

# 字幕颜色及字体测试demo
# clip = ImageSequenceClip(['E:\\Dealing\\PythonForMovie\\截取视频\\python\\test\\能解世间惆怅.jpg'], fps=(0.2))
# # black,white,
# txt_clip = TextClip('你好呀！',font='simhei.ttf',fontsize=30,color='purple')
# txt_clip = txt_clip.set_pos(('center',0.84),relative=True).set_duration(5)
# video = CompositeVideoClip([clip, txt_clip])
# video.write_videofile('E:\\Dealing\\PythonForMovie\\截取视频\\python\\test\\' + '能解世间惆怅' +".mp4")

# 新建文件夹测试demo
# os.mkdir(r'E:\\Dealing\\PythonForMovie\\截取视频\\python\\test\\测试文件夹2')


# 视频拼接测试demo
# path = 'E:\\Dealing\\PythonForMovie\\截取视频\\python\\data\\1664366200339\\'
# file_name = '愿你早日开启'
# result_file_url = 'E:\\Dealing\\PythonForMovie\\截取视频\\python\\data\\result\\'
# video_list = []
# for file_name in os.listdir(path):
#     suffix = os.path.splitext(file_name)[-1]
#     if(suffix == '.mp4'):
#         video_list.append(VideoFileClip(path + file_name))
#         print(file_name)
# print("拼接视频===============>")
# # Merge video,注意此时分辨率设置为最大即最大片段那个分辨率，否则会花屏
# final_clip = concatenate_videoclips(video_list,method="compose")
# # Save Merged video file
# final_clip.write_videofile(result_file_url + file_name + ".mp4")



# 测试jieba分词权重

# #读取文件,返回一个字符串，使用utf-8编码方式读取，该文档位于此python同以及目录下
# content  = '小时候以为，长大后会有好多意想不到的事情以及惊喜，长大以后才明白，生活不仅没有惊喜，还处处让你难过'
# content_list = content.split('，')   
# for part_content in content_list:
#     print('===================================>')
#     tags = jieba.analyse.extract_tags(part_content,topK=10,withWeight=True,allowPOS=("nr")) 
#     print(tags)




# # # Excel读取
# file = 'E:\\Dealing\\PythonForMovie\\截取视频\\python\\data\\TestCasePre.xls'
# videoSourceFile = os.path.join(os.getcwd(),file)
# if not os.path.exists(videoSourceFile):
#     # logging.error('文件不存在！！！')
#     sys.exit()
# testCase = xlrd.open_workbook(videoSourceFile)
# table = testCase.sheet_by_index(0)
# for i in range(1,table.nrows):
#     if ((table.cell(i,12).value.replace('\n','').replace('\r','') == 'yes') and (table.cell(i,11).value.replace('\n','').replace('\r','') == 'no')):
#         print("No:")
#         print(int(table.cell(i,0).value))
#         print("Save URL:")
#         print(str(table.cell(i,1).value).replace('\n','').replace('\r',''))
#         print("Copywriting Content:")
#         print(str(table.cell(i,2).value).replace('\n','').replace('\r',''))
#         print("Cutter:")
#         print(str(table.cell(i,3).value).replace('\n','').replace('\r',''))
#         print("Timbre:")
#         print(int(table.cell(i,4).value))
#         print("Reading Speed:")
#         print(int(table.cell(i,5).value))
#         print("Reading Volume:")
#         print(float(table.cell(i,6).value))
#         print("Subtitle Font Size:")
#         print(int(table.cell(i,7).value))
#         print("Subtitle Color:")
#         print(str(table.cell(i,8).value).replace('\n','').replace('\r',''))
#         print("Subtitle Font:")
#         print(str(table.cell(i,9).value).replace('\n','').replace('\r',''))
#         print("Subtitle Height:")
#         print(float(table.cell(i,10).value))
#         print("Used:")
#         print(str(table.cell(i,11).value).replace('\n','').replace('\r',''))
#         print("Enable:")
#         print(str(table.cell(i,12).value).replace('\n','').replace('\r',''))
#     else:
#         continue



# # 制作动图
# #import imageio
# #imageio.plugins.ffmpeg.download()
# import moviepy.editor as mpy

# #视频文件的本地路径
# content = mpy.VideoFileClip("F:\XunLeiDownload\Movies\yourMZ_bd.mp4")
# # 剪辑78分55秒到79分6秒的片段。注意：不使用resize则不会修改清晰度
# c1 = content.subclip((78,55),(79,6)).resize((480,320))
# # 将片段保存为gif图到python的默认路径，可保存到"C:\Users\Administrator\Desktop"
# c1.write_gif("gav.gif")


# # 声音选择 语音包 1.经典中文（女声），2.英文（美式）3.英文（美式），4.粤语（香港）5.中文（台湾）
# # 生成音频，即文字转音频
# save_url = 'E:\\Dealing\\PythonForMovie\\截取视频\\python\\test\\'
# part_content = 'test'
# audio_suffix = '.mp3'
# # 阅读速度
# reading_speed = 120
# engine = pyttsx3.init()
# # engine.say("Life is short, so we choose python!")
# #设置速度
# engine.setProperty('rate', reading_speed)
# # engine.setProperty('volume', volume+0.25)
# # engine.save_to_file('小时候以为，长大后会有好多意想不到的事情以及惊喜，长大以后才明白，生活不仅没有惊喜，还处处让你难过', save_url + part_content + audio_suffix)
# # engine.runAndWait()


# voices = engine.getProperty('voices')
# i = 0
# for voice in voices:
#     print(voice, voice.id)
# #     engine = pyttsx3.init()
# #     engine.setProperty('rate', reading_speed)
#     engine.setProperty('voice', voice.id)
# #     # engine.say("Hello World!")
#     engine.save_to_file('小时候以为，长大后会有好多意想不到的事情以及惊喜，长大以后才明白，生活不仅没有惊喜，还处处让你难过', save_url + str(i) + part_content + audio_suffix)
#     engine.runAndWait()
#     i = i + 1
# #     engine.stop()

def get_image_ulr(base_url,keword):
    driver = webdriver.Chrome(executable_path="chromedriver.exe")
    search_keyword = base_url + keword
    print("开始访问")
    driver.get(search_keyword)
    time.sleep(10)
    return driver

if __name__ == '__main__':  
    
    base_url2 = "https://image.baidu.com/search/index?tn=baiduimage&word="
    key_word_name = '大笑'
    
    # 测试百度图片源
    driver = get_image_ulr(base_url2,key_word_name)
    response_text = driver.page_source
    print(response_text)
    html = etree.HTML(response_text)
    image_list = html.xpath('//div[@id="imgid"]//li[contains(@class,"imgitem")]')
    if(len(image_list) == 0):
        driver = get_image_ulr(base_url2,'表情包')
        response_text = driver.page_source
        print(response_text)
        html = etree.HTML(response_text)
        image_list = html.xpath('//div[@id="container"]//img[contains(@class,"image")]')
        #每个关键字取一个图片
        image_key = random.choice(image_list)
        image_url = image_key.xpath('.//@src')
        print(image_url)
        image_link = ''.join(image_url)
    else:
        #每个关键字取一个图片
        image_key = random.choice(image_list)
        # for image_key in image_list:       
        image_url = image_key.xpath('.//@data-objurl')
        print(image_url)
        image_link = ''.join(image_url)
```

