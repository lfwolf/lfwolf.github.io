---
title: 'perf between c++,java and dotnet'
date: 2019-09-19 21:25:52
tags:
---

## 为什么要做这个性能测试对比

一直以来C++都是以性能著称的，我接触到的项目中的项目中用C++的有2种:

    1. 一是做windows界面开发，传统物流行业的操作界面，支持拖拉拽，接受实时消息。
    2. 二是做linux后台服务框架，主要用于操作内存处理消息，调用lua脚本。

而C++的弱点则众所周知是令人崩溃的指针。在之前的界面开发中无数次碰到指针越界系统崩溃、内存泄露导致内存暴涨以及GDI句柄泄漏导致图形无法绘制问题，此类问题相当耗时费力。

很早之前我做过C++，之后转C#，最近几年转java。很想对三者做个比较，正好看到博客园有[相关文章](https://www.cnblogs.com/xiaotie/p/perf-3langs.html),所以我也动手对比了下。

## 测试环境

MacBook Pro
    1. OS: macOS Mojave (10.14.6)
    2. CPU: 2.3 GHz Intel Core i5
    3. Mem: 8 GB 2133 MHz LPDDR3）
    4. G++: Apple LLVM version 10.0.1 (clang-1001.0.46.4)
    5. java: 1.8.0_202
    6. jvm: Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)
    7. .NET Core: 2.2.402

## 测试用例

两个简单的测试用例。均是循环5000次，操作 len = 1000000 的连续内存，计算执行时间。Test1是循环赋固定值，Test2则是动态值（位置相关）。

## 测试结果

<font color="red">Java居然比C++更快！</font>
Dotnet Core在mac下太慢了，找机会到windows/linux下再测一遍。

| 语言 | Test1 | Test2 | 备注 |
| --- | -- | --- | -- |
| C++ | 0.864638 | 1.66328 | 编译参数 O2优化|
| Java | 0.607 | 1.590 | 默认|
| C# | 10.5485014 | 13.8977086 | 默认 |

## 深入分析

显然，jvm开启了超能力。要理解这一现象，就需要对Java虚拟机的机制有深入了解。HotSpot 虚拟机里内置了两个JIT编译器：Client Compiler和Server Compiler，简称为C1编译器和C2编译器。C1编译器将字节码编译为本地代码，进行简单、 可靠的优化，如有必要将加入性能监控的逻辑。C2编译会启用一些编译耗时较长的优化，甚至进行一些激进优化。

查找文献可知，默认情况下，当方法调用次数+循环回边次数超过10000、计数器是int等几个简单类型、步增是常量时，会触发C2编译优化。test2恰恰满足这三种情况！

java版本测试增加Test3如下,测试耗时（<font color="red">6.132s</font>）:
```java
public static void testpoint3(int[] array,int loops,int step){ 
    for(int k =0;k< loops;k++)
    {
        for(int i=1;i< array.length;i+=step)
        {
            array[i] = array[i-1];
        }
    }
}
```
总结：随着JVM、.Net等虚拟机技术的发展，语言特性对高性能计算性能影响越来越低，对计算机体系结构、编译原理、虚拟机编译机制的理解，对性能的影响变得更为重要。
## 完整代码

