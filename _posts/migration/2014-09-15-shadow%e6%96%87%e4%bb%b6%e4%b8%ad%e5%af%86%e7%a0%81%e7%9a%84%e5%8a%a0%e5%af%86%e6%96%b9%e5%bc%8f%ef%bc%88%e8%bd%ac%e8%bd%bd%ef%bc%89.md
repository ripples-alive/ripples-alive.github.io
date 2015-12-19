---
title: shadow文件中密码的加密方式（转载）
author: Ripples
layout: post
permalink: /shadow%e6%96%87%e4%bb%b6%e4%b8%ad%e5%af%86%e7%a0%81%e7%9a%84%e5%8a%a0%e5%af%86%e6%96%b9%e5%bc%8f%ef%bc%88%e8%bd%ac%e8%bd%bd%ef%bc%89/
views:
  - 341
categories:
  - gossip
tags:
  - crypto
  - shadow
---
**<span style="font-size:32px"></span>**<span style="font-size:18px"></span>

<span style="font-size: 16px;">1)&nbsp;查看shadow文件的内容</span>

<blockquote dir="ltr" style="margin-right:0px">
  <p>
    <span style="font-size: 16px;">cat&nbsp;/etc/shadow</span>
  </p>

  <p>
    <span style="font-size: 16px;">可以得到shadow文件的内容，限于篇幅，我们举例说明：</span>
  </p>

  <p>
    <span style="font-size: 16px;">root:$1$Bg1H/4mz$X89TqH7tpi9dX1B9j5YsF.:14838:0:99999:7:::</span>
  </p>

  <p>
    <span style="font-size: 16px;">其&#26684;式为：</span>
  </p>

  <p>
    <span style="font-size: 16px;">{用户名}：{加密后的口令密码}：{口令最后修改时间距原点(1970-1-1)的天数}：{口令最小修改间隔(防止修改口令，如果时限未到，将恢复至旧口令)：{口令最大修改间隔}：{口令失效前的警告天数}：{账户不活动天数}：{账号失效天数}：{保留}</span>
  </p>

  <p>
    <span style="font-size: 16px;">【注】：<span style="font-size: 14px; font-family: KaiTi_GB2312; color: rgb(255, 0, 0);">shadow文件为可读文件，普通用户没有读写权限，超级用户拥有读写权限。如果密码字符串为*，则表示系统用户不能被登入；如果字符串为！，则表示用户名被禁用；如果字符串为空，则表示没有密码。</span></span>
  </p>

  <p>
    <span style="font-family: KaiTi_GB2312; font-size: 16px; color: rgb(255, 0, 0);">我们可以使用passwd&nbsp;–d&nbsp;用户名&nbsp;清空一个用户的口令密码。</span>
  </p>
</blockquote>

<!--more-->

<span style="font-size: 16px;">2)&nbsp;解析shadow文件中密码字符串的内容</span>

<blockquote dir="ltr" style="margin-right:0px">
  <p>
    <span style="font-size: 16px;">对于示例的密码域$1$Bg1H/4mz$X89TqH7tpi9dX1B9j5YsF.，我们参考了linux标准源文件passwd.c，在其中的pw_encrypt函数中找到了加密方法。</span>
  </p>

  <p>
    <span style="font-size: 16px;">我们发现所谓的加密算法，其实就是用明文密码和一个叫salt的东西通过函数crypt()完成加密。</span>
  </p>

  <p>
    <span style="font-size: 16px;">而所谓的密码域密文也是由三部分组成的，即：$id$salt$encrypted。</span>
  </p>

  <p>
    <span style="font-size: 16px;">【注】：<span style="font-size: 14px; font-family: KaiTi_GB2312; color: rgb(255, 0, 0);"> id为1时，采用md5进行加密；</span></span>
  </p>

  <p>
    <span style="font-family: KaiTi_GB2312; font-size: 16px; color: rgb(255, 0, 0);">id为5时，采用SHA256进行加密；</span>
  </p>

  <p>
    <span style="font-family: KaiTi_GB2312; font-size: 16px; color: rgb(255, 0, 0);">id为6时，采用SHA512进行加密。</span>
  </p>
</blockquote>

<span style="font-size: 16px;">3)&nbsp;数据加密函数crypt()讲解</span>

