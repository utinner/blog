---
title: Flink初识
date: 2020-11-02 16:58:40
tags: Flink
categories: 大数据
---
<meta name="referrer" content="no-referrer" />

# 一.引入Flink的目的

- 低延迟
- 高吞吐
- 结果的准确性和良好的容错性

# 二、例子（实现wordCount）
<!-- more -->

```
package com.xesonline.demo

import org.apache.flink.api.scala.{DataSet, ExecutionEnvironment}
import org.apache.flink.api.scala._
/**
 * @Classname WordCount
 * @Description
 * @Date 2020/11/3 3:06 下午
 * @Created by jinping
 */
//批处理的wordCount
object WordCount {

  def main(args: Array[String]){
    //创建执行环境
    val env:ExecutionEnvironment= ExecutionEnvironment.getExecutionEnvironment;

    val inputPath : String = "/Users/jinping/Desktop/flink/ss-flink-scala/src/main/resources/word.txt";
    val inputDataSet : DataSet[String] = env.readTextFile(inputPath);

    //对数据进行转换处理，先分词
    val resultDataset:DataSet[(String,Int)] = inputDataSet
      .flatMap(_.split(" "))
      .map((_,1))
      .groupBy(0)//以第一个元素作为key进行分组
      .sum(1);

    resultDataset.print();
  }
}

```