---
title: Unix-like 系统的文件权限（转载）
author: Ripples
layout: post
views:
  - 442
categories:
  - technology
tags:
  - gid
  - uid
  - 权限
---
网上关于uid这一块的东西多是多，只是大多都太乱了，今天偶然发现一个感觉很好的，mark一下～～～

<h2 style=";padding: 0px;">
  1&nbsp;名词解<span style="font-family: arial, helvetica, sans-serif;"></span>释
</h2>

<table cellpadding="0" cellspacing="0" valign="top">
  <tr class="firstRow">
    <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="171">
      <p style=";font-family: 微软雅黑;font-size: 14px">
        uid（user id）
      </p>
    </td>

    <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="550">
      <p style=";font-family: 微软雅黑;font-size: 14px">
        一个uid表示系统中的一个用户，可对应多个gid
      </p>
    </td>
  </tr>

  <tr>
    <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="171">
      <p style=";font-family: 微软雅黑;font-size: 14px">
        gid（group id）
      </p>
    </td>

    <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="550">
      <p style=";font-family: 微软雅黑;font-size: 14px">
        一个gid表示系统中的一个组，可对应多个uid
      </p>
    </td>
  </tr>

  <p>
    <br />

    <tr>
      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="171">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          ruid（real user id）
        </p>
      </td>

      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="540">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          一个进程的真实uid，如果不是root，进程间的signal必须是相同的ruid之间才能进行，例如子进程继承了父进程的ruid，所以可以互相signal。
        </p>
      </td>
    </tr>

    <tr>
      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="171">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          rgid（real group id）
        </p>
      </td>

      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="540">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          一个进程的真实gid
        </p>
      </td>
    </tr>

    <tr>
      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="161">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          euid（effective user id）
        </p>
      </td>

      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="540">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          一个进程的有效uid，当一个进程创建或访问文件时，用的不是ruid的权限，而是euid的权限。
        </p>
      </td>
    </tr>

    <tr>
      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="161">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          egid（effective group id）
        </p>
      </td>

      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="540">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          一个进程的有效gid，原理同上
        </p>
      </td>
    </tr>

    <tr>
      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="161">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          suid（saved user id）
        </p>
      </td>

      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px; word-break: break-all;" width="540">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          一个特权进程（privilege&nbsp;process，它的euid是特权值）在运行中间可能需要做一些非特权工作，就会将euid设置为某个非特权的，这时会将特权值存放在suid，注意，这时进程已经变成非特权进程（unprivileged process）了，而一般普通的非特权进程是无法主动把自己变成一个特权进程的，因为对euid的赋值只能是ruid、euid、suid之一，但由于前面把特权值存放在了suid，所以这时进程就可以恢复成特权进程了。
        </p>
      </td>
    </tr>

    <tr>
      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="161">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          sgid（saved group id）
        </p>
      </td>

      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="540">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          原理同上，给一个特权进程恢复的机会。
        </p>
      </td>
    </tr>

    <tr>
      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="161">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          fsuid（file system user id）
        </p>
      </td>

      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="540">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          linux引入的，单独控制对文件系统的访问权限，不过一旦euid发生变化，fsuid立即跟着变化。fsuid的用途是，允许一些程序（例如NFS server）拥有一些已知uid的对文件系统的访问权限，但是不拥有uid对应的发送signal的权限。也就是说，在不修改ruid、euid的情况下，就改变了文件系统访问权限。对fsuid的赋值只能是ruid、euid、suid之一。
        </p>
      </td>
    </tr>

    <tr>
      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="161">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          fsgid（file system group id）
        </p>
      </td>

      <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="540">
        <p style=";font-family: 微软雅黑;font-size: 14px">
          原理同上，赋值只能是rgid、egid、sgid之一。
        </p>
      </td>
    </tr></tbody> </table>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      注：suid和sgid和下面所说的SUID和SGID含义不同，这里指的是某一个uid或gid的值，而下面的SUID和SGID是一个位（bit）是否设置。缩写相同会导致混淆，所以我用大小写来区分。
    </p>

    <h2 style=";padding: 0px">
      <a style="color: rgb(106, 57, 6)" name="t1"></a>2&nbsp;文件权限基础
    </h2>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      2.1&nbsp;用户的分类：文件的拥有者（owner，简称u）、文件的组ID对应的组用户（file'sgroup，简称g）、其它用户（others，简称o）。
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;&nbsp;&nbsp;如果是所有用户（包括三种，all，简称a）。
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      2.2&nbsp;对于一个文件用户可能有以下三种基本权限
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <table cellpadding="0" cellspacing="0" valign="top">
      <tr class="firstRow">
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="27">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="47">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            文件
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="596">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            文件夹
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="27">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            读r
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="47">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            读文件
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="586">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            列出文件夹内容。屏蔽这个位可以对别人隐藏文件夹中各个文件名。
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="42">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            写w
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="72">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            写文件
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="586">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            创建或移除文件。屏蔽这个位可以使文件夹中的文件数目不变。
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="27">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            执行x
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="47">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            执行文件（run it as a program）
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="586">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            访问文件夹中的文件。如果不具有x权限：无法cd到这个文件夹；无法创建或删除文件（即使有文件夹w权限）；无法读取或写入文件内容（即使对文件夹和文件本身都有rw权限）。
          </p>

          <p style=";font-family: 微软雅黑;font-size: 14px">
            屏蔽这个位，用户唯一能做的就是查看文件夹有哪些文件。
          </p>
        </td>
      </tr>
    </table>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      结论是：必需对文件夹同时拥有wx权限，才能创建或删除文件；必需对文件夹拥有x权限并对文件拥有r或w权限，才能读或写文件。
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      2.3&nbsp;还有三种特殊权限（影响可执行文件，有些系统还影响文件夹）
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <table cellpadding="0" cellspacing="0" valign="top">
      <tr class="firstRow">
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="68">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="180">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            文件
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="402">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            文件夹
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="68">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            SUID（setuid bit）
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="170">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            如果设置，当文件执行时设置进程的euid，这个值是文件拥有者的uid
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="392">
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="128">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            SGID（setgid bit）
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="170">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            如果设置，当文件执行时设置进程的egid，这个值是文件所属的组ID
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="392">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            某些系统中，如果对某文件夹设置了这个位，那么无论是拥有者属于哪个组，在此文件夹中创建的文件和此文件夹同组。
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="68">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            sticky bit
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="170">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            如果设置，执行文件时，即使程序结束，二进制映像也保存在swap分区，可以实现快速重启。但是linux并不支持这个特点。
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="392">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            对文件夹设置这个位，则只有此文件夹的拥有者，或者文件夹中文件的拥有者，或者超级用户才能重命名或删除文件。
          </p>

          <p style=";font-family: 微软雅黑;font-size: 14px">
            Solaris有一个独有特性，针对不可执行文件，如果设置了这个位，不会在内核中缓冲。
          </p>
        </td>
      </tr>
    </table>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      2.4&nbsp;某些文件系统相关属性
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      例如访问控制列表（ACLs，表示一个文件系统只能由哪些用户访问），一个文件能否被压缩、修改或核心转储，这些属性通常是通过文件系统特有的命令来设置的（例如ext2可以用chattr，FFS可以用chrflags）。
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      所以尽管文件本身的属性可能允许用户进行操作，但是实际操作还是会失败，例如：文件系统属性不允许，文件系统挂载为只读的。
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      2.5&nbsp;一个文件创建的时候就会分配拥有者和组，通常来说，拥有者是当前用户，组是文件所属文件夹的组。不过这也不是绝对的，取决于操作系统和文件系统的不同。可以用chown和chgrp修改文件的拥有者和组。
    </p>

    <h2 style=";padding: 0px">
      <a style="color: rgb(106, 57, 6)" name="t2"></a>3&nbsp;文件权限修改（shell命令）
    </h2>

    <p style="margin: 0in 0in 0in 0.375in;font-size: 14px">
      <strong><span style="font-family: Calibri">3.1&nbsp;</span><span style="font-family: 微软雅黑">相关命令</span></strong>
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      添加、删除、修改用户或组（这些命令影响uid和gid）：useradd/userdel/usermod/groupadd/groupdel/groupmod
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      修改文件拥有者：chown
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      修改文件所属组：chgrp
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      修改文件属性：chmod
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      设置忽略掩码：umask
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-size: 14px">
      <strong><span style="font-family: Calibri">3.2</span><span style="font-family: 微软雅黑">&nbsp;chmod</span><span style="font-family: 微软雅黑">命令</span></strong>
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      下面详细说明一下chmod，首先我们看一下Linux中对文件权限的显示
    </p>

    <ol start="1" style="padding: 5px 0px;border: none;position: relative;list-style-position: initial;list-style-image: initial" class=" list-paddingleft-2">
      <li>
        <pre class="brush:plain;toolbar:false">----------&nbsp;1&nbsp;guanxin&nbsp;root&nbsp;0&nbsp;12-28&nbsp;09:35&nbsp;empty.txt&nbsp;&nbsp;
