title: csharparalle_problem
date: 2016-08-30 23:06:01
tags:
---

```
 string[] strs= new string[100];
            for (int i = 0; i < 100; i++)
			{
                strs[i] = DateTime.Now.Add(TimeSpan.FromDays(i)).ToString("yyyy-MM-dd");
			}

            string[] strs2 = new string[100];
            for (int i = 0; i < 100; i++)
            {
                strs2[i] = DateTime.Now.AddYears(1).Add(TimeSpan.FromDays(i)).ToString("yyyy-MM-dd");
            }

            string[] strs1 = new string[100];
            for (int i = 0; i < 100; i++)
            {
                strs1[i] = DateTime.Now.Add(TimeSpan.FromDays(i)).ToString("HH:mm:ss");
            }

            ConcurrentDictionary<int, string> dic = new ConcurrentDictionary<int, string>();

            ConcurrentDictionary<DateTime,List<int>> dic1 = new ConcurrentDictionary<DateTime,List<int>>();

            ParallelLoopResult result = Parallel.For(0, 10, (i) =>
            {
                dic1.AddOrUpdate(DateTime.Now.Date,(k) =>
                {
                    // Thread.Sleep(1000);
                    List<int> newList = new List<int>();
                    newList.Add(i);
                    foreach (var item in newList)
                    {
                        Console.WriteLine("init add :" + item + " " + DateTime.Now.ToString("HH:mm:ss fff"));
                    }
                    return newList;
                }, (k, v) =>
                {
                    //Console.WriteLine("Task:update index:" + i + ":" + v + " to " + strs[i]);
                    // Thread.Sleep(1000);
                    v.Add(i);
                    Console.WriteLine("init update : " + DateTime.Now.ToString("HH:mm:ss fff"));
                    return v;
                });

                List<int> countList = null;
                if (dic1.TryRemove(DateTime.Now.Date, out countList) && countList.Count >= 2)
                {
                	//这边会报集合已修改，无法进行遍历，看来countList线程共享？
                   foreach (var item in countList)
                    {
                        Console.WriteLine("removed :" + " " + DateTime.Now.ToString("HH:mm:ss fff"));
                   }
                }
                else if (countList != null)
                {
                    dic1.AddOrUpdate(DateTime.Now.Date, (k) =>
                    {
                        // Thread.Sleep(1000);
                        foreach (var item in countList)
                        {
                            Console.WriteLine("also add : "+ DateTime.Now.ToString("HH:mm:ss fff"));
                        }
                        return countList;
                    }, (k, v) =>
                    {
                        //Console.WriteLine("Task:update index:" + i + ":" + v + " to " + strs[i]);
                        // Thread.Sleep(1000);
                        v.AddRange(countList);

                        foreach (var item in v)
                        {
                            Console.WriteLine("update : " + DateTime.Now.ToString("HH:mm:ss fff"));
                        }


                        return v;
                    });
                }


            });

            //while (!result.IsCompleted)
            //{

            //}

            //foreach (var item in dic1)
            //{
            //    List<int> intList = item.Value;
            //    for (int i = 0; i < intList.Count; i++)
            //    {
            //        Console.WriteLine(intList[i]);
            //    }

            //}
            Console.Read();


           Task task0 = Task.Run(() =>

           {
               for (int i = 0; i < 100; i++)
                {
                    dic.AddOrUpdate(i, (k) =>{
                       // Thread.Sleep(1000);
                        Console.WriteLine("Task:add:" +" index:"+i+" "+ strs[i]);
                        return strs[i];

                    }, (k, v) =>
                    {
                        Console.WriteLine("Task:update index:"+i+":"+ v + " to " + strs[i]);
                       // Thread.Sleep(1000);
                        return strs[i];
                    });
                }
            });

           //task0.Start();

           Task task1 = Task.Run(() =>
            {
                for (int i = 0; i < 100; i++)
                {
                    string outValue ;
                    dic.TryRemove(i, out outValue);
                    if (!string.IsNullOrEmpty(outValue))
                    {
                        Console.WriteLine("Task1:remove:" + " index:" + i + " " + outValue);
                        dic.AddOrUpdate(i, (k) =>
                        {
                           // Thread.Sleep(1000);
                            Console.WriteLine("Task1:add:" + " index:" + i + " " + strs1[i]);
                            return strs1[i];

                        }, (k, v) =>
                        {
                            Console.WriteLine("Task1:update index:" + i + ":" + v + " to " + strs1[i]);
                            //Thread.Sleep(1000);
                            return strs1[i];
                        });
                    }
                    else
                    {
                        Console.WriteLine("Task1:remove:" + " index:" + i + " empty");
                    }


                }
            });

           Task task2 = Task.Run(() =>
           {
               for (int i = 0; i < 100; i++)
               {
                   dic.AddOrUpdate(i, (k) =>
                   {
                       // Thread.Sleep(1000);
                       Console.WriteLine("Task2:add:" + " index:" + i + " " + strs2[i]);
                       return strs2[i];

                   }, (k, v) =>
                   {
                       Console.WriteLine("Task2:update index:" + i + ":" + v + " to " + strs2[i]);
                       // Thread.Sleep(1000);
                       return strs2[i];
                   });
               }
           });

           Task.WaitAll();
           //https://msdn.microsoft.com/zh-cn/library/dd997369.aspx
          // task1.Start();
            Console.Read();
```