1. C++
    ```c++
    #include <stdio.h>
    #include <time.h>
    #include <iostream>

    using namespace std;

    void test1(int32_t* p0,int len);
    void test2(int32_t* p0,int len);

    int main()
    {
        printf("Hello world C++\r\n");

        int len = 1000000;
        int32_t* p0 = new int[len];

        clock_t start,end;


        start = clock();
        test1(p0,len);
        //需要测试运行时间的程序段
        end = clock();
        cout<<"test1运行时间(s): "<<(double)(end-start)/CLOCKS_PER_SEC<<endl;



        start = clock();
        test2(p0,len);
        //需要测试运行时间的程序段
        end = clock();
        cout<<"test2运行时间(s): "<<(double)(end-start)/CLOCKS_PER_SEC<<endl;


        printf("Bye world\r\n");
        return 0;
    }

    void test1(int32_t* p0,int len)
    {
        for(int k=0;k<5000;k++){
            int32_t* p= p0 +1;
            int32_t* pEnd = p + len;
            while(p < pEnd){
                *p = k;
                p++;
            }
        }
    }
    void test2(int32_t* p0,int len)
    {
        for(int k=0;k<5000;k++){
            int32_t* p= p0 +1;
            int32_t* pEnd = p + len;
            while(p < pEnd){
                *p = *(p-1);
                p++;
            }
        }
    }
    ```

2. Java

    ```java
    public class testpoint{
        public static void main(String[] args){
            System.out.println("Hello World Java");
            int len = 1000000;
            int loops = 5000;
            int[] a = new int[len];
            long time1=System.currentTimeMillis();	
            testpoint1(a,loops);
            long time2=System.currentTimeMillis();	
            System.out.println("test1运行时间(ms): "+(double)((time2-time1)));
            time1=System.currentTimeMillis();	
            testpoint2(a,loops);
            time2=System.currentTimeMillis();	
            System.out.println("test2运行时间(ms): "+(time2-time1));
            time1=System.currentTimeMillis();	
            testpoint11(a,loops,1);
            time2=System.currentTimeMillis();	
            System.out.println("test11运行时间(ms): "+(time2-time1));
            time1=System.currentTimeMillis();	
            testpoint22(a,loops,1);
            time2=System.currentTimeMillis();	
            System.out.println("test22运行时间(ms): "+(time2-time1));
            System.out.println("Bye World Java");
        }

        public static void testpoint1(int[] array,int loops){ 
            for(int k =0;k< loops;k++)
            {
                for(int i=1;i< array.length;i++)
                {
                    array[i] = k;
                }
            }

        }
        public static void testpoint2(int[] array,int loops){ 
            for(int k =0;k< loops;k++)
            {
                for(int i=1;i< array.length;i++)
                {
                    array[i] = array[i-1];
                }
            }

        }
        public static void testpoint22(int[] array,int loops,int step){ 
            for(int k =0;k< loops;k++)
            {
                for(int i=1;i< array.length;i+=step)
                {
                    array[i] = array[i-1];
                }
            }

        }
        public static void testpoint11(int[] array,int loops,int step){ 
            for(int k =0;k< loops;k++)
            {
                for(int i=1;i< array.length;i+=step)
                {
                    array[i] = k;
                }
            }

        }
    }
    ```

3. C#

    ```c#
    using System;
    using System.Diagnostics;
    namespace testpoint
    {
        class Program
        {
            static void Main(string[] args)
            {
                Console.WriteLine("Hello World C#!");
                Stopwatch stopwatch = new Stopwatch();
                int len = 1000000;
                int loops = 5000;
                int[] a = new int[len];
                stopwatch.Start();
                testpoint1(a, loops);
                stopwatch.Stop();
                Console.WriteLine("test1运行时间(s): " + stopwatch.Elapsed.TotalSeconds);
                stopwatch.Restart();
                testpoint2(a, loops);
                stopwatch.Stop();
                Console.WriteLine("test2运行时间(s): " + stopwatch.Elapsed.TotalSeconds);
                Console.WriteLine("Bye World C#");
            }

            public static void testpoint1(int[] array, int loops)
            {
                for (int k = 0; k < loops; k++)
                {
                    for (int i = 1; i < array.Length; i++)
                    {
                        array[i] = k;
                    }
                }

            }
            public static void testpoint2(int[] array, int loops)
            {
                for (int k = 0; k < loops; k++)
                {
                    for (int i = 1; i < array.Length; i++)
                    {
                        array[i] = array[i - 1];
                    }
                }

            }
        }
    }
    ```
