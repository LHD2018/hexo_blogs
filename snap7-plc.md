---
uuid: eb11ebb1-df9f-8074-738f-57bed2c95296
title: 在VS中配置snap7并用snap7与PLC通信
date: 2020-07-29 21:22:22
tags:
- PLC
- snap7
---

# 前言

之前实验室的小车使用OPC与上位机通信，但由于年代已久，师兄们写的代码已看不懂加上OPC配置比较麻烦，故现在现在采用snap7进行开发。

## snap7介绍

Snap7是一个基于以太网与西门子S7系列PLC通信的开源库，在世界领域应用很广。但也许是因为资料比较少，而且很多都是纯英文，在国内反而没有大规模的应用。
[snap7官网](http://snap7.sourceforge.net/)有具体的说明以及相应的论坛。

# vs上snap7配置

1. 首先去这个网址下载snap7的开发包<https://sourceforge.net/projects/snap7/files/1.4.2/>， 下载解压后如图所示有许多文件，但是大部分我们都是用不到的，可以看doc里面的文档和example里的代码示例，我们主要用的就是snap7.h, snap7.cpp, snap7.lib, snap7.dll 四个文件，在windows搜索栏中将他们找到。

![](https://i.loli.net/2020/07/29/bcDz8ZL7KTXf3nS.png)

2. VS中新建一个项目，切换文件夹视图创建src， include， lib三个文件夹，将snap7.cpp放入src，将snap7.h放入include，将snap7.lib放入lib，将snap7.dll放入项目根目录。切换回解决方案视图，在头文件和源文件中添加现有项snap7.h和snap7.cpp，如图。
![](https://i.loli.net/2020/07/30/OIbveFdYJCBsoiN.png)
3. 包含头文件和库文件，选中项目右击选择属性，如图添加刚才的include，lib文件夹

![](https://i.loli.net/2020/07/30/iIWT1Uv384r5om6.png)

![](https://i.loli.net/2020/07/30/SCXwhcTtiudfEpQ.png)



# PLC上的配置

plc在博图上写好程序后要进行设置，确定ip地址和子网掩码，必须和连接的PC一样，将保护设为无保护，允许put/get远程连接，如图所示。注意如果用到DB块，需要取消对块的优化。好了之后就可以下载到PLC上了。

![](https://i.loli.net/2020/07/30/Kn6w17r5uB9EJel.png)

![](https://i.loli.net/2020/07/30/VCScgdR1WzPFnYD.png)

![](https://i.loli.net/2020/07/30/5fNAue7PIj6rUbg.png)



# snap7的简单使用

上面配置好开发环境后就可以写具体的代码了,下面一个简单的demo
~~~c++
#include <iostream>
#include <bitset>
#include <snap7.h>

#pragma comment(lib, "snap7.lib")

using namespace std;

#ifdef OS_WINDOWS
# define WIN32_LEAN_AND_MEAN
# include <windows.h>
#endif

const char* plc_ip = "192.168.1.11";//定义plc的IP地址

int main(int argc, char** argv) {
	
	TS7Client* client = new TS7Client();//创建一个客户端

	int ret = client->ConnectTo(plc_ip, 0, 0);//连接plc
	if (ret != 0)
	{
		return -1;
	}
	byte buff[10] = {0};//创建一个读写缓存区
	
	
	client->WriteArea(0x82, 0, 0, 1, 0x01, &buff);//写操作
	client->ReadArea(0x81, 0, 0, 1, 0x01, &buff);//读操作
	//cout << bitset<8>(buff[0]) << endl;
	client->Disconnect();//断开连接
	return 0;
}
~~~

这里我们主要用到的就是ReadArea和WriteArea函数。

+ ReadArea（int Area，int DBNumber，int Start，int Amount，int WordLen，void * pUsrData）;

|      | type | mean |
| ---- | ---- | ---- |
| Area | integer | 寻址确定 |
| DBNumber | integer | DB块序号，如果不是DB块就是0 |
| Start | integer | 变量开始地址 |
| Amount | integer | 读取变量数量 |
|Wordlen|integer|每一个变量的大小|
| pusrData | Pointer | 数据缓存的地址 |

这里注意Start变量，比如读取变量Q1.3的数据，那么start=（1*8）+3

+ Area 表
|value|mean|
|---|----|
|0x81|input|
|0x82|output|
|0x83|Merker|
|0x84|DB|
|0x1C|conter|
|0x1D|Timer|

+ wordlen 表
|Value|mean|
|---|---|
|0x01|bit|
|0x02|byte(8bit)|
|0x04|word(16bit)|
|0x06|double word(32bit)|
|0x08|Real(32bit float)|
|0x1C|Counter(16bit)|
|0x1D|Timer(16bit)|

还有许多用法请参考手册

推荐几个相关的博客可以参考一下
<https://blog.csdn.net/weixin_29482793/article/details/79555836>

<https://blog.csdn.net/qq_34935373/article/details/97374783>



