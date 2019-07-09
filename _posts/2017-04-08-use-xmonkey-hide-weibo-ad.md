---
title: 使用 TamperMonkey 隐藏新浪微博广告
tags: 技术 浏览器脚本
--- 

微博的时间线混乱已经为人诟病很久，在时间线上插入的广告起乌七八糟的内容，更是令人作呕。尝试借助*Monkey，先把WEB的广告干掉。（Chrome插件：tampermonkey / FireFox插件：greasemonkey）

<!--more-->

### 实现思路
#### 隐藏广告牛皮癣
幸亏在微博的时间流中的牛皮癣有个特别的标记，这个div有个属性：
```feedtype="ad"``` ，就靠这个属性，然后hide掉所有有这个属性div就好了。

#### 处理自动加载的微博
WEB版的微博在滚动到下部之后，会触发加载更多微博的事件，动态的加载更多的微博。<del>目前简单的解决方法是勾住页面滚动事件，滚一下鼠标就触发清理牛皮癣广告的方法</del> 还是换成setInerval每10秒清理一下页面上的广告（2017.12.04）。

### 完整代码如下：
目前我的环境是：MacOS/Safari/Tampermonkey

	// ==UserScript==
	// @name         Fuck Weibo.AD
	// @namespace    http://topcaver.com/
	// @version      0.1
	// @description  try to fuck weibo.ad off!
	// @author       TopCaver
	// @match        https://weibo.com/*
	// @grant        none
	// ==/UserScript==
	
	(function() {
	    'use strict';
	
	    // Your code here...
	    var clear_ad_func = function(){
	        var ad_div_list = document.querySelectorAll('div[feedtype="ad"]');
	        for(var i=0; i < ad_div_list.length; i++){
	            ad_div_list[i].hidden=true;
	        }
	    };
	    setInterval(clear_ad_func,10000); //10s定时清理广告
	})();