-rwx------&nbsp;1&nbsp;guanxin&nbsp;root&nbsp;0&nbsp;12-28&nbsp;09:36&nbsp;owner.txt&nbsp;&nbsp;
----rwx---&nbsp;1&nbsp;guanxin&nbsp;root&nbsp;0&nbsp;12-28&nbsp;09:36&nbsp;group.txt&nbsp;&nbsp;
-------rwx&nbsp;1&nbsp;guanxin&nbsp;root&nbsp;0&nbsp;12-28&nbsp;09:36&nbsp;others.txt</pre>

        <p>
          &nbsp; &nbsp;
        </p>
      </li>
    </ol>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      <span style="font-size: 14px">可以看到一共有</span><span style="font-size: 14px">10</span><span style="font-size: 14px">个位置，第一个位置标示文件类型（</span><span style="font-size: 14px">&#8211;</span><span style="font-size: 14px">普通文件，</span><span style="font-size: 14px">d</span><span style="font-size: 14px">文件夹，</span><span style="font-size: 14px">b</span><span style="font-size: 14px">块设备文件，</span><span style="font-size: 14px">c</span><span style="font-size: 14px">字符设备文件，</span><span style="font-size: 14px">l</span><span style="font-size: 14px">链接文件，</span><span style="font-size: 14px">p</span><span style="font-size: 14px">命名管道文件，</span><span style="font-size: 14px">s</span><span style="font-size: 14px">套接字文件），接下来的</span><span style="font-size: 14px">9</span><span style="font-size: 14px">个位置分别是拥有者、文件所属组的组用户、其它用户的读、写、执行权限（如果不具有权限显示</span><span style="font-size: 14px">&#8211;</span><span style="font-size: 14px">，具有权限显示一个对应字母）。</span>
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      在Unix-like系统中，文件夹也是文件的一种，所以对于文件夹也可以设置权限。例如：
    </p>

    <ol start="1" style="padding: 5px 0px;border: none;position: relative;list-style-position: initial;list-style-image: initial" class=" list-paddingleft-2">
      <li>
        <pre class="brush:plain;toolbar:false">drwxrwxr--&nbsp;&nbsp;2&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4096&nbsp;12-28&nbsp;09:36&nbsp;test</pre>

        <p>
          &nbsp; &nbsp;
        </p>
      </li>
    </ol>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      <span style="font-size: 14px">以这个</span><span style="font-size: 14px">test</span><span style="font-size: 14px">文件夹为例，</span><span style="font-size: 14px">u</span><span style="font-size: 14px">和</span><span style="font-size: 14px">g</span><span style="font-size: 14px">用户拥有全部权限，但是</span><span style="font-size: 14px">o</span><span style="font-size: 14px">用户只能</span><span style="font-size: 14px">ls test</span><span style="font-size: 14px">，因为具有</span><span style="font-size: 14px">r</span><span style="font-size: 14px">权限，但是不能</span><span style="font-size: 14px">cd test</span><span style="font-size: 14px">（这需要</span><span style="font-size: 14px">x</span><span style="font-size: 14px">权限），也不能</span><span style="font-size: 14px">rm -ftest/others.txt</span><span style="font-size: 14px">（这需要</span><span style="font-size: 14px">wx</span><span style="font-size: 14px">权限，注意！！！创建和删除文件只需要拥有文件夹的</span><span style="font-size: 14px">wx</span><span style="font-size: 14px">权限，而无需对文件本身具有任何权限，例如上面的</span><span style="font-size: 14px">empty.txt</span><span style="font-size: 14px">文件的权限是全空，但是用户只要拥有对</span><span style="font-size: 14px">test</span><span style="font-size: 14px">文件夹的</span><span style="font-size: 14px">wx</span><span style="font-size: 14px">权限，仍然可以删除它！！！）。</span>
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      下面列举几个常用命令：
    </p>

    <ol start="1" style="padding: 5px 0px;border: none;position: relative;list-style-position: initial;list-style-image: initial" class=" list-paddingleft-2">
      <li>
        <pre class="brush:bash;toolbar:false">chmod&nbsp;g+rwx&nbsp;test&nbsp;&nbsp;&nbsp;#&nbsp;给g用户添加rwx三种权限&nbsp;&nbsp;
