---
layout: post
title: "karma配置中的一些问题"
date: 2013-12-19 23:31
comments: true
categories: 
---
之前看了[icodeit](www.icodeit.org)的博客[使用Karma运行JavaScript测试](http://icodeit.org/2013/10/using-karma-as-the-javascript-test-runner/)，试了一下karma，果然是非常好用。

关于使用karma基础流程，就请参照那篇博客。

这里主要针对我在配置karma的时候遇到的一些特殊问题进行一个总结。

##使用karma-html-reporter配置生成的html测试报告
####配置
`package.json`中的内容：
    
    {
	  "devDependencies": {
    	"karma": "~0.10.8",
	    "karma-html-reporter": "~0.1.1"
	  }
	}

执行`npm install`进行plugin的安装。

使用`./node_modules/.bin/karma init`初始化karma配置文件

然后打开`karma.conf.js`配置文件，添加html格式的report：

	reporters: ['progress', 'html'],
	
    htmlReporter: {
      outputDir: 'report',
      templatePath: __dirname + '/jasmine_template.html'
    },

####问题
然而这时候启动karma
	
	./node_modules/.bin/karma start karma.conf.js
	
会报错：

	ERROR [karma]: { [Error: ENOENT, open '/Users/twer/learn/jasmine/jasmine_template.html']
	  errno: 34,
	  code: 'ENOENT',
	  path: '/Users/twer/learn/jasmine/jasmine_template.html' }
	Error: ENOENT, open '/Users/twer/learn/jasmine/jasmine_template.html'
	
#####解决
由于karma-html-reporter为了保持生成报告的灵活性，所以在生成html的时候使用了template。也就是配置中的这一句：

      templatePath: __dirname + '/jasmine_template.html'

但是配置好的时候我们是没有自己的template	的，那要怎么办呢？
其实如果进入`node_modules/karma-html-reporter/`目录看一下的话，会发现这个目录下有一个叫`jasmine_template.html`的html文件。那如果我们将templatePath指向这个文件的话，会怎么样呢？

      templatePath: 'node_modules/karma-html-reporter/jasmine_template.html'
      
然后重启karma服务器
	
	./node_modules/.bin/karma start karma.conf.js
	
会发现已经没有报错了，并且在`report`目录下，生成了一个新的目录，比如我的是`Chrome 31.0.1650 (Mac OS X 10.7.5)`。在这个目录下就会有我们生成的html报告了，打开看一下，会发现他的样式跟jasmine的SpecRunnerHtml结果是很像的，除了不能点链接。

到这里karma-html-reporter的配置就完成了。

###在karma里借助jasmine-jquery来使用external fixtures
jasmine-jquery由于支持多种fixture的实现，所以被广泛的使用在jasmine测试中。
但是在karma中使用jasmine-jquery会存在一些配置的问题。
####配置
首先要把jasmine-jquery的库文件放在files配置项里：

    files: [
      'src/**/*.js',
      'test/*.js',
      'test/lib/jquery-1.10.2.min.js',
      'test/lib/jasmine-fixture.min.js',
      'test/lib/jasmine-jquery.js'
    ],

在jasmine测试中使用jasmine-jquery的loadFixtures方法：

    jasmine.getFixtures().fixturesPath = path + 'fixtures';
    loadFixtures('test_fixtures.html');

####问题
那么在启动karma的时候会出现这样的错误：

	Chrome 31.0.1650 (Mac OS X 10.7.5) Firefox fixtures should work with imported html with jasmine-jquery FAILED
	Error: Fixture could not be loaded: base/test/fixtures/test_fixtures.html (status: error, message: undefined)
	    at Object.$.ajax.error (/Users/twer/learn/jasmine/test/lib/jasmine-jquery.js:130:17)
	    at c (/Users/twer/learn/jasmine/test/lib/jquery-1.10.2.min.js:4:26036)
	    at Object.p.fireWith (/Users/twer/learn/jasmine/test/lib/jquery-1.10.2.min.js:4:26840)
	    at k (/Users/twer/learn/jasmine/test/lib/jquery-1.10.2.min.js:6:14283)
	    at r (/Users/twer/learn/jasmine/test/lib/jquery-1.10.2.min.js:6:18646)
	    at Object.send (/Users/twer/learn/jasmine/test/lib/jquery-1.10.2.min.js:6:18771)
	    at Function.x.extend.ajax (/Users/twer/learn/jasmine/test/lib/jquery-1.10.2.min.js:6:13722)
	    at jasmine.Fixtures.loadFixtureIntoCache_ (/Users/twer/learn/jasmine/test/lib/jasmine-jquery.js:122:21)
	    at jasmine.Fixtures.getFixtureHtml_ (/Users/twer/learn/jasmine/test/lib/jasmine-jquery.js:114:12)
	    at jasmine.Fixtures.read (/Users/twer/learn/jasmine/test/lib/jasmine-jquery.js:77:28)
	 
####尝试   
看了一下jasmine-jquery的源码，发现在loadFixtures的时候调用了jquery里面的ajax方法。也就是说，当运行这段代码的时候，js会从浏览器中取出这个html文件，而不是从本地硬盘读取。

而karma的原理是借助于node.js启动了一个服务器，那么如果这个html文件不在服务器上的话，js是无法从浏览器里取出这个文件的。

那么我们首先让这个文件能在服务器上出现，把html文件假如files配置项：

    files: [
      'src/**/*.js',
      'test/*.js',
      'test/lib/jquery-1.10.2.min.js',
      'test/lib/jasmine-fixture.min.js',
      'test/lib/jasmine-jquery.js',
      {
        pattern: 'test/fixtures/**/*.html',
        watched: false,
        included: false
      }
    ],

这里稍微解释一下定义格式：每个定义的文件pattern都由三项属性：

* watched, 文件的修改是否被监控；
* included, 文件是否是测试代码；
* served，文件是否出现在服务器上。

对于前面的字符串定义，其实是默认设置这三个属性为true的定义。那么其实我们的hmtl文件的修改不需要被监控，本身也不是测试代码，所以可以使用完整定义将这两项定义为false，只有文件是否出现在服务器上为true。

那么现在这个html文件应该可以被karma加载在服务器上了吧，重新运行karma，发现错误依旧。

####解决
后来进行了大量搜索，最后在这个karma的[github issue](https://github.com/karma-runner/karma/issues/788)里找到了答案。

所谓karma-html2js-preprocessor，是一个预处理器，会将files配置项中的html文件都转化成js文件，就拿karma-html2js-preprocessor主页上的一个例子来说：

有一个`test.html`，内容为

	<div>something</div>

会被转化为一个js文件`template.html.js`，内容为

	window.__html__ = window.__html__ || {};
	window.__html__['template.html'] = '<div>something</div>';

而且这个preprocessor会被默认加载，所以其实我们在前面写的pattern中的html文件已经全部变成js文件被karma加载了，这也是html文件在服务器上不能被找到的原因。

那么怎么让preprocessor不做这样的转化呢？我们需要在增加一条配置：

    preprocessors: {
      '**/*.html': []
    },

然后再重启karma服务器，就会发现一切正常了。