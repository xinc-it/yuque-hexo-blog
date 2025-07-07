---
title: Buttterfly美化教程集合
date: '2021/12/20 22:06:26'
updated: '2025-07-06 11:27:08'
categories: Butterfly
tags:
  - hexo
  - Butterfly
---
# 1. 设置背景渐变效果
## 1.效果图
![](/images/1ce129e52d2f6fa640b2dda888b6d2aa.png)



## 2.步骤


1. 在博客根目录下的`themes\butterfly\source\css`创建一个css后缀文件
2. 在新建的后缀文件中添加如下配置。 由于每个人用的butterfly 版本不同，所以修改的不一定是#body-wrap。具体需要通过在页面上使用`F12`，查看标签对应的id。![](/images/f55339dcccefd5f7c58b5080f049cc70.png)

```css
/* 修改文章页背景及透明度 */
#body-wrap {
  background: -webkit-linear-gradient(
    0deg,
    rgba(247, 149, 51, 0.1) 0,
    rgba(243, 112, 85, 0.1) 15%,
    rgba(239, 78, 123, 0.1) 30%,
    rgba(161, 102, 171, 0.1) 44%,
    rgba(80, 115, 184, 0.1) 58%,
    rgba(16, 152, 173, 0.1) 72%,
    rgba(7, 179, 155, 0.1) 86%,
    rgba(109, 186, 130, 0.1) 100%
  );
  background: -moz-linear-gradient(
    0deg,
    rgba(247, 149, 51, 0.1) 0,
    rgba(243, 112, 85, 0.1) 15%,
    rgba(239, 78, 123, 0.1) 30%,
    rgba(161, 102, 171, 0.1) 44%,
    rgba(80, 115, 184, 0.1) 58%,
    rgba(16, 152, 173, 0.1) 72%,
    rgba(7, 179, 155, 0.1) 86%,
    rgba(109, 186, 130, 0.1) 100%
  );
  background: -o-linear-gradient(
    0deg,
    rgba(247, 149, 51, 0.1) 0,
    rgba(243, 112, 85, 0.1) 15%,
    rgba(239, 78, 123, 0.1) 30%,
    rgba(161, 102, 171, 0.1) 44%,
    rgba(80, 115, 184, 0.1) 58%,
    rgba(16, 152, 173, 0.1) 72%,
    rgba(7, 179, 155, 0.1) 86%,
    rgba(109, 186, 130, 0.1) 100%
  );
  background: -ms-linear-gradient(
    0deg,
    rgba(247, 149, 51, 0.1) 0,
    rgba(243, 112, 85, 0.1) 15%,
    rgba(239, 78, 123, 0.1) 30%,
    rgba(161, 102, 171, 0.1) 44%,
    rgba(80, 115, 184, 0.1) 58%,
    rgba(16, 152, 173, 0.1) 72%,
    rgba(7, 179, 155, 0.1) 86%,
    rgba(109, 186, 130, 0.1) 100%
  );
  background: linear-gradient(
    90deg,
    rgba(247, 149, 51, 0.1) 0,
    rgba(243, 112, 85, 0.1) 15%,
    rgba(239, 78, 123, 0.1) 30%,
    rgba(161, 102, 171, 0.1) 44%,
    rgba(80, 115, 184, 0.1) 58%,
    rgba(16, 152, 173, 0.1) 72%,
    rgba(7, 179, 155, 0.1) 86%,
    rgba(109, 186, 130, 0.1) 100%
  );
}

```





# 2. 页脚渐变透明
## 1. 效果图
![](/images/94decede6f8e35f4e26e1ab04ce9e8b9.png)



## 2. 步骤
1. 在博客根目录下的`themes\butterfly\source\css`创建一个css后缀文件
2. 在新建的后缀文件中添加如下配置

```css
/* 页脚透明渐变 */
#footer {
    background: rgba(255,255,255,.15);
    color: #000;
    border-top-right-radius: 20px;
    border-top-left-radius: 20px;
    backdrop-filter: saturate(100%) blur(5px)
}

#footer::before {
    background: rgba(255,255,255,.15)
}

#footer #footer-wrap {
    color: var(--font-color)
}

#footer #footer-wrap a {
    color: var(--font-color)
}
```







# 3. 页脚徽标设置
## 1. 效果图
![](/images/6178258757ec9b06967a9ce88097b945.png)



## 2.步骤
在_cofing.yml中添加如下配置

```yaml
footer:
  custom_text: <p><a style="margin-inline:5px"target="_blank" href="https://hexo.io/"><img src="https://img.shields.io/badge/Frame-Hexo-blue?style=flat&logo=hexo" title="博客框架为 Hexo" alt="HEXO"></a><a style="margin-inline:5px"target="_blank" href="https://butterfly.js.org/"><img src="https://img.shields.io/badge/Theme-Butterfly-6513df?style=flat&logo=bitdefender" title="主题采用 Butterfly" alt="Butterfly"></a><a style="margin-inline:5px"target="_blank" href="https://www.jsdelivr.com/"><img src="https://img.shields.io/badge/CDN-jsDelivr-orange?style=flat&logo=jsDelivr" title="本站使用 Jsdelivr 为静态资源提供CDN加速" alt="Jsdelivr"></a><a style="margin-inline:5px"target="_blank" href="https://github.com/"><img src="https://img.shields.io/badge/Source-Github-d021d6?style=flat&logo=GitHub" title="本站项目由 GitHub 托管" alt="GitHub"></a><a style="margin-inline:5px"target="_blank"href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img src="https://img.shields.io/badge/Copyright-BY--NC--SA%204.0-d42328?style=flat&logo=Claris" alt="img" title="本站采用知识共享署名-非商业性使用-相同方式共享4.0国际许可协议进行许可"></a></p> 
```







# 4. 图片懒加载


## 1. 步骤
1. 安装懒加载插件

```bash
npm install hexo-lazyload-image --save
```

2. 在配置文件对应主题.yml中添加配置

```bash
lazyload:
  enable: true 
  onlypost: false  # 是否只对文章的图片做懒加载
  loadingImg: # eg ./images/loading.gif
```

[  
  
](http://img.wuxin.website/img/b3f6b2565c3149a2a8df94ab43cf3a3b.jpg)