chmod&nbsp;o-wx&nbsp;test&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;删除o用户的wx权限&nbsp;&nbsp;
chmod&nbsp;+x&nbsp;&nbsp;haha.sh&nbsp;&nbsp;#&nbsp;给脚本haha.sh添加执行权限，前面没有指定用户，则默认是a，全部。</pre>

        <p>
          &nbsp; &nbsp;
        </p>
      </li>
    </ol>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      设置SUID位
    </p>

    <ol start="1" style="padding: 5px 0px;border: none;position: relative;list-style-position: initial;list-style-image: initial" class=" list-paddingleft-2">
      <li>
        <pre class="brush:bash;toolbar:false">chmod&nbsp;u+s&nbsp;a.out&nbsp;&nbsp;
-rwsr-xr-x&nbsp;1&nbsp;guanxin&nbsp;root&nbsp;7577&nbsp;12-26&nbsp;12:31&nbsp;a.out</pre>

        <p>
          &nbsp; &nbsp;
        </p>
      </li>
    </ol>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      <span style="font-size: 14px">可以看到</span><span style="font-size: 14px">owner</span><span style="font-size: 14px">对应的</span><span style="font-size: 14px">x</span><span style="font-size: 14px">权限变成了小写</span><span style="font-size: 14px">s</span><span style="font-size: 14px">，如上所述，设置</span><span style="font-size: 14px">SUID</span><span style="font-size: 14px">位就会使运行这个文件时获得</span><span style="font-size: 14px">guanxin</span><span style="font-size: 14px">用户的权限。当然执行完，权限也就消失了。</span>
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      一般这功能用于临时获取root权限，例如执行/usr/bin/passwd修改密码，需要访问修改/etc/passswd文件，而这个文件对其它用户而言是只读的，那我想修改自己的密码怎么办？答案是设置SUID位。
    </p>

    <pre class="brush:plain;toolbar:false">-rwsr-xr-x1&nbsp;root&nbsp;root&nbsp;27768&nbsp;2007-01-07&nbsp;/usr/bin/passwd</pre>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      可以看到这个文件的拥有者是root，所以执行时会获得root权限。
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      对于文件夹设置SUID位是没有任何作用的，对于没有x权限的文件设置SUID位，结果是一个大写S标志。
    </p>

    <ol start="1" style="padding: 5px 0px;border: none;position: relative;list-style-position: initial;list-style-image: initial" class=" list-paddingleft-2">
      <li>
        <pre class="brush:plain;toolbar:false">-rwS------&nbsp;1&nbsp;guanxin&nbsp;root&nbsp;15&nbsp;12-28&nbsp;10:57&nbsp;owner.txt&nbsp;&nbsp;
