此项目为一些前端练习项目，便于更好的学习前端代码。
01.1.主要知识点
1. 掌握标签：h1~h6、p、ul+li、ol+li、div、span、a、img；
2. 了解标签：b、u、i、em、strong；
3. 了解：语义元素、非语义元素；
4. 掌握:url、绝对路径、相对路径；
5. 掌握:本章节中所出现快捷书写；



01.2.官方教程网址
可以点击此处，进入官方教程对应章节；

01.3.标题与段落
 熟练掌握 h1~h6、p标签
<h1>(1级标题)王者荣耀</h1>
    <p>
      (段落）王者荣耀》是由腾讯游戏天美工作室群开发并运行的一款运营在Android、IOS、NS平台上的MOBA类国产手游。
    </p>
    <p>
      (段落）王者荣耀中的玩法以竞技对战为主，玩家之间进行1VS1、3VS3、5VS5等多种方式的PVP对战。
    </p>
    <p>
      (段落）2022年6月7日，腾讯海外发行品牌Level
      Infinite宣布，并将逐步公布多轮内测。
    </p>
    <h2>(2级标题)游戏背景</h2>
    <p>
      (段落）神明乘坐方舟穿越无边的宇宙，降临王者大陆。他们利用宇宙最强大的力量——方舟核心，将传奇英雄的基因注入新人类，创造了那些为人熟知的英雄。
    </p>
01.4.列表标签
01.4.1.掌握ul,li.ol标签使用

    <ul>
      <li>测试1</li>
      <li>测试2</li>
      <li>测试3</li>
      <li>测试4</li>
    </ul>
    <ol>
      <li>测试xzx</li>
      <li>测试xzx</li>
      <li>测试xzx</li>
      <li>测试xzx</li>
    </ol>
列表嵌套
列表标签是成套标签，即：
1. 列表用 ul或者 ol；
2. 列表内只能放列表项li； 
3. 列表项内可以放完整的内容；

01.4.2.利用列表嵌套实现二级目录
快捷写法ul>li*5>{第$章}ul>li*5>{第节$}，观察帮助提示是否正确，正确则按回车;
<ul>
      <li>
        第1一项目录
        <ul>
          <li>第一项两个子列表项</li>
          <li>第一项两个子列表项</li>
        </ul>
      </li>
      <li>
        第2一项目录
        <ul>
          <li>
            第二项三个子列表项
            <ul>
              <li>子列表的第一项</li>
              <li>子列表的第一项</li>
            </ul>
          </li>
          <li>第二项三个子列表项</li>
          <li>第二项三个子列表项</li>
        </ul>
      </li>
      <li>第3一项目录</li>
      <li>第4一项目录</li>
      <li>第5一项目录</li>
    </ul>
运行效果展示


01.5.超链接a标签
01.5.1.什么是超链接

01.5.2.超链接解析 

01.5.3. 超链接标签的target属性

超链接练习
    <ul>
        <!-- 没有超链接 -->
      <li>1111</li>
      
      <!-- 没属性和没链接的意图 -->
      <li><a></a></li>
      
      <li><a>没有href属性</a></li>
      
      <!-- 没有写，表示默认值为 target="_self" -->      
      <li><a href="https://www.taobao.com/">淘宝</a></li>
      <!-- 手机网页的用户习惯 -->
      <li><a target="_self" href="https://www.qq.com/"> QQ </a></li>

      <!-- PC版网页用户习惯 -->
      <li><a href="https://www.qq.com/" target="_blank"> QQ </a></li>
       
      <!-- 链接一个文件 -->
      <li><a href="./0204.2.列表.html">0204.2.列表</a></li>

      <!-- 实战小技巧：目前不知道链接到哪儿，先用#占个位置即可 -->      
      <li><a href="#">0204.2.列表</a></li>
      
    </ul>
01.6.绝对路径
（1） 资料截图

（2） 试一试
1. 找到【a\1.1.html】，并打开；
2. 利用快捷方式敲入：hr+ul>li*4>a[ target="_blank"] ，观察提示是否正确，正确后回车；
3. 按照图示，前两个修改为外网的网址，后两个修改为本机网站的网址；
4. 运行，分别点击四个超链接，看看能否正确的跳转到目标网址；
5. 把本网页复制粘贴到【b】，打开并运行新粘贴的网页文件，分别点击四个超链接，看看能否正确的跳转到目标网址。
（3）附生成后的代码截图

（4）附绝对路径的习惯书写提示截图

（5）总结
1. 网页中，绝对路径只用网络绝对路径，磁盘绝对路径禁止使用；
2. 网络绝对路径就是，浏览器地址栏中完整的网址；
3. 同一个网站内的路径，可以用【/项目名/资源名...】格式；
01.7.相对路径
（1）为什么需要用相对路径
在window系统中，有C盘、D盘、E盘等盘符，因此能告诉别人到哪个磁盘的哪个文件夹的 哪个子文件，把某个文件拷贝出来。但是linux和基于Linux的安卓中没有盘符，你不能让人到哪个磁盘的哪个文件下拷贝文件。
生活中，碰到:"我现在到你那儿去，应该怎么走？"的情景时，对方一般会问:"你现在的位置在哪儿"，" 从你当前的位置出发，怎么走，怎么走，怎么走，就到了"。
生活中的经典解决方式，电脑中也一样，于是就有了相对路径。相对当前位置，如何走如何走，就到了或者找到了。
（2） 几个概念和基本常识
1. 文件夹就是目录，目录就是文件夹。window用户喜欢叫文件夹，linux用户和程序员喜欢用目录；
2. 符号【./】表示当前文件所在的文件夹，可以省略不写，但不建议省略；
3. 符号【../】表示上一级文件夹；
4. 子文件夹，用【文件夹名称/】表示；
5. 注意，路径中第一个符号如果是【/】，是绝对路径的写法哦；
（3）试一试
1. 在【1.1.html】，在绝对路径练习的下面；利用快捷方式敲入：hr+ul>li*5>a[target="_blank"] ，观察提示是否正确，正确后回车；
2. 按照下图所示，分别练习：同一个文件夹内的文件，上一级文件夹内的文件，子文件夹内的文件，绕一绕的文件，三大类文件的超链接；
3. 运行，观察超链接是否能正常跳转；
4. 把本文件复制粘贴到c文件夹下，再看看本次练习的几个超链接是否能正确访问，为什么？
（4）输入截图

（5）输入完成时的截图

（6）运行效果
