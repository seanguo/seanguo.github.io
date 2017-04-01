---
layout: post
title: "Chef 介绍 Part1 Setup"
date: 2017-02-17 20:20:59 +0800
comments: true
categories: tool chef
keywords: Chef Tool Operation Devops
---

##引言
Chef是一个自动化部署的工具。它的思想是“Infrastructure as code”, 所以整个DevOps流程下来都是代码，都可以保存在代码仓库里面。像Chef之类的工具出现之前，简单部署的话就是手工来了，像Java的web应用手工部署个War包或者改个配置文件什么的，高级点的可能会写个shell脚本来自动化。但是当需要管理的机器增多或者配置项增多，或者不同的机器要求不一样的配置的话，就力不从心了。Chef出现之后，这样的难题就迎刃而解了。

Chef的实现是用的Ruby，所以它的配置脚本也是基于Ruby的, 所以想要学好Chef也需要了解一下Ruby的基本语法，如果想要扩展，定制，增加自己的Chef库的话，还是需要仔细学习一下Ruby才行。这导致Chef陡峭的学习曲线，了解个皮毛很容易，但是想要深入了解，诊断问题的话需要做很多功课才可以。

想要学好，我们先搭环境，在应用中学习是最有成效的。

##安装
Chef安装很简单，就去下载一个叫做[Chef DK](https://downloads.chef.io/)的包安装就可以了。这个会把Chef全家桶装好，里面开发测试用的所有工具都包含了。

安装完后可以运行下面的命令检查一下
{% codeblock lang:bash%}
➜  ~ chef-solo -v
Chef: 12.17.44
{% endcodeblock %}

##第一个程序
我们可以通过下面的命令来创建我们的第一个Chef程序(它叫Cookbook)
{% codeblock lang:ruby%}
chef generate cookbook sample1
{% endcodeblock %}

下面是这个cookbook的目录结构:
{% codeblock lang:bash%}
.
├── Berksfile
├── README.md
├── chefignore
├── metadata.rb
├── recipes
│   └── default.rb
├── spec
│   ├── spec_helper.rb
│   └── unit
│       └── recipes
│           └── default_spec.rb
├── test
│   └── smoke
│       └── default
│           └── default_test.rb


{% endcodeblock %}

简单来说Chef的运行是顺序执行一个一个的Resource，Chef在Ruby语法的基础上封装了自己的DSL来把经常用到的命令，配置文件的管理等抽象成一个个的Resource。这些Resource根据需求顺序放在Chef的脚本里（它叫Recipe）就可以执行了。所以Chef的入口是Recipe，下面是一个简单的例子。

{% codeblock lang:ruby%}
file '/tmp/test.txt' do
  content 'This is myfirst sample'
  mode '0755'
end
{% endcodeblock %}

这个主代码块我们要放到recipes/default.rb里

像这个file Resource就会在制定的路径创建一个php文件，然后填充文件的内容，设置用户和组，然后还有权限。简单而直观，没有Ruby的背景也能够轻易理解。
要写在本地运行起来，我们还要创建两个文件：
{% codeblock lang:bash%}
cat ~/.chef/solo.rb 
cookbook_path ['~/blog_samples/chef_intro/']%
{% endcodeblock %}
{% codeblock lang:bash%}
cat test.json
{
"run_list":["sample1"]
}
{% endcodeblock %}

然后可以用下面的命令在本地来运行这个cookbook:
{% codeblock lang:bash%}
chef-solo -j test.json -c ~/.chef/solo.rb 
{% endcodeblock %}

然后我们看到了如下的输出
{% codeblock lang:bash%}
Starting Chef Client, version 12.17.44
resolving cookbooks for run list: ["sample1"]
Synchronizing Cookbooks:
  - sample1 (0.1.0)
Installing Cookbook Gems:
Compiling Cookbooks...
Converging 1 resources
Recipe: sample1::default
  * file[/tmp/test.txt] action create
    - create new file /tmp/test.txt
    - update content in file /tmp/test.txt from none to ea8447
    --- /tmp/test.txt	2017-02-23 21:20:46.000000000 +0800
    +++ /tmp/.chef-test20170223-2804-c8bnol.txt	2017-02-23 21:20:46.000000000 +0800
    @@ -1 +1,2 @@
    +This is myfirst sample
    - change mode from '' to '0755'

Running handlers:
Running handlers complete
Chef Client finished, 1/1 resources updated in 12 seconds
{% endcodeblock %}

这表示成功执行完了，可以到那个地方检查一下这个文件是否被创建了。
{% codeblock lang:bash%}
cat /tmp/test.txt 
This is myfirst sample
{% endcodeblock %}     
到这之后然后可能会想要修改文件的mode， 这可以通过Attribute来实现。

{% codeblock lang:ruby%}
file '/tmp/test.txt' do
  content 'This is myfirst sample'
  mode node['chef_intro']['sample1']['file_mode']
end
{% endcodeblock %}
这些Attribute是在另一个Ruby文件（attributes/default.rb）定义的：
{% codeblock lang:ruby%}
  default['chef_intro']['sample1']['file_mode'] = '755'
{% endcodeblock %}
然后修改我们的用户配置test.json为:
{% codeblock lang:bash%}
{
  "run_list":["sample1"],
  "chef_intro": {
  	"sample1": {
  		"file_mode": "644"
  	}
  }
 {% endcodeblock %}
 再次运行这个cookbook可以看到输出已经把文件的mode改成644了，通过这种方式我们就可以通过从外面的配置覆盖cookbook的默认值，达到定制化配置的目的。
 {% codeblock lang:bash%}
 Recipe: sample1::default
  * file[/tmp/test.txt] action create
    - change mode from '0755' to '0644'
 {% endcodeblock %}
