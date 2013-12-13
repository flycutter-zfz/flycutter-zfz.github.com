---
layout: post
title: Mahout NaiveBayes中文分类
description: 使用mahout的NaiveBayes实现中文分类
category: "tools"
tags: [mahout]
---

###1. 简介
mahout集成了好多分类、聚类、推荐算法，这些算法大部分都可以在hadoop上运行。

###2. 环境
mahout0.8.2 + hadoop1.0.3


###3. 步骤

####(1) Sequencing
将训练样本从文本形式，转换为hadoop的sequenceFile的形式
####(2) Vectorize
向量化，将文章转换为向量的形式，本文里用的是分词，然后每个词的词频
####(3) Split
将样本分为训练集和测试集
####(4) Train
根据训练集训练出模型
####(5) Test
使用测试集测试模型，并且得出准确率
####(6) Classify
使用模型对新样本进行分类

###4. 详细步骤
####1. Sequencing
{% highlight java %}
//sequence files in a directory
public void sequenceFiles()throws Exception{
	/**
	* -i: 指定输入文件夹
	* sampleDir: 输入文件夹,结构为：每个类是一个子文件夹，然后每个类的文件夹内包含这个类的实例（比如新闻）
	* -o: 指定输出文件位置，此处值为sequenceDir
	*/
	String[] options = new String[]{"-i", sampleDir, "-o", sequenceDir};
	SequenceFilesFromDirectory sequence = new SequenceFilesFromDirectory();
	sequence.setConf(this.conf);
	ToolRunner.run(sequence, options);
}
{% endhighlight %}

####2. Vectorize

{% highlight java %}
//vector the sequence files
public void vectorize()throws Exception{
	/**
	* -i: 指定输入文件夹，此处值为sequenceDir，也即上一阶段的输出
	* -o: 指定向量化后的文件路径，此处值为vectorDir
	* -lnorm: 指用对数来normalize向量
	* -nv: 指定向量需要命名
	* -wt: 指定权重方法来tf或idf
	* -s: 指定最小支持数量（即只考虑出现指定次数或者以上的词）
	* -a: 指定分词方法，此处用庖丁分词
	*/
	String[] options = new String[]{"-i",sequenceDir, "-o", vectorDir, "-lnorm", "-nv","-wt","tfidf","-s","2","-a","net.paoding.analysis.analyzer.PaodingAnalyzer"};
	SparseVectorsFromSequenceFiles vectorizeJob = new SparseVectorsFromSequenceFiles();
	vectorizeJob.setConf(this.conf);
	vectorizeJob.run(options);	
}
{% endhighlight %}

####3. Split

{% highlight java %}
//split the sequence files to train data and test data
//对应：bin/mahout split
public void split()throws Exception{
	/**
	* -i: 指定tfidf的文件夹，值为：vectorDir + "/tfidf-vectors"
	* -tr: 指定训练数据集的输出文件夹
	* -te: 指定测试数据集的输出文件夹
	* -rs: 随机选择作为测试用例的数量
	* -ow: 如果输出存在，则覆盖掉；
	* -seq: 如果输入是sequenceFile格式，则需设置此项
	* -xm: 选择执行的方式，此处是非MapReduce，默认是MapReduce
	*/
	String[] options = new String[]{"-i",tfidfVectorDir,"-tr",trainVectorDir,"-te",testVectorDir, "-rs","40","-ow","-seq","-xm","sequential"};
	SplitInput splitJob = new SplitInput();
	splitJob.setConf(this.conf);
	splitJob.run(options);
}
{% endhighlight %}

####4. Train

{% highlight java %}
//train the model
//对应 bin/mahout trainnb
public void train()throws Exception{
	/**
	* -i: 指定训练集的tfidf文件夹，也就是上一部的输出
	* -o: 模型的生成位置
	* -li: labelIndex文件的生成位置，每个类对应到一个labelIndex，比如 新闻类对应到0，微博对应到1等
	* -el: 指明label从样本中提取
	* -ow: 可以覆盖文件
	*/
	String[] options = new String[]{"-i",trainVectorDir, "-o", modelDir,"-li", labelIndexDir ,"-el", "-ow"};
	TrainNaiveBayesJob trainJob = new TrainNaiveBayesJob();
	trainJob.setConf(this.conf);
	trainJob.run(options);
}
{% endhighlight %}

