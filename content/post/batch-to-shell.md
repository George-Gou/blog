---
title: "Windows bat 脚本移植到 linux bash "
date: 2021-11-13T20:37:13+08:00
lastmod: 2021-11-13T20:37:13+08:00
draft: false
keywords: []
description: ""
tags: []
categories: []
author: "Gravpher"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true 
toc: true 
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---
本文系统地介绍 Windows bat 和 Linux bash 脚本的区别以及常用的脚本命令。


**注意：本文的测试平台分别为： Win10、Cetnos 7**





```bat
rem bat测试脚本：
set eqfile=eq.dat
set topodata="earth_relief_06m.grd"
set SIZE=w15c/0.25c


set R=-R70/15.5/138/54.5r
set J=-JM105/35/7.72i
set BLOCK=Data\CN-block-L2.gmt
set BLOCKONE=Data\CN-block-L1.gmt
set BOUNDARY=..\Data\xsp.xyz 
set L=..\Data\nine.dat

::set FONT_LABEL=20p,Helvetica,black
::set FONT_ANNOT_PRIMARY=20p,Helvetica,black
gmt set FONT_ANNOT_PRIMARY +25p
gmt set FONT_LABEL +20p
gmt set FONT_LABEL 8p,39 MAP_LABEL_OFFSET 4p
::gmt grdcut %topodata% -R70/135/15/55 -GcutTopo.grd
::gmt grdgradient cutTopo.grd -Ne0.7 -A50 -GcutTopo_i.grd
::gmt grd2cpt cutTopo.grd -Cglobe -T-20000/20000/400 -Z -D > colorTopo.cpt

gmt begin 8_8_1 png
REM 绘制底图
::gmt set FORMAT_GEO_MAP=ddd:mm:ssF
::gmt basemap %R% %J% -Bf5a10 -BWesN
::gmt grdimage cutTopo.grd -IcutTopo_i.grd -CcolorTopo.cpt -Q
::gmt coast -Dh -W1/0.2p -I1/0.25p -N1/0.5p
::gmt coast -JM105/35/6.5i %R% -G244/243/239 -S167/194/223 -B10f5g10  -Lg85/17.5+c17.5+w800k+f+u+l'比例尺'
::gmt plot CN-border-La.dat -W0.4p,black50


gmt psxy %R% %J% -T 
gmt coast  %R% %J% -Gwhite -S174/216/230 -Ba10g10 -N1 -W1 -A10000  --FONT_ANNOT_PRIMARY=20p 
gmt basemap -R70/15.5/138/54.5r -Lfx2/2.7/-38/500+u --FONT_ANNOT_PRIMARY=15p

REM 绘制colorbar
::gmt colorbar -DjCB+w7i/0.15i+o0/-0.5i+m -CcolorTopo.cpt -Bxa2000f400+l"Elevation/m" -G-8000/8000
::gmt colorbar -DjCB+w18c/0.3c+o0/-2.5c+h -BWSEN -Bxa2000f400+l"Elevation/m" -G-8000/8000

gmt makecpt -Chaxby -T-200/200/1 -H -Icz > gravity.cpt

gmt surface 8_8_1.txt %R% -I1 -Graws0.nc  -T0.25
gmt psclip shengjie.xyz %R% %J%
gmt grdview raws0.nc %R% %J% -Cgravity.cpt -Qs 
gmt psclip -C 
::gmt colorbar -CIgravity.cpt -Dn0.5/0.7+jCM+%SIZE%+h+e+n -B+l"-uGal"
gmt psscale -Cgravity.cpt -Dx10c/-1.5c+w17c/0.4c+jTC+h -Bxaf+2"gravity change" -By+l\265Gal    -Ac -E 
gmt psxy %BOUNDARY% %R% %J% -W0.8p
gmt psxy %L% %R% %J%

gmt plot -R1/10/1/10 -JX20c -T
::gmt colorbar -Cgravity2.cpt -Dn0.5/0.7+jCM+w15c/0.25c+h+e+n -B+l"-Icz"
::gmt colorbar -DjMR+w2c/0.5c+o-1c/0c -Cgravity2.cpt -Bxa2000f400+l"uGal" -G0/4.8

::gmt psscale -Cgravity2.cpt -DjBC+w20c/0.5c+o0c/-2c+m  -Ac -E 

::gmt psscale -Cgravity.cpt -DjMR+w18c/0.5c+o-1c/0c+m  -Ac -E 

::gmt psmeca eq.txt %R% %J% -Ewhite -Gred -Sm0.3 
::gmt text text.txt %R% %J% -F+f+a+j 

gmt psxy gPhone.txt %R% %J% -i0,1 -St0.5 -G0/0/255
::gmt psxy SG.txt %R% %J% -i0,1 -St0.5 -G255/0/0 
gmt psxy GS15.txt %R% %J% -i0,1 -St0.5c -G010/150/200
::gmt pstext station_name.txt %R% %J% -F+f+a+j 

gmt psxy  %R% %J% %BLOCK% -W1.2p,white

gmt psxy  %R% %J% %BLOCKONE% -W2p,white

::gmt psxy %R% %J% fault.dat  -W3p,blue 
::echo G 0.01i >legend.txt

::echo C 0/0/0 >>legend.txt
::echo S 0.25c t 0.3c blue 0.25p 0.6c PET/gPhone  >>legend.txt
::echo C 0/0/0 >>legend.txt
::echo S 0.25c t 0.3c cyan 0.25p 0.6c GS15 >>legend.txt
::echo C 0/0/0 >>legend.txt
::echo S 0.25c t 0.3c red 0.25p 0.6c  SG >>legend.txt
::gmt legend legend.txt -DjBL+w1.5i+l1.2+o0 -F+g229+p0.25p
::gmt legend legend.txt -DjBL+w1.5i+l1.2+o0 -F+g229+p0.25p

gmt pstext dategravity.txt %R% %J% -F+f+a+j 

REM 南海
gmt gmtset MAP_FRAME_TYPE plain 
gmt gmtset MAP_FRAME_PEN 0.5p
set R1=-R106/121/3/24
gmt psbasemap %R1% -JM1.12i -B50  -X6.59i 
gmt pscoast  %R1%  -N1  -W1 -JM  -B0 -Gwhite -S174/216/230 
gmt psxy  %BOUNDARY% %R1% -JM -W0.5p  
echo 107 4.3 SouthChinaSea|gmt  pstext %R1% -JM -F+f9,37+a0+jLM -Gwhite
gmt psxy %L% %R1% -JM -W1p 
::gmt psscale -D6.5i/2i/7.5c/0.75c+h -CSNM.cpt -O -K >> %ps%
gmt psxy  %R% %J% -T  
gmt end

pause
```



