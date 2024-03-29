---
title: POJ 1300 Door Man 欧拉路&字符串处理TAT
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [October 26, 2014](https://web.archive.org/web/20210418234931/https://void-shana.moe/acmalgo/poj-1300-door-man-%e6%ac%a7%e6%8b%89%e8%b7%af%e5%ad%97%e7%ac%a6%e4%b8%b2%e5%a4%84%e7%90%86tat.html "3:01 am") 
[VOID001](https://web.archive.org/web/20210418234931/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20210418234931/https://void-shana.moe/acmalgo/poj-1300-door-man-%e6%ac%a7%e6%8b%89%e8%b7%af%e5%ad%97%e7%ac%a6%e4%b8%b2%e5%a4%84%e7%90%86tat.html#respond)





Door Man



| **Time Limit:** 1000MS |  | **Memory Limit:** 10000K |
| **Total Submissions:** 2140 |  | **Accepted:** 859 |



Description



You are a butler in a large mansion. This mansion has so many rooms that they are merely referred to by number (room 0, 1, 2, 3, etc…). Your master is a particularly absent-minded lout and continually leaves doors open throughout a particular floor of the house. Over the years, you have mastered the art of traveling in a single path through the sloppy rooms and closing the doors behind you. Your biggest problem is determining whether it is possible to find a path through the sloppy rooms where you:


1. Always shut open doors behind you immediately after passing through
2. Never open a closed door
3. End up in your chambers (room 0) with all doors closed


In this problem, you are given a list of rooms and open doors between them (along with a starting room). It is not needed to determine a route, only if one is possible.



Input


Input to this problem will consist of a (non-empty) series of up to 100 data sets. Each data set will be formatted according to the following description, and there will be no blank lines separating data sets.  

A single data set has 3 components: 
1. Start line – A single line, “START M N”, where M indicates the butler’s starting room, and N indicates the number of rooms in the house (1 <= N <= 20).
2. Room list – A series of N lines. Each line lists, for a single room, every open door that leads to a room of higher number. For example, if room 3 had open doors to rooms 1, 5, and 7, the line for room 3 would read “5 7”. The first line in the list represents room 0. The second line represents room 1, and so on until the last line, which represents room (N – 1). It is possible for lines to be empty (in particular, the last line will always be empty since it is the highest numbered room). On each line, the adjacent rooms are always listed in ascending order. It is possible for rooms to be connected by multiple doors!
3. End line – A single line, “END”


Following the final data set will be a single line, “ENDOFINPUT”.


Note that there will be no more than 100 doors in any single data set.



Output


For each data set, there will be exactly one line of output. If it is possible for the butler (by following the rules in the introduction) to walk into his chambers and close the final open door behind him, print a line “YES X”, where X is the number of doors he closed. Otherwise, print “NO”.
Sample Input



```
START 1 2
1

END
START 0 5
1 2 2 3 3 4 4

END
START 0 10
1 9
2
3
4
5
6
7
8
9

END
ENDOFINPUT
```

Sample Output



```
YES 1
NO
YES 10
```

首先，需要搞清楚如何处理输入  参考我的上一篇文章，可知道应该用什么函数进行输入和输出 。 先把输入处理的代码放上 ，下面代码可以把数据读入并且记录每一个节点的度数值



```
/*************************************************************************
    > File Name: 1300.cpp
    > Author: VOID\_133
    > ################### 
    > Mail: ################### 
    > Created Time: 2014年10月25日 星期六 00时31分24秒
 ************************************************************************/
#include<iostream>
#include<algorithm>
#include<cstdio>
#include<vector>
#include<cstring>
#include<map>
#include<queue>
#include<stack>
#include<string>
#include<cstdlib>
#include<ctime>
#include<set>
using namespace std;

const int maxn=25;
int deg[maxn];

int main(void)
{
	char s[50];
	char inpstr[5000];
	while(scanf("%s",s) && strcmp(s,"ENDOFINPUT"))
	{
		if(!strcmp(s,"END")) continue;
		memset(deg,0,sizeof(deg));
		int sumdeg=0;
		int n,m;				//m is the first room ,total has n rooms
		scanf("%d%d",&m,&n);
		//printf("START %d %dn",m,n);							//DBEUG
		getchar();				//Clear the Enter
		int nodenum=0;
		while(nodenum<n-1)
		{
			gets(inpstr);
			int len=strlen(inpstr);
			int inpnum=0;
			bool flag;
			//printf("%sn",s);
			for(int i=0;i<=len;i++)
			{
				//INPUT READIN
				if(inpstr[i]The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)' '|| inpstr[i]The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)' ')
				{
					deg[nodenum]++;
					deg[inpnum]++;
					inpnum=0;
				}
				else
				{
					//printf("CHAR %c",inpstr[i]
					inpnum=inpnum*10+inpstr[i]-'0';
				}
			}
			nodenum++;
		}
		gets(s);
		gets(s);
		//Read in  END String
		for(int i=0;i<n;i++)
		{
			printf("Deg[%d]=%dn",i,deg[i]);
		}
	}
	return 0;
}
```

实际上上面代码有问题 ，不过 在我写的时候还没有发现问题所在，下面先说这道题的思路，其实这个题很简单就是一个一笔画问题， 每一个房间代表一个节点，每一个开着的门代表一条节点之间的边，图为无向图，问是否存在从节点 m到 0 的一条路径，使得图中每一个边都被经过仅一次。


没有什么太多需要注意的，直接拿判断条件去判断就可以了，判断条件就是，当 m!=0时  即非Euler回路 ，这时要求 有两个奇节点 m 0 其它节点均为偶节点 当m=0时 要求所有节点均为偶节点 然后 根据边数=度数/2我们可以算出来有多少个门


因此我们只需要开个数组记录一下度数 就好了，


下面是AC代码 之前错了一次 是因为字符处理上出问题了，由于最初我gets读入空行的时候，会当作读入了一个数字 0 结果导致出问题 修改代码后就没有问题了 AC代码如下



```
/*************************************************************************
    > File Name: 1300.cpp
    > Author: VOID\_133
    > ################### 
    > Mail: ################### 
    > Created Time: 2014年10月25日 星期六 00时31分24秒
 ************************************************************************/
#include<iostream>
#include<algorithm>
#include<cstdio>
#include<vector>
#include<cstring>
#include<map>
#include<queue>
#include<stack>
#include<string>
#include<cstdlib>
#include<ctime>
#include<set>
using namespace std;

const int maxn=25;
int deg[maxn];

int main(void)
{
	char s[50];
	char inpstr[5000];
	while(scanf("%s",s) && strcmp(s,"ENDOFINPUT"))
	{
		if(!strcmp(s,"END")) continue;
		memset(deg,0,sizeof(deg));
		int sumdeg=0;
		int n,m;				//m is the first room ,total has n rooms
		scanf("%d%d",&m,&n);
		//printf("START %d %dn",m,n);							//DBEUG
		getchar();				//Clear the Enter
		int nodenum=0;
		while(nodenum<n-1)
		{
			gets(inpstr);
			int len=strlen(inpstr);
			int inpnum=0;
			bool flag;
			//printf("%sn",s);
			for(int i=0;i<=len;i++)
			{
				//INPUT READIN
                                //if(inpstr[i]The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)' '|| inpstr[i]The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)' ' )  //WRONG CODE
				if(inpstr[i]The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)' '|| (inpstr[i]The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)' ' && len))   //CORRECTED
				{
					//printf("%d ",inpnum);
					deg[nodenum]++;
					deg[inpnum]++;
					inpnum=0;
				}
				else
				{
					//printf("CHAR %c",inpstr[i]
					inpnum=inpnum*10+inpstr[i]-'0';
				}
			}
			nodenum++;
		}
		//Read in  END String
		gets(s);
		gets(s);
		/*for(int i=0;i<n;i++)
		{
			printf("DEG[%d]=%dn",i,deg[i]);
		}*/
		bool isEuler=true;
		if(mThe content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)0)					//START FROM 0 and END to 0 NO ODD POINTS
		{
			for(int i=0;i<n;i++)
			{
				sumdeg+=deg[i];
				if(deg[i]%2)
				{
					isEuler=false;
					break;
				}
			}
		}
		else
		{
			for(int i=0;i<n;i++)
			{
				sumdeg+=deg[i];
				if(iThe content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)0 || iThe content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)m) continue;
				if(deg[i]%2)
				{
					isEuler=false;
					break;
				}
			}
			if(!(deg[0]%2 && deg[m]%2))
			{
				isEuler=false;
			}
		}
		if(isEuler) printf("YES %dn",sumdeg/2);
		else printf("NOn");
	}
	return 0;
}
```

 






---


[ACM Algorithm](https://web.archive.org/web/20210418234931/https://void-shana.moe/category/acmalgo), [GraphTheory](https://web.archive.org/web/20210418234931/https://void-shana.moe/category/acmalgo/graphtheory) [C. Linux](https://web.archive.org/web/20210418234931/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20210418234931/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20210418234931/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20210418234931/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20210418234931/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20210418234931/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20210418234931/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20210418234931/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
Laravel 学习笔记 #1 Installation and Configuration](https://web.archive.org/web/20210418234931/https://void-shana.moe/webdev/laravel-%e5%ad%a6%e4%b9%a0%e7%ac%94%e8%ae%b0-1-installation-and-configuration.html)
[PREVIOUS 
把基本输入 scanf getchar gets 的区别](https://web.archive.org/web/20210418234931/https://void-shana.moe/acmalgo/%e5%9f%ba%e6%9c%ac%e8%be%93%e5%85%a5-scanf-getchar-gets-%e7%9a%84%e5%8c%ba%e5%88%ab.html)

            