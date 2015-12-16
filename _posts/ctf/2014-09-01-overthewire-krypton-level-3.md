---
title: OverTheWire Krypton Level 2 → Level 3
author: Ripples
layout: post
views:
  - 276
categories:
  - CTF
tags:
  - decryption
  - krypton
  - wargame
  - WriteUp
  - 频率分析
---
<p style="text-indent: 2em;">
  题目传送门<a href="http://overthewire.org/wargames/krypton/krypton3.html" target="_blank">在此</a>。
</p>

<p style="text-indent: 2em;">
  这一关终于不想前几关那么水了，不过也不难。题目给了3个加密后文本用来破解，然后明确表示是频率分析，于是乎，就按照要求来吧。
</p>

<p style="text-indent: 2em;">
  首先找到频率表：
</p>

<!--more-->




<table cellspacing="0" cellpadding="0" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="width: 735px;" interlaced="enabled">
  <tr removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" class="ue-table-interlace-color-single firstRow">
    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        A
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        B
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        C
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        D
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        E
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        F
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        G
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        H
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        I
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        J
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>
  </tr>

  <tr removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" class="ue-table-interlace-color-double">
    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.082
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.015
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.028
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.043
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.127
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.022
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.020
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.061
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.070
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.002
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>
  </tr>

  <tr removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" class="ue-table-interlace-color-single">
    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        K
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        L
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        M
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        N
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        O
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        P
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        Q
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        R
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        S
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        T
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>
  </tr>

  <tr removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" class="ue-table-interlace-color-double">
    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.008
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.040
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.024
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.067
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.075
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.019
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.001
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.060
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.063
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.091
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>
  </tr>

  <tr removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" class="ue-table-interlace-color-single">
    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        U
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        V
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        W
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        X
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        Y
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        Z
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
    </td>
  </tr>

  <tr removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" class="ue-table-interlace-color-double">
    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.028
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.010
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.023
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.001
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.020
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
      <p style="margin-bottom: 10px;">
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
        0.001
      </p>

      <p style="margin-bottom: 10px; text-indent: 2em;">
      </p>
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="58" removechild="function MyRC(arg1){var self = this;if (self.removeAttribute)self.removeAttribute(&quot;removeChild&quot;);var result = self[&quot;removeChild&quot;](arg1);self[&quot;removeChild&quot;] = arguments.callee; if(arg1.clearAttributes)arg1.clearAttributes();if(arg1.onclick)arg1.onclick=null;if(arg1.onmousemove)arg1.onmousemove=null;if(arg1.onmouseover)arg1.onmouseover=null;if(arg1.ondblclick)arg1.ondblclick=null;if(arg1.onmouseenter)arg1.onmouseenter=null;if(arg1.onmouseleave)arg1.onmouseleave=null;return result;}" style="border-width: 1px; border-style: solid;">
    </td>
  </tr>
</table>

<p style="text-indent: 2em;">
  字母可分为五组：
</p>

<ul class=" list-paddingleft-2" style="list-style-type: disc;">
  <li>
    <p style="text-indent: 2em;">
      E：0.127
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      TAOINSHR：0.06~0.09
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      DL：0.04
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      CUMWFGYPB：0.015~0.023
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      VKJXQZ：小于0.01
    </p>
  </li>
</ul>

<p style="text-indent: 2em;">
  最常见的两字母组合，依照出现次数递减的顺序排列：TH、HE、IN、ER、AN、RE、DE、ON、ES、ST、EN、AT、TO、NT、HA、ND、OU、EA、NG、AS、OR、TI、IS、ET、IT、AR、TE、SE、HI、OF。
</p>

<p style="text-indent: 2em;">
  最常见的三字母组合，依照出现次数递减的顺序排列：THE、ING、AND、HER、ERE、ENT、THA、NTH、WAS、ETH、FOR、DTH。
</p>

<p style="text-indent: 2em;">
  频率表准备好了然后就可以开工了，首先对给出的文本统计频率：
</p>