<blockquote dir="ltr" style="margin-right:0px">
  <p>
    <span style="font-size: 16px;">i.&nbsp;头文件：#define&nbsp;_XOPEN_SOURCE</span>
  </p>

  <p>
    <span style="font-size: 16px;">#include&nbsp;<unistd.h></span>
  </p>

  <p>
    <span style="font-size: 16px;">ii.&nbsp;函数原型：char&nbsp;*crypt(const&nbsp;char&nbsp;*key,&nbsp;const&nbsp;char&nbsp;*salt);</span>
  </p>

  <p>
    <span style="font-size: 16px;">iii.&nbsp;函数说明：crypt()将使用DES演算法将参数key所指的字符串加以编码，key字符串长度仅取前8个字符，超过此长度的字符没有意义。参数salt为两个字符组成的字符串，由a-z、A-Z、0-9，’.’和’/’所组成，用来决定使用4096种不同内建表&#26684;的哪一种。函数执行成功后会返回指向编码过的字符串指针，参数key所指向的字符串不会有所改动。编码过的字符串长度为13个字符，前两个字符为参数salt代表的字符串。</span>
  </p>

  <p>
    <span style="font-size: 16px;">iv.&nbsp;返回&#20540;：返回一个指向以NULL结尾的密码字符串</span>
  </p>

  <p>
    <span style="font-size: 16px;">v.&nbsp;附加说明：使用GCC编译时需要加上&nbsp;–lcrypt</span>
  </p>
</blockquote>

<span style="font-size: 16px;">4)&nbsp;加密参数salt的由来</span>

<blockquote dir="ltr" style="margin-right:0px">
  <p>
    <span style="font-size: 16px;">在我们的示例密码域中salt为Bg1H/4mz，那么它又是如何来的？</span>
  </p>

  <p>
    <span style="font-size: 16px;">我们还是从标准源文件passwd.c中查找答案。在passwd.c中，我们找到了与salt相关的函数crypt_make_salt。</span>
  </p>

  <p>
    <span style="font-size: 16px;">在函数crypt_make_salt中出现了很多的判断条件来选择以何种方式加密(通过id&#20540;来判断)，但其中对我们最重要的一条语句是gensalt(salt_len)。</span>
  </p>

  <p>
    <span style="font-size: 16px;">我们继续查看了函数static&nbsp;char&nbsp;*gensalt&nbsp;(unsigned&nbsp;int&nbsp;salt_size)，才发现原来神秘无比的salt参数只是某个固定长度的随机字符串而已。</span>
  </p>
</blockquote>

<span style="font-size: 16px;">5)&nbsp;最终结论</span>

<blockquote dir="ltr" style="margin-right:0px">
  <p>
    <span style="font-size: 16px;">在我们每次改写密码时，都会随机生成一个这样的salt。我们登录时输入的明文密码经过上述的演化后与shadow里的密码域进行字符串比较，以此来判断是否允许用户登录。</span>
  </p>

  <p>
    <span style="font-family: KaiTi_GB2312; font-size: 16px; color: rgb(255, 0, 0);">【注】：经过上述的分析，我们发现破解linux下的口令也不是什么难事，但前提是你有机会拿到对方的shadow文件。</span>
  </p>
</blockquote>

<span style="font-size: 16px;">6)&nbsp;示例代码(测试代码)：</span>

