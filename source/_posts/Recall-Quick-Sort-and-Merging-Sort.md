---
title: '温故:快排和归并'
date: 2017-09-14 21:37:04
toc: true
tags:
 - 温故
---

可能要去实习了，笔试的常考的算法要看一下，今天无聊的时候随便写了几个。

# 快速排序

  这个应该是一个要背出来的的算法了随便用scala实现了一下，快排的话是英国人霍尔在26岁的时候偶然发现的，当时他为了解决词典排序问题，想到了冒泡排序这种比较普遍的方法，然后就发现了霍尔划分法，20年之后就得了图灵奖，真是我辈楷模，所以快排又被看作是特殊的冒泡，也就是这个道理了。

  我们上面提到的并不是霍尔发现了快排，而是霍尔发现了霍尔划分法，实际上快排最主要的部分是划分，而不是排序或者合并，有两种划分方法，分别是Lomuto划分，Hoare划分，其中前者常常被用来实现基于减治的快速查找算法。但两者都可以实现快排。

## Lomuto划分

  先用1行代码写一个交换函数，然后，额....这里我犯了小错误，因为我当时考虑用基类实现数字划分的时候的时候发现这AnyVal这个东西没有比较函数，所以我为了偷懒就用了一下Int直接写了，实际只花费了5行代码。

```scala
//读取一个Array和要交换的两个元素的位置
def swap[T](l:Array[T],a:Int,b:Int):Unit={val temp=l(a);l(a)=l(b);l(b)=temp}

//实现Lomuto划分
def lomutoPartion(buf:Array[Int],start:Int,end:Int)={
  var (p,s)=(buf(start),start);
  //s指向大于p的第一个数的前一个数，一旦发现小于p的数就与s指向的位置交换，当然还有一种情况是不存在大于p的数，最后p被换到最后
  for(i<-s+1 to end) if(buf(i)<p){s+=1;swap(buf,s,i)}
  //返回中轴s的位置
  swap(buf,start,s);s
}
```

<!--more-->

### 快速查找

Lomuto划分算法常常用来做减值法的快速查找,这种查找法和在一个已经排序的数组里二分找某个元素的位置不同，这个查找是从一个无序数组里找到第n大或者第n小的元素。

```scala
//找第n大的元素
def quickSelect(l:Array[Int],start:Int,end:Int,i:Int):Int={
    val s=lomutoPartion(l,start,end);
   //如果第k大的元素在前半部分，那么后半部分和中轴被丢弃的元素是end+1-s个，所以前半部分应该找第i-(end+1-s)大的元素
    s+i-1-end match{
    //如果正好是中轴直接返回
    case 0=>l(s)
    //x>0时，表示元素在前半部分
    case x=>x>0 match{
      case true =>{quickSelect(l,start,s-1, x)}
      case false => {quickSelect(l,s+1,end,i)}
    }
    }
}


//找第n小的元素
def quickSelectLittle(l:Array[Int],start:Int,end:Int,i:Int):Int={
    val s=lomutoPartion(l,start,end);
   //如果第k大的元素在后半部分，那么后前部分和中轴被丢弃的元素是s+1-start个，所以前后部分应该找第start+i-s+1小的元素
    start+i-1-s match{
      //如果正好是中轴直接返回
      case 0=>l(s)
      //x>0时，表示元素在后半部分
      case x=>x<0 match{
        case true =>quickSelectLittle(l,start,s-1, i)
        case false =>quickSelectLittle(l,s+1,end,x)
      }
    }
}
```



## Hoare划分



```scala
//这个霍尔划分法没有做其他的增强效率的操作，比如用特殊的方法，取一个更好的p为中轴
def HoarePartion(l:Array[Int],start:Int,end:Int)={
  var (p,i,j)=(l(start),start,end+1);
  do{
    //i指从左到右扫描到第一个大于等于p的数为止
    do i+=1 while(i<end&&l(i)<p)
    //j从右向左扫描到小于p的第一个数为止
    do j-=1 while(j>start&&l(j)>p)
    //交换i,j
    swap(l, i, j)
  }while(i<j)//知道i和j交叉，即当i>=j的时候结束循环
  //复原上次交换，因为i>=j这次交换是无效的
  swap(l, i, j)
  //把起始的l的第一个元素交换到j位置上，完成划分，返回j的位置，即中轴
  swap(l, start, j);j
}
```

测试：

```scala
def main(args: Array[String]): Unit = {
  val s=Array(9,12,42,41,2,5,0,41,2,5,9,3,13,4,5)
  quickSort(s, 0, s.length-1);
  println(s.toList)
}

/*
output:
List(0, 2, 2, 3, 4, 5, 5, 5, 9, 9, 12, 13, 41, 41, 42)
*/
```



### 实现快排

```scala
  def quickSort(l:Array[Int],start:Int,end:Int){
    if(start<end){
      val s=HoarePartion(l, start, end);
      
      //val s=LomutoPartion(l, start, end);   你也可以用Lomuto划分，或者你自己的划分算法
      
      //分治策略，更好的方法时在l.lengtg<10的时候用选择排序，其实改进后可以成为尾递归，这里没做
      quickSort(l, start, s-1);
      quickSort(l, s+1, end);
    }
  }
```

# 归并排序

  基于分治策略的归并排序，如果需要的话空间复杂度可以到O(1),即在位排序，只用一个空间来做交换，但是这样对算法时间效率就毫无提升了，失去意义。

  要发挥最差nlog(n)的时间效率（由主定理得出）就需要有O(n)空间牺牲。但是应该记住归并排序的最大优势是**稳定性**！如果你需要在某些单片机上在位排序，又想获得最差nlong(n)可以用**堆排序**

```scala
//归并排序    
def mergeSort(ls:Array[Int]):Array[Int]={
      //这里要注意的是如果ls的length是1那么l是空，而r长度是1
      val (l,r)=(ls.take(ls.length/2),ls.drop(ls.length/2))
    
  	  //如果l，r的长度大于1的话就做递归
      if(l.length>1) mergeSort(l);
      if(r.length>1) mergeSort(r);
      
      merge(l, r, ls);
}
    
//合并两个数组
def merge(l:Array[Int],r:Array[Int],ls:Array[Int]):Array[Int]={
  var i,j=0;
  for(k <- 0 to ls.length-1){
    //用模式配置来匹配各种情形
    (i<l.length,j<r.length) match{
      case (true,true) => if(l(i)<=r(j)){ls(k)=l(i);i+=1}else{ls(k)=r(j);j+=1}
      case (true,false) => {ls(k)=l(i);i+=1}
      case (false,true) => {ls(k)=r(j);j+=1}
      case _=>
    }
   }
  ls
}
```

测试：

```scala
def main(args: Array[String]): Unit = {
  val s=Array(9,12,42,41,2,5,0,41,2,5,9,3,13,4,5)
  mergeSort(s)
  println(s.toList)
}

/*
output:
List(0, 2, 2, 3, 4, 5, 5, 5, 9, 9, 12, 13, 41, 41, 42)
*/
```