<pre class="brush:python;toolbar:false">l&nbsp;=&nbsp;len(s)
for&nbsp;j&nbsp;in&nbsp;xrange(3):
&nbsp;&nbsp;&nbsp;&nbsp;d&nbsp;=&nbsp;{}
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(l&nbsp;-&nbsp;j):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;c&nbsp;=&nbsp;s[i:i+j+1]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;c&nbsp;in&nbsp;d:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;d[c]&nbsp;+=&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;d[c]&nbsp;=&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;ans&nbsp;=&nbsp;sorted(d.items(),&nbsp;key=lambda&nbsp;d:d[1])
&nbsp;&nbsp;&nbsp;&nbsp;fp.write(&#39;n&#39;.join([str(one)&nbsp;for&nbsp;one&nbsp;in&nbsp;ans])&nbsp;+&nbsp;&#39;n&#39;)</pre>

<p style="text-indent: 2em;">
  分别对三个文本统计之后，感觉单字母频率差不多，于是三个放到一起又统计了一遍，按照三个一起的统计结果来做。
</p>

<p style="text-indent: 2em;">
  首先看最高的几个三字母：
</p>

<pre class="brush:plain;toolbar:false">(&#39;JDQ&#39;,&nbsp;15)
(&#39;CBG&#39;,&nbsp;15)
(&#39;JSN&#39;,&nbsp;16)
(&#39;CGE&#39;,&nbsp;16)
(&#39;SNS&#39;,&nbsp;19)
(&#39;DCU&#39;,&nbsp;19)
(&#39;DSN&#39;,&nbsp;22)
(&#39;SQN&#39;,&nbsp;23)
(&#39;QGW&#39;,&nbsp;27)
(&#39;JDS&#39;,&nbsp;61)</pre>

<p style="text-indent: 2em;">
  JDS明显高出其它很多，似乎就是the，然后又看到SNS，那基本就肯定JDS是the、SNS是ere了。
</p>

<p style="text-indent: 2em;">
  然后QGW我首先猜测是ing，但替换了一下感觉不太对劲，然后SQN对应eir似乎也不太说的过去，于是猜测QGW对应and，这样SQN对应ear也似乎稍微好点。
</p>

<p style="text-indent: 2em;">
  然后查看双字母：
</p>

<pre class="brush:plain;toolbar:false">(&#39;QG&#39;,&nbsp;46)
(&#39;JS&#39;,&nbsp;47)
(&#39;UJ&#39;,&nbsp;47)
(&#39;SQ&#39;,&nbsp;48)
(&#39;SW&#39;,&nbsp;52)
(&#39;DQ&#39;,&nbsp;52)
(&#39;CG&#39;,&nbsp;53)
(&#39;QN&#39;,&nbsp;54)
(&#39;NS&#39;,&nbsp;54)
(&#39;SU&#39;,&nbsp;63)
(&#39;SN&#39;,&nbsp;68)
(&#39;DS&#39;,&nbsp;83)
(&#39;JD&#39;,&nbsp;96)</pre>

<p style="text-indent: 2em;">
  由SU、UJ感觉U是s。
</p>

<p style="text-indent: 2em;">
  接着我在文本中发现了CnsteadBXthe，感觉就是instead of the了。
</p>

<p style="text-indent: 2em;">
  这样，观察三字母，CGE就应该是ing了。
</p>

<p style="text-indent: 2em;">
  查看单字母：
</p>

<pre class="brush:plain;toolbar:false">(&#39;W&#39;,&nbsp;129)d
(&#39;V&#39;,&nbsp;130)
(&#39;Z&#39;,&nbsp;132)
(&#39;D&#39;,&nbsp;210)h
(&#39;C&#39;,&nbsp;227)i
(&#39;G&#39;,&nbsp;227)n
(&#39;N&#39;,&nbsp;240)r
(&#39;B&#39;,&nbsp;246)o
(&#39;U&#39;,&nbsp;257)s
(&#39;J&#39;,&nbsp;301)t
(&#39;Q&#39;,&nbsp;340)a
(&#39;S&#39;,&nbsp;456)e</pre>

<p style="text-indent: 2em;">
  对比单字母频率表会发现，最高频的字母ETAOINSHR正好对应上了这边的频率，紧接着的DL中，L应该就是对应着VZ当中的一个了。
</p>

<p style="text-indent: 2em;">
  在三字母和双字母中查找V和Z，发现似乎只有在双字母的中等频率中能见到他们，如下：
</p>

<pre class="brush:plain;toolbar:false">(&#39;ZS&#39;,&nbsp;24)
(&#39;VQ&#39;,&nbsp;24)
(&#39;UQ&#39;,&nbsp;24)
(&#39;NJ&#39;,&nbsp;24)
(&#39;ZB&#39;,&nbsp;25)
(&#39;ZQ&#39;,&nbsp;25)
(&#39;QV&#39;,&nbsp;25)</pre>

<p style="text-indent: 2em;">
  首先尝试将Z替换成l，然后查看替换后的文本，感觉很多地方似乎看起来不太像个单词了，于是乎改成将V替换成l。
</p>

<p style="text-indent: 2em;">
  查看第二个文本的开头发现althoMghnoattendanZereZordsfortheYeriods，就应该是although no attendance records for the，带回去看频率也挺符合。
</p>

<p style="text-indent: 2em;">
  这时候，文本基本已经解密的差不多了，但由于本人英语太烂，实在不想看了，于是乎，将第二个文本的开头<span style="text-indent: 32px;">（although no attendance records for theYeriodsur）拿到搜索引擎中一搜，就发现是关于莎士比亚的一段话，于是得以解决～～～</span>
</p>

<p style="text-indent: 2em;">
  对应表如下（密文 -> 明文）：
</p>

<pre class="brush:python;toolbar:false">[(&#39;A&#39;,&nbsp;&#39;b&#39;),&nbsp;(&#39;B&#39;,&nbsp;&#39;o&#39;),&nbsp;(&#39;C&#39;,&nbsp;&#39;i&#39;),&nbsp;(&#39;D&#39;,&nbsp;&#39;h&#39;),&nbsp;(&#39;E&#39;,&nbsp;&#39;g&#39;),&nbsp;(&#39;F&#39;,&nbsp;&#39;k&#39;),&nbsp;(&#39;G&#39;,&nbsp;&#39;n&#39;),&nbsp;(&#39;H&#39;,&nbsp;&#39;q&#39;),&nbsp;(&#39;I&#39;,&nbsp;&#39;v&#39;),&nbsp;(&#39;J&#39;,&nbsp;&#39;t&#39;),&nbsp;(&#39;K&#39;,&nbsp;&#39;w&#39;),&nbsp;(&#39;L&#39;,&nbsp;&#39;y&#39;),&nbsp;(&#39;M&#39;,&nbsp;&#39;u&#39;),&nbsp;(&#39;N&#39;,&nbsp;&#39;r&#39;),&nbsp;(&#39;O&#39;,&nbsp;&#39;x&#39;),&nbsp;(&#39;P&#39;,&nbsp;&#39;z&#39;),&nbsp;(&#39;Q&#39;,&nbsp;&#39;a&#39;),&nbsp;(&#39;R&#39;,&nbsp;&#39;j&#39;),&nbsp;(&#39;S&#39;,&nbsp;&#39;e&#39;),&nbsp;(&#39;T&#39;,&nbsp;&#39;m&#39;),&nbsp;(&#39;U&#39;,&nbsp;&#39;s&#39;),&nbsp;(&#39;V&#39;,&nbsp;&#39;l&#39;),&nbsp;(&#39;W&#39;,&nbsp;&#39;d&#39;),&nbsp;(&#39;X&#39;,&nbsp;&#39;f&#39;),&nbsp;(&#39;Y&#39;,&nbsp;&#39;p&#39;),&nbsp;(&#39;Z&#39;,&nbsp;&#39;c&#39;)]</pre>

<p style="text-indent: 2em;">
  然后便可以解得密码～～～
</p>

<p style="text-indent: 2em;">
  不得不说，这关简直太符合频率分析，按照频率来，一猜一个准～～～
</p>