####5. Test
{% highlight java %}
//test the model
//对应 bin/mahout testnb
public void test()throws Exception{
	/**
	* -i: 指定测试集的tfidf文件夹
	* -m: 模型文件位置
	* -l: labelIndex文件的位置
	* -ow: 可覆盖
	* -o：测试结果的输出文件位置
	*/
	String[] options = new String[]{"-i",testVectorDir,"-m",modelDir,"-l",labelIndexDir,"-ow","-o",testingDir};
	TestNaiveBayesDriver testJob = new TestNaiveBayesDriver();
	testJob.setConf(this.conf);
	testJob.run(options);
}
{% endhighlight %}

####6. Classify
{% highlight java %}
//classify the text
public ClassifierResult classify(String text)throws Exception{
	AbstractNaiveBayesClassifier classifier = loadClassifier();
	Vector instance = buildInstance(text);   //向量化
	Vector result = classifier.classifyFull(instance);
	
	Path labelIndexPath = new Path(labelIndexDir);
	//真正分类的时候，不用每次都去读
	Map<Integer, String> labelMap = BayesUtils.readLabelIndex(this.conf, labelIndexPath); 
		
	int bestIdx = Integer.MIN_VALUE;
	double bestScore = Long.MIN_VALUE;
	HashMap<String, Double> resultMap = new HashMap<String, Double>();
	for(int i=0; i<labelMap.size();i++){
		Vector.Element element = result.getElement(i);
		resultMap.put(labelMap.get(element.index()), element.get());
		if(element.get()>bestScore){
			bestScore =element .get();
			bestIdx = element.index();
		}
	}
		
	ClassifierResult classifierResult = new ClassifierResult();
	if(bestIdx != Integer.MIN_VALUE){
		String label = labelMap.get(bestIdx);
		double score = bestScore;
		classifierResult.setLabel(label);
		classifierResult.setScore(score);
	}
	return classifierResult;
}
	
//Construct the naive bayes classifier
private AbstractNaiveBayesClassifier loadClassifier()throws IOException{
	Path modelPath = new Path(modelDir);
	NaiveBayesModel model = NaiveBayesModel.materialize(modelPath, this.conf);
	AbstractNaiveBayesClassifier classifier = new StandardNaiveBayesClassifier(model);
	return classifier;
}
	
//Construct the directory and frequency map
private void buildDirectory()throws IOException{
	if(isDicAndFreConstructed){
		//if the directory and frequence file has been constructed, return directly
		return;
	}
	Path dictionaryFilePath = new Path(dictionaryFile);
	for(Pair<Text, IntWritable> record : new SequenceFileIterable<Text, IntWritable>(dictionaryFilePath, true, this.conf)){
		dictionary.put(record.getFirst().toString()	, record.getSecond().get());
	}
		
	Path frequenceFilePath = new Path(frequenceFile);
	for(Pair<IntWritable, LongWritable> record : new SequenceFileIterable<IntWritable, LongWritable>(frequenceFilePath, true, this.conf)){
		frequency.put(record.getFirst().get(), record.getSecond().get());
	}
	isDicAndFreConstructed = true;
}
	
	
//construct the vector from text
private Vector buildInstance(String text)throws Exception{
	Vector vector = new RandomAccessSparseVector(this.dictionary.size());
	Set<String> words = new HashSet<String>();
	@SuppressWarnings("resource")
	Analyzer analyzer = new PaodingAnalyzer();
	TokenStream stream = analyzer.tokenStream("", new StringReader(text));
	CharTermAttribute termAtt = stream.addAttribute(CharTermAttribute.class);
		
	while(stream.incrementToken()){
		if(termAtt.length()>0){
			words.add(new String(termAtt.buffer(),0,termAtt.length()));
		}
	}
		
	for(String s : words){
		if(!dictionary.containsKey(s)){
			continue;
		}
		int index = dictionary.get(s);
		vector.setQuick(index, frequency.get(index));
	}
	return vector;
}
{% endhighlight %}

###5. 参考文章：

(1)[通过Mahout的核心api来实现中文文本的分类](http://zhangv.com/wordpress/archives/1781)