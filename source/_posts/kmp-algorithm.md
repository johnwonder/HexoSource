title: kmp-algorithm
date: 2017-03-18 12:57:19
tags: algorithm
---

## kmp算法理解一

前几个月之前读了[KMP算法详解](http://blog.csdn.net/joylnwang/article/details/6778316/)之后，一知半解，昨天再次读了之后有点感悟：

上代码：

```
inline void BuildNext(const char* pattern, size_t length, unsigned int* next)  
{  
  unsigned int i, t;  

  i = 1;  
  t = 0;  
  next[1] = 0;  

  while(i < length + 1)  
  {  
      //abcabcacab
      //当求next[5]时，先计算前4位
      //当i = 4时 ,t = 1
      //找到满足pattern[k] = pattern[j]的最大k值
      //退出循环就是找到
      while(t > 0 && pattern[i - 1] != pattern[t - 1])  
      {  
          t = next[t];  
      }  

      ++t;  
      ++i;  

      if(pattern[i - 1] == pattern[t - 1])  
      {  
          next[i] = next[t];  
      }  
      else  
      {  
          next[i] = t;  
      }  
  }  

  //pattern末尾的结束符控制，用于寻找目标字符串中的所有匹配结果用  
  while(t > 0 && pattern[i - 1] != pattern[t - 1])  
  {  
      t = next[t];  
  }  

  ++t;  
  ++i;  

  next[i] = t;  
}  
```
