---
layout: post
title: "Shell+Ruby"
date: 2017-05-28 20:25:34 +0800
comments: true
categories: shell ruby automation
---


最近公司产品的新特性开发的不是那么多了，所以自己有时间闲下来偿还一下以前欠下的技术债。公司的发版流程是自己一步步实现出来的，整体结构是由一个个的shell脚本来实现的。把发版的每一个步骤拆分为独立的一个个脚本，这样流程变化了可以通过脚本组合的变化来适应变化。毕竟能够维持不变的东西永远是**变化**。

以前的问题是由于当时开发的紧迫性，加上自己对脚本的不太重视，导致开发的这些脚本整体结构还算好，但是就是里面会有很多重复的常量在里面。这里的常量就是要发版的各个组件的名字，由于目前产品组件的不确定性，经常会添加或者删除一个组件，目前的缺点就是添加或者删除一个组件的时候需要更改多个脚本，违背了SPOT原则，会很容易遗落下本应同步修改的脚本。

这次自己有时间了决定把这些常量提取出去作为一个配置文件单独存在，思路是简单而直接的，但是由于shell脚本的局限性，把组件名字提出去还算可行，但是我想要对每个组件加上单独的其他配置的话好像比较绕。以以下脚本为例来说：
{% codeblock lang:bash%}
for repo in "COM_A COM_B COM_C"
do
  cd $repo
  if [ $repo == "COM_B"
    cd modules
  fi
  mvn setVersion=$newVersion
done
{% endcodeblock %}
这个脚本是迭代每个组件，然后去修改pom.xml里面的version改为要发版的version，问题有的组件的parent的pom.xml没在根目录下，需要切换到对应的目录才能进行操作。

我们可以很容易就把 "COM_A COM_B COM_C" 提取出去， 例如：
{% codeblock lang:bash%}
export all_repos="COM_A COM_B COM_C"
{% endcodeblock %}

这样脚本就变成了
{% codeblock lang:bash%}
for repo in $all_repos
do
  cd $repo
  if [ $repo == "oneRepo"
    cd modules
  fi
  mvn versions:set -DnewVersion=$newVersion
done
{% endcodeblock %}

问题是这个modules目录怎么配置出去，想过这样去做：
{% codeblock lang:bash%}
export COM_B_pom_path=modules
{% endcodeblock %}

但是问题是这只是一个配置而已，因为发版流程还涉及到其他的目录，像我们的组件用的Chef的cookbook来配置安装，所以我还要配置这个cookbook的路径，然后我们的cookbook用的test kitchen来测试，所以还要配置test kitchen的测试文件所在的路径，这样会有很多export，可读性和可维护性都很差。

根据以往的经验，所有复杂的问题理论上都可以经过抽象或者添加中间层的方式来简化实现。这里我想到了用Ruby来生成shell需要的export全局变量，然后既然用到Ruby了很自然的配置文件就用到了YAML。这种配置文件既简洁又保持了很好的可读性。下面就是就是yaml关于这次修改的部分样例：
{% codeblock lang:ruby%}
---
components
  COM_A
  	pom: /
  	cookbook: cookbook
  	test_kitchen: /
  COM_B
    pom: modules
    cookbook: cookbook
  	test_kitchen: /
  COM_C
  	pom: /
  	cookbook: cookbook
  	test_kitchen: /
...
{% endcodeblock %}

这样我们可以很容易的实现一段Ruby代码来生成这个shell的环境文件：
{% codeblock lang:yaml%}
require "yaml"


def all_repo_list(yaml) 
	return yaml.keys.join(" ")
end

def all_cookbook_dirs(yaml)
	all_dir_for(yaml, "cookbook")
end

def all_pom_dirs(yaml)
	all_dir_for(yaml, "pom")
end

def all_test_kitchen_dirs(yaml)
	all_dir_for(yaml, "test_kitchen")
end

def all_dir_for(yaml, type)
	dirs = []
	yaml.each do |k, v|
		next if v.nil? || v[type].nil?
		subfolder = v[type]
		dirs << "#{k}#{subfolder}"
	end
	dirs.join(" ")
end


current_dir = File.dirname(__FILE__)
yaml = YAML.load_file("#{current_dir}/config.yml")
all_components = yaml["components"]

text = []
text << "export all_repos=\"#{all_repo_list(all_components)}\""
text << "export all_cookbook_dirs=\"#{all_cookbook_dirs(all_components)}\""
text << "export all_pom_dirs=\"#{all_pom_dirs(all_components)}\""
text << "export all_test_kitchen_dirs=\"#{all_test_kitchen_dirs(all_components)}\""


File.open("#{current_dir}/config", "w") {|file| file.write(text.join("\n")) }	
{% endcodeblock %}

然后在脚本执行前source一下这个生成的文件**config**即可。