# 快手爬虫 解决粉丝数 关注数等字体加密

![image.png](https://cdn.nlark.com/yuque/0/2020/png/97322/1607128457270-0819e577-4819-47f9-a3a2-4d8a90dc3227.png#align=left&display=inline&height=184&margin=%5Bobject%20Object%5D&name=image.png&originHeight=368&originWidth=754&size=112492&status=done&style=none&width=377)<br />想拿一下粉丝数 关注数 描述等<br />发现字体是加密的 elements是这样的<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1607128435174-179522d1-d962-494a-855c-5cb31f67d96b.png#align=left&display=inline&height=102&margin=%5Bobject%20Object%5D&originHeight=102&originWidth=443&size=0&status=done&style=none&width=443)<br />源代码里是这样的<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1607128435278-90c448e3-73a6-4bb8-be87-d26d6232f090.png#align=left&display=inline&height=74&margin=%5Bobject%20Object%5D&originHeight=74&originWidth=952&size=0&status=done&style=none&width=952)<br />找了找js 原来是用
```
&#xed69;&#xe34c;&#xf149;&#xf6d6;&#xf19d;
```
这些玩意 去<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1607128435167-d5648480-2f2d-4427-9cef-78476dd3bb32.png#align=left&display=inline&height=135&margin=%5Bobject%20Object%5D&originHeight=135&originWidth=892&size=0&status=done&style=none&width=892)<br />这个ttf里一一对应 然后用 js + css画出来的<br />找到问题所在，就fuck掉它<br />把js扣出来？用execjs去执行？太LOW了<br />既然做python 那就用python去重写<br />首先用re去拿这个ttf的url (因为每次都变)<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1607128435168-d12d4991-23d7-47ff-911c-e824017d186e.png#align=left&display=inline&height=144&margin=%5Bobject%20Object%5D&originHeight=144&originWidth=882&size=0&status=done&style=none&width=882)<br />先给这玩意下载下来 把这个ttf文件扔fonteditor里 然后去<br />[http://fontstore.baidu.com/static/editor/index.html](http://fontstore.baidu.com/static/editor/index.html)<br />瞅瞅<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/97322/1607128495742-8ae1ab68-b508-4bf1-83e7-fca1f77a7e6d.png#align=left&display=inline&height=328&margin=%5Bobject%20Object%5D&name=image.png&originHeight=656&originWidth=2080&size=200374&status=done&style=none&width=1040)<br />这个时候就发现了东西<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1607128435312-de725fa6-3ac4-45bd-a900-7d271ed7010f.png#align=left&display=inline&height=77&margin=%5Bobject%20Object%5D&originHeight=77&originWidth=1188&size=0&status=done&style=none&width=1188)<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/97322/1607128505741-385f5df0-1f07-43c6-b4df-421d8a51b85b.png#align=left&display=inline&height=339&margin=%5Bobject%20Object%5D&name=image.png&originHeight=678&originWidth=2116&size=206916&status=done&style=none&width=1058)<br />不就是这玩意吗，找到对应关系了 那就ok了<br />TTF文件没办法直接搞啊 ？怎么办<br />保存成xml !<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1607128435182-e8a0b03f-6251-4a2e-a017-4587cf7829c9.png#align=left&display=inline&height=101&margin=%5Bobject%20Object%5D&originHeight=101&originWidth=343&size=0&status=done&style=none&width=343)<br />然后就成了这玩意 ok对应关系也有了 python也能搞了<br />去写一下 整逻辑就是<br />先去拿 ttf文件 url 请求url 保存 然后转xml<br />然后 拿加密前的特殊字符
```
# 就是这玩意
&#xed69;&#xe34c;&#xf149;&#xf6d6;&#xf19d;
```
然后去切割 对应 OK完事<br />对应关系的代码<br />根据看到的 id : 0啥也不说<br />从1~15 就是这些东西<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/97322/1607128528831-f2a0f153-dbf3-49b0-a25f-1039a1172330.png#align=left&display=inline&height=333&margin=%5Bobject%20Object%5D&name=image.png&originHeight=666&originWidth=2108&size=172483&status=done&style=none&width=1054)<br />上代码
```
# 对应关系 

def kuaishou_un_font(soup, font_size):
	# soup 就是传的 ttf转码成xml的
    font_dict = {}
    for font_m in soup.glyphorder.children:
        if font_m != '\n' and 'humans' not in font_m:
            id = font_m.get('id')
            name = font_m.get('name')
            if id != '0' and int(id) < 11:
                font_dict[name] = str(int(id)-1)
            elif id == '11':
                font_dict[name] = '.'
            elif id == '12':
                font_dict[name] = 'w'
            elif id == '13':
                font_dict[name] = 'k'
            elif id == '14':
                font_dict[name] = 'm'
            elif id == '15':
                font_dict[name] = '+'

    size_dict = {}
    for font_k in soup.cmap_format_4.children:
        if 'map' in str(font_k):
            code = font_k.get('code')[-4:]
            name = font_k.get('name')
            size_dict[code] = name
    return font_dict[size_dict[font_size]]


```
然后是拿TTF 文件然后转成xml
```
# TTF转XML
 font = TTFont('font_size.ttf')
  font.saveXML('font_size.xml')

```
和split后list去一一解密
```
# font_url 自己去动态拿 每次都变动
font_url = ''
font_res = requests.get(font_url)

  with open('font_size.ttf', 'wb+') as f:
      f.write(font_res.content)

  font = TTFont('font_size.ttf')
  font.saveXML('font_size.xml')
  soup = BeautifulSoup(open('font_size.xml'), 'lxml')
  try:
      fan = user_data_json['obfuseData']['fan'][40:-8].split(';&#x')
      fans = ''
      for f in fan:
          fans += kuaishou_un_font(soup, f)
  except:
      fans = ''

```
最后<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/97322/1607128560204-6cf72f87-5288-4e31-b176-81760ae1e8d3.png#align=left&display=inline&height=217&margin=%5Bobject%20Object%5D&name=image.png&originHeight=434&originWidth=1098&size=88471&status=done&style=none&width=549)<br />OK~ 解码完成 <br />
<br />——————————————————————————————————————————
<a name="9794cc28"></a>
#### TiToData：专业的短视频、直播数据接口服务平台。
<a name="1c5f89ff"></a>
#### 更多信息请联系： [TiToData](https://www.titodata.com?from=douyinarticle)
覆盖主流平台：抖音，快手，小红书，TikTok，YouTube<br />

