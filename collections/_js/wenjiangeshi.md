---
layout: post
title: MIME, Base64, Blob
---
## MIME
"Multipurpose Internet Mail Extensions"，中译为"多用途互联网邮件扩展"，指的是一系列的电子邮件技术规范。
早期的邮件只能只能使用ASCII字符，不能发图片和附件。

所以用`Content-Type`来标记文件格式。它包含了主要类型（primary type）和次要类型（subtype）两个部分，两者之间用"/"分割。主要类型有9种，分别是application、audio、example、image、message、model、multipart、text、video。

如果信息的主要类型是"text"，那么还必须指明编码类型"charset"，缺省值是ASCII，其他可能值有"ISO-8859-1"、"UTF-8"、"GB2312"等等。

```
text/plain; charset="ISO-8859-1"：纯文本，文件扩展名.txt
text/html：HTML文本，文件扩展名.htm和.html
image/jpeg：jpeg格式的图片，文件扩展名.jpg
image/gif：GIF格式的图片，文件扩展名.gif
audio/x-wave：WAVE格式的音频，文件扩展名.wav
audio/mpeg：MP3格式的音频，文件扩展名.mp3
video/mpeg：MPEG格式的视频，文件扩展名.mpg
application/zip：PK-ZIP格式的压缩文件，文件扩展名.zip
```

MIME用`Content-transfer-encoding`将8位的非英语字符转化为7位的ASCII字符。
可选值有"7bit"、"8bit"、"binary"、"quoted-printable"和"base64"----其中"7bit"是缺省值，即不用转化的ASCII字符。
真正常用是"quoted-printable"和"base64"两种。

### Quoted-printable
用于ACSII文本中夹杂少量非ASCII码字符的情况，不适合于转换纯二进制文件。

它将每一个8位的字节，转换为3个字符，第一个字符是"="号，这是固定不变的。后面二个字符是二个十六进制数，分别代表了这个字节前四位和后四位的数值。

举例来说，ASCII码中"换页键"（form feed）是12，二进制形式是00001100，写成十六进制就是0C，因此它的编码值为"=0C"。"="号的ASCII值是61，二进制形式是00111101，因为它的编码值是"=3D"。除了可打印的ASCII码以外，所有其他字符都必须用这种方式进行转换。

## base64
选出64个字符----小写字母a-z、大写字母A-Z、数字0-9、符号"+"、"/"（再加上作为垫字的"="，实际上是65个字符）----作为一个基本字符集。然后，其他所有符号都转换成这个字符集中的字符。
**好处**
1. 所有的二进制文件，都可以因此转化为可打印的文本编码，使用文本软件进行编辑；
2. 能够对文本进行简单的加密。

**原理**
Base64将三个字节转化成四个字节，因此Base64编码后的文本，会比原文本大出三分之一左右。

<table border="1"><tbody><tr><td style="padding:0px;">Text content</td><td style="padding:0px;" colspan="8" align="center"><b>M</b></td><td style="padding:0px;" colspan="8" align="center"><b>a</b></td><td style="padding:0px;" colspan="8" align="center"><b>n</b></td></tr><tr><td style="padding:0px;">ASCII</td><td style="padding:0px;" colspan="8" align="center">77</td><td style="padding:0px;" colspan="8" align="center">97</td><td style="padding:0px;" colspan="8" align="center">110</td></tr><tr><td style="padding:0px;">Bit pattern</td><td style="padding:0px;">0</td><td style="padding:0px;">1</td><td style="padding:0px;">0</td><td style="padding:0px;">0</td><td style="padding:0px;">1</td><td style="padding:0px;">1</td><td style="padding:0px;">0</td><td style="padding:0px;">1</td><td style="padding:0px;">0</td><td style="padding:0px;">1</td><td style="padding:0px;">1</td><td style="padding:0px;">0</td><td style="padding:0px;">0</td><td style="padding:0px;">0</td><td style="padding:0px;">0</td><td style="padding:0px;">1</td><td style="padding:0px;">0</td><td style="padding:0px;">1</td><td style="padding:0px;">1</td><td style="padding:0px;">0</td><td style="padding:0px;">1</td><td style="padding:0px;">1</td><td style="padding:0px;">1</td><td style="padding:0px;">0</td></tr><tr><td style="padding:0px;">Index</td><td style="padding:0px;" colspan="6" align="center">19</td><td style="padding:0px;" colspan="6" align="center">22</td><td style="padding:0px;" colspan="6" align="center">5</td><td style="padding:0px;" colspan="6" align="center">46</td></tr><tr><td style="padding:0px;">Base64-Encoded</td><td style="padding:0px;" colspan="6" align="center"><b>T</b></td><td style="padding:0px;" colspan="6" align="center"><b>W</b></td><td style="padding:0px;" colspan="6" align="center"><b>F</b></td><td style="padding:0px;" colspan="6" align="center"><b>u</b></td></tr></tbody></table>

第一步，"M"、"a"、"n"的ASCII值分别是77、97、110，对应的二进制值是01001101、01100001、01101110，将它们连成一个24位的二进制字符串010011010110000101101110。

第二步，将这个24位的二进制字符串分成4组，每组6个二进制位：010011、010110、000101、101110。

第三步，在每组前面加两个00，扩展成32个二进制位，即四个字节：00010011、00010110、00000101、00101110。它们的十进制值分别是19、22、5、46。

第四步，根据上表，得到每个值对应Base64编码，即T、W、F、u。

**两个字节时**
将这二个字节的一共16个二进制位，按照上面的规则，转成三组，最后一组除了前面加两个0以外，后面也要加两个0。这样得到一个三位的Base64编码，再在末尾补上一个"="号。

0100110101100001 = 010011 + 010110 + 0001
                 = 00010011 + 00010110 + 00010000
                 = T + W + E
                 = TWE=

**一个字节时**
这一个字节的8个二进制位，按照上面的规则转成二组，最后一组除了前面加二个0以外，后面再加4个0。这样得到一个二位的Base64编码，再在末尾补上两个"="号。
01001101 = 010011 + 01
         = 00010011 + 00010000
         = T + Q
         = TQ==

**中文字符**
汉字本身可以有多种编码，比如gb2312、utf-8、gbk等等，每一种编码的Base64对应值都不一样。下面的例子以utf-8为例。
"严"的utf-8编码为E4B8A5，写成二进制就是三字节的"11100100 10111000 10100101"，转换成四组一共32位的二进制值"00111001 00001011 00100010 00100101"，相应的十进制数为57、11、34、37，它们对应的Base64值就为5、L、i、l。

**编码解码**
Javascript内部的字符串，都以utf-16的形式进行保存，因此编码的时候，我们首先必须将utf-8的值转成utf-16再编码，解码的时候，则是解码后还需要将utf-16的值转回成utf-8。
[http://www.ruanyifeng.com/blog/2008/06/base64.html](http://www.ruanyifeng.com/blog/2008/06/base64.html)