echo&nbsp;haha&nbsp;&gt;&gt;owner.txt&nbsp;&nbsp;&nbsp;#&nbsp;如果对owner.txt进行修改，那么它会自动丢失大S属性。&nbsp;&nbsp;
-rw-------&nbsp;1&nbsp;guanxin&nbsp;root&nbsp;20&nbsp;12-28&nbsp;11:26&nbsp;owner.txt</pre>

        <p>
          &nbsp; &nbsp;
        </p>
      </li>
    </ol>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      <span style="font-size: 14px">鉴于SUID位容易有安全隐患，万一有人无意中设置被别人钻空子（使得运行特权进程后，不正常结束，通过缓冲区溢出或代码注入继续运行特权程序），所以许多操作系统会忽略对脚本的SUID设置。</span>
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      设置SGID位
    </p>

    <ol start="1" style="padding: 5px 0px;border: none;position: relative;list-style-position: initial;list-style-image: initial" class=" list-paddingleft-2">
      <li>
        <pre class="brush:bash;toolbar:false">chmod&nbsp;g+s&nbsp;a.out&nbsp;&nbsp;
-rwsr-sr-x&nbsp;1&nbsp;guanxin&nbsp;root&nbsp;7577&nbsp;12-26&nbsp;12:31&nbsp;a.out</pre>

        <p>
          &nbsp; &nbsp;
        </p>
      </li>
    </ol>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      这时g用户对应的x位变成了小s，它表示文件启动执行时，会获得root组的组权限（设置egid）。同样也是如果对不带x权限的文件设置，会显示大写S。&nbsp;
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      如果对文件夹设置SGID位，表示在此文件夹中创建的文件，组都是和文件夹相同。
    </p>

    <ol start="1" style="padding: 5px 0px;border: none;position: relative;list-style-position: initial;list-style-image: initial" class=" list-paddingleft-2">
      <li>
        <pre class="brush:plain;toolbar:false">drwxrwxrwx&nbsp;&nbsp;2&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4096&nbsp;12-28&nbsp;11:43&nbsp;test&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;没对test文件夹设置SGID位&nbsp;&nbsp;