```shell
topodata="earth_relief_06m.grd"
SIZE=w15c/0.25c
###########5
R=-R70/15.5/138/54.5r
J=-JM105/35/7.72i
BLOCK=Data/CN-block-L2.gmt
BLOCKONE=Data/CN-block-L1.gmt
BOUNDARY=Data/xsp.xyz
L=Data/nine.dat
###########12
gmt set MAP_FRAME_TYPE fancy
gmt set FONT_ANNOT_PRIMARY +25p
gmt set FONT_LABEL +20p
gmt set FONT_LABEL 8p,39 MAP_LABEL_OFFSET 4p
###########绘制底图16
gmt begin gravity_change png
	#绘制底图
	###########19
	gmt psxy $R $J -T
	gmt coast  $R $J -Gwhite -S174/216/230 -Ba10g10 -N1 -W1 -A10000  --FONT_ANNOT_PRIMARY=20p
	gmt basemap -R70/15.5/138/54.5r -Lfx2/2.7/-38/500+u --FONT_ANNOT_PRIMARY=15p
	###########24
	gmt makecpt -Chaxby -T-200/200/0.1 -H -Icz > gravity2.cpt
	gmt surface 8_8_1.txt $R -I1 -Graws0.nc  -T0.25
	gmt psclip shengjie.xyz $R $J
	gmt grdview raws0.nc $R $J -Cgravity.cpt -Qs
	gmt psclip -C
	gmt psscale -Cgravity.cpt -Dx10c/-1.5c+w17c/0.4c+jTC+h -Bxaf+10 -By+l\\265Gal    -Ac -E
	gmt psxy $BOUNDARY $R $J -W0.8p
	gmt psxy $L $R $J
	###########32
	gmt plot -R1/10/1/10 -JX20c -T
	gmt psxy gPhone.txt $R $J -i0,1 -St0.5 -G0/0/255
	gmt psxy GS15.txt $R $J -i0,1 -St0.5c -G010/150/200
	###########36
	gmt psxy  $R $J $BLOCK -W1.2p,white
	gmt psxy  $R $J $BLOCKONE -W2p,white
	gmt pstext dategravity.txt $R $J -F+f+a+j
	#南海
	gmt gmtset MAP_FRAME_TYPE plain
	gmt gmtset MAP_FRAME_PEN 0.5p
	R1=-R106/121/3/24
	gmt psbasemap $R1 -JM1.12i -B50  -X6.59i
	gmt pscoast  $R1  -N1  -W1 -JM  -B0 -Gwhite -S174/216/230
	gmt psxy  $BOUNDARY $R1 -JM -W0.5p
	echo 107 4.3 SouthChinaSea|gmt  pstext $R1 -JM -F+f9,37+a0+jLM -Gwhite
	gmt psxy $L $R1 -JM -W1p
	gmt psxy  $R $J -T
gmt end show
```

## Shell基础

Shell 是为 shell 编写的脚本程序，shell 是由 c 语言编写的程序，实现了 linux 控制台与 linux 内核的交互。

```shell
#!/bin/bash  #告诉系统使用什么内核执行命令
echo "hello world! "  #输出语句命令
chmod +x ./test.sh #改变脚本执行权限
./test.sh #运行脚本

#shell 定义变量
eqfile=eq.dat
topodata="earth_relief_06m.grd"
SIZE=w15c/0.25c
#bat 定义变量
set var = value

#注意 shell 中的变量引用使用 $VARIABLE, Bat中使用%VARIABLE%
gmt psxy $R $J -T   #shell
gmt psxy %R% %J% -T # bat

# REM为bat的注释符号，linux下为#
# 删除文件 linux 为 rm , bat 为 del
```

为了方便和兼容性，建议大家在 win 下也使用 shell 脚本编程，这样脚本就具有跨平台性，不用转码。win 下建议安装 git，使用 git bash 环境运行。



