---
date : 2021-09-01T16:21:57+08:00
draft : false
title : 'Web 前端'
author : '木村凉太'
categories : ['Web']
hiddenFromHomePage : true 
---

# Web 前端

# Web 前端

* HTML ：Web 主要结构及内容
* CSS ：Web 主要表现及样式
* Javascript：Web 主要行为及交互

## HTML

### HTML 的概念

**HTML ：** 是一种超文本编辑语言。

**标志 ：** 一般为成对出现的尖角号。例如：`<span>  </span>`

扩充：

**网页 ：** 本质是一个文件，后缀为 htm 或 html ;

**浏览器：** 解析网页文件的软件

### HTML 的基本结构

```html
<!DOCTYPE html><!-- 文档类型声明，告知浏览器使用那种HTML版本解析 -->
<html><!-- 文档开始标记 -->
	<head><!-- 头部，不会在浏览器的文档窗口显示，解析其他 -->
		<meta charset="utf-8">
		<title></title><!-- 标题 -->
	</head>
	<body><!-- 网页主要内容显示 -->
	</body>
</html>

```

### HTML 的部分标签

```html
<!DOCTYPE html>
<html>
	<head>
		<!-- 指定编码格式 -->
		<meta charset="utf-8">
		<!-- 搜索后的描述信息-->
		<meta name="descriprion" content="这是搜索后的小段描述信息">
		<!-- 搜索后的关键词 -->
		<meta name="keywords" content="描述，关键词" />
		<!-- X秒后跳转到其他网页 -->
		<meta http-equiv="refresh" content="5;url=http://www.mucunliangtai.com">
		<title>网页标题</title>

		<!-- 添加收藏夹图标-->
		<link rel='shortcut icon' href="XX.ico" />
	</head>
	<body>
		<!-- div体现页面布局-->
		<div align="center">
			<!-- width 一般不用固定尺寸用百分比动态调整 -->
			<img src="XX.png" width="300" title="图片名字" alt="加载失败时，显示的文字" />
		</div>

		<!-- 列表标签 -->
		<!-- 无序列表 type 决定了无序列表前面的小圆圈-->
		<ul type="circle">
			<li>无序列表1</li>
			<li>无序列表2</li>
		</ul>
		<!-- 有序列表 -->
		<ol type="1">
			<li>无序列表1</li>
			<li>无序列表2</li>
		</ol>
		<!-- 自定义列表 -->
		<dl>
			<dt>Q1</dt>
			<dd>A1</dd>
			<dt>Q2</dt>
			<dd>A2</dd>
		</dl>

		<!-- 表格标签 -->
		<table border="1" cellpadding="20" cellspacing="5">
			<caption> 表格标题 </caption>
			<thead>
				<tr>
					<th colspan="2">H5写法 横向合并</th>
				</tr>
			</thead>
			<tbody>
				<tr>
					<td>其他写法</td>
					<td rowspan="2">纵向合并</td>
				</tr>
				<tr>
					<td>其他写法</td>
				</tr>
			</tbody>

		</table>
	</body>
</html>

```

效果展示：

![请输入图片描述](./images/2022/01/4152653597.jpg)

5 秒后跳转到对应网站

### HTML 表单

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title>表单标签</title>
	</head>
	<body>
		<h3>注册页面</h3>
		<!-- 明文用get，有密码用post -->
		<form action="http://mucunliangtai.com" method="get">
			<!-- action='#'表示当前页面 -->
			<p>用户名：<input type="text" name="user" placeholder="木村凉太" /></p>
			<p>密码：<input type="password" name="pwd" /></p>
			<p>爱好：
				<input type="checkbox" name="hobby" value="0" checked="checked" />读书
				<input type="checkbox" name="hobby" value="1" />游戏
				<input type="checkbox" name="hobby" value="2" />运动
			</p>
			<p>性别：
				<input type="radio" name="gender" value="0" checked="checked" />男
				<input type="radio" name="gender" value="1" />女
			</p>
			<p>头像：<input type="file" /></p>
			<p>生日：<input type="date" /></p>
			<p>
				<input type="button" value="无用的按钮">
				<input type="reset" />
				<input type="submit" />
			</p>
			<p>一线城市：
				<select name="province" size="1">
					<option value="sz">深圳</option>
					<option value="gz">广州</option>
					<option value="bj">北京</option>
					<option value="sh">上海</option>

				</select>
			</p>
			<p>
				<textarea name="profile" cols="20" rows="5" placeholder="介绍"></textarea>
			</p>

		</form>
	</body>
</html>

```

效果展示：

![请输入图片描述](./images/2022/01/3340414582.jpg)

### 超链接标签

**超链接：** 指向一个目标的连接关系

**链接其他文件/网页等：**

`<a href='http://mucunliangtai.com'> </a>`

**链接本网页的其他位置：**(例：返回顶部)

`<a href='#id'> </a>`

id 为标签的对应 id ，一般需要自己定义。

**链接打开新的标签页：**

`<a href = 'http://mucunliangtai.com' target='_blank'>`