-rw-rw-r--&nbsp;1&nbsp;guanxin2&nbsp;guanxin2&nbsp;&nbsp;0&nbsp;12-28&nbsp;11:43&nbsp;haha3.txt&nbsp;&nbsp;&nbsp;#&nbsp;用户guanxin2创建的文件是创建者的名字和组&nbsp;&nbsp;
chmod&nbsp;g+s&nbsp;test&nbsp;&nbsp;
drwxrwsrwx&nbsp;&nbsp;2&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4096&nbsp;12-28&nbsp;11:43&nbsp;test&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;对test文件夹设置SGID位&nbsp;&nbsp;
-rw-rw-r--&nbsp;1&nbsp;guanxin2&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;0&nbsp;12-28&nbsp;11:45&nbsp;haha4.txt&nbsp;&nbsp;&nbsp;#&nbsp;用户guanxin2创建的文件拥有者还是guanxin2，但是组变成了root</pre>

        <p>
          &nbsp; &nbsp;
        </p>
      </li>
    </ol>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      设置sticky bit
    </p>

    <ol start="1" style="padding: 5px 0px;border: none;position: relative;list-style-position: initial;list-style-image: initial" class=" list-paddingleft-2">
      <li>
        <pre class="brush:bash;toolbar:false">chmod&nbsp;+t&nbsp;&nbsp;a.out</pre>

        <p>
          &nbsp; &nbsp;
        </p>
      </li>
    </ol>

    <p>
      <a style="color: rgb(106, 57, 6);" name="t3"></a>4&nbsp;进程权限修改（Linux系统C函数）
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      C函数可以用chmod/fchmod来修改文件属性。下面两个表格是在进程运行中如何修改uid和gid从而获得不同的权限（空白表示值不变）。
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <table cellpadding="0" cellspacing="0" valign="top">
      <tr class="firstRow">
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="102">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="429">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            before call
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            ruid
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="28">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="24">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            suid
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="27">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            fsuid
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="92">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            setuid( e )
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="429">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid != 0 (e是ruid、euid、suid之一)
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="28">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="24">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="27">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="92">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="419">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid == 0(注意，这种情况下，进程无法回到特权级，因为suid也变了。)
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="7">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="92">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            seteuid( e )
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="419">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid != 0 (e是ruid、euid、suid之一)
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="7">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="92">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="419">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid == 0
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="7">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="92">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            setreuid( r, e )
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="419">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid != 0 (r是ruid、euid之一，e是ruid、euid、suid之一)
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="7">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            r
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="92">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="419">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid == 0
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="7">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            r
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="92">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            setresuid( r, e, s )
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="419">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid != 0 (r，e, s是ruid、euid、suid之一)
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="7">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            r
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            s
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="92">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="419">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid == 0
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="7">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            r
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            s
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="92">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            setfsuid( f )
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="419">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid != 0 (f是ruid、euid、suid、fsuid之一)
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="7">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            f
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="92">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="419">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid == 0
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="7">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            f
          </p>
        </td>
      </tr>
    </table>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      &nbsp;
    </p>

    <table cellpadding="0" cellspacing="0" valign="top">
      <tr class="firstRow">
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="100">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="423">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            before call
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="25">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            rgid
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="28">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            egid
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="27">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            sgid
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="24">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            fsgid
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="100">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            setgid( e )
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="423">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid != 0 (e是rgid、egid、sgid之一)
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="25">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="28">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="27">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="24">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="90">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="413">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid == 0
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="15">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="90">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            setegid( e )
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="413">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid != 0 (e是rgid、egid、sgid之一)
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="15">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="90">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="413">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid == 0
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="15">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="90">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            setregid( r, e )
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="413">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid != 0 (r是rgid、egid之一，e是rgid、egid、sgid之一)
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="15">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            r
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px; word-break: break-all;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="90">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="413">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid == 0
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="15">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            r
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="90">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            setresgid( r, e, s )
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="413">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid != 0 (r，e, s是rgid、egid、sgid之一)
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="15">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            r
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            s
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="90">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="413">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid == 0
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="15">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            r
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            s
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            e
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="90">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            setfsgid( f )
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="413">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid != 0 (f是rgid、egid、sgid、fsgid之一)
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="15">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            f
          </p>
        </td>
      </tr>

      <tr>
        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="90">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="413">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            euid == 0
          </p>
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="15">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="18">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px;" width="17">
        </td>

        <td style="border: 1px solid rgb(163, 163, 163); vertical-align: top; padding: 5px; word-break: break-all;" width="14">
          <p style=";font-family: 微软雅黑;font-size: 14px">
            f
          </p>
        </td>
      </tr>
    </table>

    <h2 style=";padding: 0px">
      <a style="color: rgb(106, 57, 6)" name="t4"></a>参考文档
    </h2>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      <a href="http://en.wikipedia.org/wiki/Sticky_bit" style="color: rgb(106, 57, 6)">http://en.wikipedia.org/wiki/Sticky_bit</a>
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      <a href="http://en.wikipedia.org/wiki/User_id" style="color: rgb(106, 57, 6)">http://en.wikipedia.org/wiki/User_id</a>
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      <a href="http://en.wikipedia.org/wiki/File_system_permissions" style="color: rgb(106, 57, 6)">http://en.wikipedia.org/wiki/File_system_permissions</a>
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      GNUcoreutils.info
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      <a href="http://blog.sina.com.cn/s/blog_4c23939b0100q4zl.html" style="color: rgb(106, 57, 6)">http://blog.sina.com.cn/s/blog_4c23939b0100q4zl.html</a>
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      <a href="http://www.2cto.com/os/201208/147429.html" style="color: rgb(106, 57, 6)">http://www.2cto.com/os/201208/147429.html</a>
    </p>

    <p style="margin: 0in 0in 0in 0.375in;font-family: 微软雅黑;font-size: 14px">
      <a href="http://en.wikipedia.org/wiki/Sgid" style="color: rgb(106, 57, 6)">http://en.wikipedia.org/wiki/Sgid</a>
    </p>

    <p>
    </p>

    <p>
      原文链接：<a href="http://blog.csdn.net/gogdizzy/article/details/8447204" target="_self">http://blog.csdn.net/gogdizzy/article/details/8447204</a>
    </p>
