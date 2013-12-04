---
layout: post
title: 中文分词-庖丁分词
description: lucene中文分词插件
category: "tools"
tags: [机器学习]
---

##1. 概述
&nbsp;&nbsp;&nbsp;&nbsp;最近因为实验室项目的需要，要使用mahout进行分类，在处理中文文档的时候又需要进行分词等处理，所以就发现了[庖丁分词](https://code.google.com/p/paoding/)这个不错的工具。但是这个工具的文档不是很多，下面简单写一下使用方法。

##2. 使用方法

###(1)代码
{% highlight java %}
import java.io.StringReader;
import net.paoding.analysis.analyzer.PaodingAnalyzer;
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.TokenStream;
import org.apache.lucene.analysis.tokenattributes.TermAttribute;


public class ChineseAnalyzer {
	public static void main(String[] args)throws Exception{
		String strZH = "核心提示，据媒体报道：" +
				"美国正在计划大规模扩张在亚洲的导弹防御系统。" +
				"8月23日，美国国务院回应称，此举意在抵御来自朝鲜的导弹威胁，" + 
				"而不是针对中国。美国表示，美国通过美中军事对话以及美中战略与" + 
				"经济对话等机制，已经就该导弹防御系统的意图与中国进行了广泛的对话。";
		
		Analyzer analyzer = new PaodingAnalyzer();
		showAnalyzer(analyzer, strZH);
		
	}
	
	public static void showAnalyzer(Analyzer analyzer, String str)throws Exception{
		long start = System.currentTimeMillis();
		System.out.println("\n" + analyzer.getClass().getSimpleName());
		StringReader reader = new StringReader(str);
		TokenStream ts = analyzer.tokenStream("", reader);
		TermAttribute termAttribute = ts.getAttribute(TermAttribute.class);
		
		long end = System.currentTimeMillis();
		long time = end - start;
		System.out.println("耗时：" + time);
		
		while(ts.incrementToken()){
			System.out.println(termAttribute.term() + "|");
		}
	}
}
{% endhighlight %}
###(2)jar包

&nbsp;&nbsp;&nbsp;&nbsp;需要依赖lucene的jar包：lucene-analyzers-3.0.3.jar，lucene-core-3.0.3.jar，lucene-highlighter-3.0.3.jar；直接使用庖丁分词的paoding-analysis-2.0.4-beta.zip中的jar包会抛异常：Exception in thread "main" java.lang.AbstractMethodError: org.apache.lucene.analysis.TokenStream.incrementToken()。通过svn check下代码后，自己ant一下，得到的jar包是可以用的。


###(3)环境变量配置
&nbsp;&nbsp;&nbsp;&nbsp;庖丁分词需要依赖词典进行分词，所以要配置词典的路径，一般配置环境变量PAODING_DIC_HOME=E:\paoding-dic(词典路径)就可以了，词典可以从svn check下来的代码里找到，就是dic文件夹，还有另一种配置方法，不过我还是倾向于用环境变量，简单，用到的时候再去查另外一种用法吧。


## 参考资料

(1)[Lucene学习笔记(5)分词](http://blog.csdn.net/authorzhh/article/details/7904560)

(2)[Lucene中文引擎，庖丁解牛的词典参数配置方法](http://www.360doc.com/content/11/1122/15/1007797_166478396.shtml)
