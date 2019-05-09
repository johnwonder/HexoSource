title: csharp-linqmerge
date: 2015-12-01 18:02:00
tags: c#
---

##   C# Linq合并List中的对象

```csharp

	static void Main(string[] args){

		List<Person> pList = new List<Person>();

		Person person1 = new Person() { FirstName = "john", LastName = "wonder" };
		Person person3 = new Person() { FirstName = "john2", LastName = "wonder22" };
		Person person4 = new Person() { FirstName = "john2", LastName = "wonder2233" };

		pList.Add(person1);
		pList.Add(person3);
		pList.Add(person4);


	    var pMergeList= pList.ToLookup(x => x.FirstName).Select(x => x.Aggregate((p1, p2) =>

	        new Person{
	             FirstName = p1.FirstName,
	             LastName =  p1.LastName + ";"+ p2.LastName
	        }
	    )).ToList();

	    foreach (var item in pMergeList)
	    {
	        Console.WriteLine("FirstName:" + item.FirstName);
	        Console.WriteLine("LastName:" + item.LastName);

	    }

	    Console.Read();
		}


		class Person
		{
		    public string FirstName { get; set; }

		    public string LastName { get; set; }
		}


		class PersonModel
		{
		    public string FirstName { get; set; }

		    public string LastNameList { get; set; }
		}
```
