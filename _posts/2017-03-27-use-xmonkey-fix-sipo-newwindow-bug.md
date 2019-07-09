---
title: 使用Greasemonkey / TamperMonkey 改进专利局专利查询网站的打开新页面问题
tags: 技术 浏览器 脚本
---  

国家知识产权局专利查询网站：http://cpquery.sipo.gov.cn 在按照关键字查询到专利列表之后，点击查看“申请信息”的时候，总是悲催的在当前页面打开。这样看第二个的时候，就要回去重新查，反复多次之后，可能出口IP就被封掉了。所以使用各种Monkey提供的user script方法，重写专利局的打开函数，以解决这个问题。

<!--more-->

## Bug分析
查询列表里的【申请信息】的代码是这样的：

    <a href="javascript:jbxxAction('2012206292175');">申请信息</a>
    
然而，这个jbxxAction是这样的：

    //基本信息
    function jbxxAction(shenqingh,zhuanlilx){
        var page = new pageDefine('/txnQueryBibliographicData.do','','','new');
        page.addValue(shenqingh,"select-key:shenqingh");
        page.addValue(location.href, "select-key:backPage");
        page.goPage();
    }
    
悲催的是，在page.goPage的注释里:

    /* 打开窗口的方式:
    如果是模式窗口：type=modal、dialog
    如果是在主窗口中：type=window
    如果是新打开的窗口：type=new-window
    如果是iframe：type=subframe
    */
    
囧，那你传个new而不是new-window进去是什么概念……不过看上去，好像传了new-window也不太对，先不管了。

我试了一下，直接控制台重新写这个jbxxAction，直接构造url然后open也可以勉强用：

    function jbxxAction(xqbh){window.open('http://cpquery.sipo.gov.cn/txnQueryBibliographicData.do?select-key:shenqingh='+xqbh)}

## 安装monkey
Chrome下插件：tampermonkey
FireFox下插件：greasemonkey

然后写一个自定义脚本：

    var inject_method = document.createElement("script");
    inject_method.setAttribute("type","text/javascript");
    inject_method.appendChild(document.createTextNode("function jbxxAction(xqbh){window.open('http://cpquery.sipo.gov.cn/txnQueryBibliographicData.do?select-key:shenqingh='+xqbh)}"));
    document.body.appendChild(inject_method);

然后把网址的查询列表的url自定义到匹配里面，这样就能自动运行了。
![image](/illustration/use-xmonkey-fix-sipo-newwindow-bug.png)

## 参考：
https://stackoverflow.com/questions/3542609/replacing-page-functions-via-userscript-in-chrome/3602611#3602611
    

    

