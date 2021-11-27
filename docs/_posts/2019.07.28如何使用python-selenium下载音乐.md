---
title: 如何使用python+selenium下载音乐
date: 2019-07-28 20:23:19
tags: 技巧
---
搭建Selenium+python框架:[这里](https://blog.csdn.net/tyx199397/article/details/79268111)

其中关键步骤为:
```
1. https://github.com/mozilla/geckodriver/releases 选择合适的版本，因为我是64位的windows系统，所以我选择下载geckodriver-v0.19.1-win64.zip
2. 下载完成后将geckodriver-v0.19.1-win64.zip解压到python的根录下。这里我的python安装路径在C:\Python3，所以我解压到C:\Python3下面，得到了C:\Python3\geckodriver.exe.
```

搭建好后就可以编码了
```
    title_str = driver.title.split(" ")[0]
    print(driver.title)
    driver.find_element_by_css_selector(".js_all_play").click()
    time.sleep(3)
    windows = driver.window_handles
    driver.switch_to.window(windows[-1])
    if driver.find_element_by_css_selector("#h5audio_media") is None:
        print("下载失败")
        return
    music_url = driver.find_element_by_css_selector("#h5audio_media").get_attribute("src")
    driver.quit()
    return music_url, title_str


# music_src, title = get_music_url_from_qq("https://y.qq.com/n/yqq/song/1572025_num.html?ADTAG=h5_playsong&no_redirect=1")
music_src, title = get_music_url_from_blue_dance("http://m.lanwuzhe.com/app/audio/audio.html?id=2664")
print(music_src)
url1 = music_src
urllib.
```
