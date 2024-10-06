---
需求: "#745102 【IM 模块】第一期群基础功能和消息类型搭建 https://icc.pm.netease.com/v6/issues/745102"
需求文档: https://docs.popo.netease.com/team/pc/east/pageDetail/7ecbeb785381429d93f754745be2e8dd
交互: https://g.126.fm/00e3dXc
UI:
  - https://g.126.fm/03l7J7Q
wiki:
  - https://km.netease.com/team/cc_studio/wiki/page/418612
排期:
  - https://url.163.com/HfzY
日志: 
耗时: 
备注: 群基础功能和消息类型搭建
分支: 
publishTime: 2024-01-22
---
---



一、公告逻辑

未加群

- 显示加群提示
    - 点击加群
        - 加群后显示群公告
    - 未点击加群
        - 不移除

已加群

- 显示过且与上次一致，则不再显示
- 未显示或与上次不一致

    - 显示最新群公告
        - 点击跳转，群公告消失
    - 更新群公告
        - 重复上述流程

未加群显示加群提示，加群提示可否关闭？

未加群可否显示其它消息，如群公告，活动消息？

消息间是否有优先级？



#### 二、[[保存图片]]


```
com.netease.cc.js.functions.PublicWebFunction#saveImageByBase64

MediaScannerConnection.scanFile(ApplicationWrapper.getContext(), new String[]{imageFile.getAbsolutePath()}, null, null);
```

保存图片后要用 MediaScannerConnection 来通知系统已添加新图像

保存图片时需要留意命名，避免触发截屏
`com.netease.cc.util.screenshot.IScreenshotWatcherImpl`

