title: 算法--二分法查找
date: 2016-10-17 16:03:03
tags: algorithm
---

## 二分法

[数学领域术语](http://baike.baidu.com/link?url=TzhiEIwhX5ycN0LxEWtUXaIsRYiv4xVo9CdaDi_TFGVhhCQXbMp4KSW4oPUxMytSN-HeKwa9XyHkS9_e1U6R9COk7rHmRfRW8SQyUCfq6fn2j-QYV0S0qpF9-VhGf0TU)
对于区间[a，b]上连续不断且f（a）·f（b）<0的函数y=f（x），通过不断地把函数f（x）的零点所在的区间一分为二，使区间的两个端点逐步逼近零点，进而得到零点近似值的方法叫二分法

```Java
  public int binarySearch(int[] data,int aim){//以int数组为例，aim为需要查找的数
    int start = 0;
    int end = data.length-1;
    int mid = (start+end)/2;//a
    while(data[mid]!=aim && end>start){//如果data[mid]等于aim则死循环，所以排除
        if(data[mid]>aim){
            end = mid-1;
        }else if(data[mid]<aim){
            start = mid+1;
        }
        mid = (start+end)/2;//b，注意a，b
    }
    return (data[mid]!=aim)?-1:mid;//返回结果
  }
```