<blockquote dir="ltr" style="margin-right: 0px;">
  <p>
    <span style="font-size: 16px;">#include&nbsp;<pwd.h><br />#include&nbsp;<stddef.h><br />#include&nbsp;<string.h><br />#include&nbsp;<shadow.h><br />#include&nbsp;<stdio.h><br />#include&nbsp;<unistd.h><br />int&nbsp;main(int&nbsp;argc,&nbsp;char&nbsp;*argv[])</span>
  </p>

  <p>
    <span style="font-size: 16px;">{</span>
  </p>

  <blockquote dir="ltr" style="margin-right:0px">
    <p>
      <span style="font-size: 16px;">if(argc&nbsp;<&nbsp;2)</span>
    </p>

    <p>
      <span style="font-size: 16px;">{</span>
    </p>

    <blockquote dir="ltr" style="margin-right:0px">
      <p>
        <span style="font-size: 16px;">printf("no&nbsp;usrname&nbsp;input");</span>
      </p>

      <p>
        <span style="font-size: 16px;">return&nbsp;1;</span>
      </p>
    </blockquote>

    <p>
      <span style="font-size: 16px;">}</span>
    </p>

    <p>
      <span style="font-size: 16px;">if&nbsp;(geteuid()&nbsp;!=&nbsp;0)</span>
    </p>

    <p>
      <span style="font-size: 16px;">{</span>
    </p>

    <blockquote dir="ltr" style="margin-right:0px">
      <p>
        <span style="font-size: 16px;">fprintf(stderr,&nbsp;"must&nbsp;be&nbsp;setuid&nbsp;root");</span>
      </p>

      <p>
        <span style="font-size: 16px;">return&nbsp;-1;</span>
      </p>
    </blockquote>

    <p>
      <span style="font-size: 16px;">}&nbsp;</span>
    </p>

    <p>
      <span style="font-size: 16px;">struct&nbsp;spwd&nbsp;*shd=&nbsp;getspnam(argv[1]);</span>
    </p>

    <p>
      <span style="font-size: 16px;">if(shd&nbsp;!=&nbsp;NULL)</span>
    </p>

    <p>
      <span style="font-size: 16px;">{</span>
    </p>

    <blockquote dir="ltr" style="margin-right:0px">
      <p>
        <span style="font-size: 16px;">static&nbsp;char&nbsp;crypt_char[80];</span>
      </p>

      <p>
        <span style="font-size: 16px;">strcpy(crypt_char,&nbsp;shd->sp_pwdp);</span>
      </p>

      <p>
        <span style="font-size: 16px;">char&nbsp;salt[13];</span>
      </p>

      <p>
        <span style="font-size: 16px;">int&nbsp;i=0,j=0;</span>
      </p>

      <p>
        <span style="font-size: 16px;">while(shd->sp_pwdp[i]!=' ')</span>
      </p>

      <p>
        <span style="font-size: 16px;">{</span>
      </p>

      <blockquote dir="ltr" style="margin-right:0px">
        <p>
          <span style="font-size: 16px;">salt[i]=shd->sp_pwdp[i];</span>
        </p>

        <p>
          <span style="font-size: 16px;">if(salt[i]=='$')</span>
        </p>

        <p>
          <span style="font-size: 16px;">{</span>
        </p>

        <blockquote dir="ltr" style="margin-right:0px">
          <p>
            <span style="font-size: 16px;">j&#43;&#43;;</span>
          </p>

          <p>
            <span style="font-size: 16px;">if(j==3)</span>
          </p>

          <p>
            <span style="font-size: 16px;">{</span>
          </p>

          <blockquote dir="ltr" style="margin-right:0px">
            <p>
              <span style="font-size: 16px;">salt[i&#43;1]=' ';</span>
            </p>

            <p>
              <span style="font-size: 16px;">break;</span>
            </p>
          </blockquote>

          <p>
            <span style="font-size: 16px;">}</span>
          </p>
        </blockquote>

        <p>
          <span style="font-size: 16px;">}</span>
        </p>

        <p>
          <span style="font-size: 16px;">i&#43;&#43;;</span>
        </p>
      </blockquote>

      <p>
        <span style="font-size: 16px;">}</span>
      </p>

      <p>
        <span style="font-size: 16px;">if(j<3)</span>
      </p>

      <p>
        <span style="font-size: 16px;">perror("file&nbsp;error&nbsp;or&nbsp;user&nbsp;cannot&nbsp;use.");</span>
      </p>

      <p>
        <span style="font-size: 16px;">if(argc==3)</span>
      </p>

      <p>
        <span style="font-size: 16px;">{</span>
      </p>

      <blockquote dir="ltr" style="margin-right:0px">
        <p>
          <span style="font-size: 16px;">printf("salt:&nbsp;%s,&nbsp;crypt:&nbsp;%sn",&nbsp;salt,&nbsp;crypt(argv[2],&nbsp;salt));</span>
        </p>

        <p>
          <span style="font-size: 16px;">printf("shadowd&nbsp;passwd:&nbsp;%sn",&nbsp;shd->sp_pwdp);</span>
        </p>
      </blockquote>

      <p>
        <span style="font-size: 16px;">}</span>
      </p>
    </blockquote>

    <p>
      <span style="font-size: 16px;">}</span>
    </p>

    <p>
      <span style="font-size: 16px;">return&nbsp;0;</span>
    </p>
  </blockquote>

  <p>
    <span style="font-size: 16px;">}</span>
  </p>

  <p>
    <span style="font-size: 16px;">编译： gcc&nbsp;passwd.c&nbsp;-lcrypt&nbsp;-o&nbsp;passwd</span>
  </p>

  <p>
    <span style="font-size: 16px;">运行： ./passwd&nbsp;root&nbsp;123</span>
  </p>

  <p>
    <span style="font-size: 16px;">结果： salt:&nbsp;$1$Bg1H/4mz$,&nbsp;crypt:&nbsp;$1$Bg1H/4mz$X89TqH7tpi9dX1B9j5YsF.</span>
  </p>

  <p>
    <span style="font-size: 14px;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shadowd&nbsp;passwd:&nbsp;$1$Bg1H/4mz$X89TqH7tpi9dX1B9j5YsF.&nbsp;</span>
  </p>
</blockquote>

<span style="font-size: 16px;"><br /></span>

<span style="font-size: 16px;">原文链接：<span style="font-size: 14px;"><a href="http://blog.csdn.net/jinyuhongye/article/details/7950961" target="_self">http://blog.csdn.net/jinyuhongye/article/details/7950961</a></span></span>
