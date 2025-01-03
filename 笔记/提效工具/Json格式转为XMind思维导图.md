
关于XMind转为Json、XML等格式的可以看下面这些链接
> https://github.com/zhuifengshen/xmind
> https://blog.csdn.net/weixin_33800593/article/details/88804838

但是关于Json转回XMind的轮子却没有，所以自己就写了一份

```python
# -*- coding: utf-8 -*-
import json
import xmind
import sys

def genXmindByJson(parent, data):
    if data is None:
        return
    node = parent.addSubTopic()
    for key in data:
        if key == 'title':
            print(data['title'])
            node.setTitle(data['title'])
        elif key == 'link':
            node.setURLHyperlink(data['link'])
        elif key == 'labels':
            node.addLabel(data['labels'][0])
        elif key == 'topic':
            node.setTitle(data['title'])
        elif key == 'topics':
            if isinstance(data['topics'], dict):
                genXmindByJson(node, data['topics'])
            elif isinstance(data['topics'], list):
                for i in range(len(data['topics'])):
                    genXmindByJson(node, data['topics'][i])
            else:
                print("其它类型")


def genJsonData(filename):
    with open(filename,'r', encoding='UTF-8') as f:
        data = json.load(f)
        return data

def genXmind(input):
    workbook = xmind.load("temp.xmind")
    sheet = workbook.getPrimarySheet()
    root = sheet.getRootTopic()
    root.setStructureClass = "org.xmind.ui.logic.right"
    genXmindByJson(root, genJsonData(input+'.json'))
    xmind.save(workbook, path=input+'.xmind')


if __name__ == '__main__':
    genXmind(sys.argv[1])
    
