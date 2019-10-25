---
layout: post
title: 寫到快不行的CSA作業-Shell Script
tags: 
  - "tech"
---

大四上修了很硬的計算機系統管理，作業二要寫shell script，沒注意到可以用bash等其他shell languages，於是用了最基本的sh來寫，應該是大學裡寫最久的一次個人作業吧:)  

Main menu
===

![](https://i.imgur.com/wabLm85.png)

```
menu(){
	dialog --title "HW2-2_0513404" --menu "SYS INFO" 12 36 6\
	1 "CPU INFO" 2 "MEMORY INFO" 3 "NETWORK INFO" 4 "FILE BROWSER"\
	2> "${RESULT}"
	ch=$( cat $RESULT)
	case $ch in
		1) cpu;;
		2) mem;;
		3) net;;
		4) file;;
	esac
	[ -f $RESULT ] && rm $RESULT
}
```
`dialog --title "A" --menu "B" 12 36 6 "C" "D"`：A標題 B內文 C選項 D選項描述 size(12,36,6)  
`2> "${RESULT}"`：代表將收到的訊息壓進RESULT裡  
`RESULT=/tmp/menu.sh.$$`：在最前面要先宣告RESULT代表什麼文件，把它存放在tmp之下。  
`[ -f $RESULT ] && rm $RESULT`：既然創建了文件，那用完就要刪掉才不會每次執行就累積一個，最後會變一坨垃圾占空間。中括號判斷，`-f`判斷檔案是否存在，存在則回傳True，跑下一段將RESULT刪掉。  

CPU Info
===
![](https://i.imgur.com/ezN1u1W.png)
```
cpu(){
	dialog --title "CPU Info" --msgbox \
	"$(sysctl hw.model hw.machine hw.ncpu)" \
	12 72
	result=$?
	if [ $result -eq 0 ] ; then
		menu
	fi
}
```

Memory Info
===
![](https://i.imgur.com/jHOBh4n.png)
```
mem(){
	while true
	do
		read -es | dialog --title "Memory Info" --gauge \
		"$(sysctl vm.kmem_size vm.kmem_map_size vm.kmem_map_free | \
		cut -d" " -f2 | awk 'NR == 1 {print "Total:"}; \
		NR == 2 {print "Used:"}; NR == 3 {print "Free:"}; \
		{ split( "B KB MB GB TB" , v ); unit=1; \
		while( $1>1024 ){$1/=1024; unit++ }print $1,v[unit] }')" \
		12 72 $z
		read -t1 ans
		if [ $? -eq 0 ]
		then
			break
		fi
	done
	menu
}
```

`dialog --title "" --gauge "" 12 72 z`:gauge 是進度條z代表跑了幾％。  
以下是z的計算：  
```
x=$(sysctl vm.kmem_size | cut -d" " -f2)
y=$(sysctl vm.kmem_map_size | cut -d" " -f2)
z=$((y*100/x))
```

`sysctl vm.kmem_size vm.kmem_map_size vm.kmem_map_free`：分別印出總共多少記憶體、用了多少和還剩多少。  
```
split( "B KB MB GB TB" , v ); unit=1; \
while( $1>1024 ){$1/=1024; unit++ }print $1,v[unit] }'
```
依照適合的單位印出大小，用while迴圈判斷。  

作業要求不斷更新目前記憶體，直到按下ENTER退出為止，因此包了一個While。其實gauge本身就沒有ok鍵，因此要自己想辦法補上退出的方式。  
```
read -t1 ans
if [ $? -eq 0 ]
	then
	break
fi
```
read 鍵盤輸入的值，t1代表暫停1秒，在while中代表1秒更新一次資訊，ans是自訂去接電腦輸入的變數，ENTER代表空字串""，這裡其實不需要ans這個變數，$?代表前一個指令，ENTER代表0，總之這裡判斷只要按下ENTER就跳出迴圈。  

NET Info
===
![](https://i.imgur.com/DbFQ8iT.png)

![](https://i.imgur.com/pTfVBxf.png)

```
net(){
	cc=$(ifconfig -l | wc -w | sed 's/[[:space:]]//g')
	i=0
	set -f
	dialog --title "HW2-2_0513404" --menu "Network Interfaces" 12 36 6 \
	$(ifconfig -l | xargs -n1 | awk '{print $1 " *"}') \
	2> "${NETCH}"
	netch=$( cat $NETCH)
	[ $netch != "" ] && device_info
	menu
}
device_info(){
	dialog --title "" --msgbox \
	"$(ifconfig $netch | grep -E "ether|inet " | xargs -n2 | \
	awk 'NR==1 {print "IPv4___:" $2};NR==2 {print "Netmask:" $2};\
	NR==3 {print "Mac____:" $2}')" 12 72
	[ -f $NETCH ] && rm $NETCH
	net
}
```
`sed 's/[[:space:]]//g'`：把空白的部分刪掉，由於`wc -w`算出的字數不知為啥前面有段空白，因此需要把它刪除才可以得到單純的數字。
這裡同樣使用menu，在製作選項的地方需要把格式弄成兩個一組的，因此用awk排成兩個一組在餵給dialog吃，最後同樣把選擇放到一個文件去再去讀取成變數，接著拿讀取到的變數去device_info這個自訂函數印出相關資訊。
msgbox是最基本的dialog，就呈現一些內文，按下ok可以退出，利用`ifconfig`將網路相關資訊印出來然後用`grep`抓自己想要的部分。  

File Browser
===
![](https://i.imgur.com/I9gAJfX.png)

```
file(){ 
	nn=$(ls -al | awk '{print $9}'|sed '1d' |wc -w)
	dialog --title "" --menu "File Browser:$(pwd)" 25 72 12\
	$(ls -al | awk '{print $9}' | sed '1d' | \
	xargs -I % file --mime-type % | xargs -n2) \
	2>"${FILE}"
	chfile=$(cat $FILE)
	chfile2=$(cat $FILE | sed 's/.$//')
	echo "chfile:$chfile"
	[ -f $FILE ] && rm $FILE
	ft=$(echo "$chfile"| sed 's/.$//' | xargs -I % file --mime-type % |\
	awk '{print $2}'| grep "text")
	ft2=$(echo "$chfile"| sed 's/.$//' | xargs -I @ file --mime-type @ |\
	awk '{print $2}'| grep "inode")
	if [ $chfile != "" ] ; then
		if [ $ft2 != "" ] 
		then
			cdto
		else
			if [ $ft != "" ]
			then
				textinfo
			else
				notextinfo
			fi
		fi
	fi
	menu
}
```
這裡最麻煩的部分，先印出目前目錄下所有檔案，然後使用`xargs`這個指令，他可以指定一個變數，然後讓pipe進來的東西一個一個丟進指定的下一個指令，我這裡是用`xargs -I % file --mime-type %`，這樣每個檔案都會被餵進`file --mime-type`，這會印出檔案的類型。然後就是根據選擇判斷是否為可以編輯的文字檔，我用`grep "text"`來判斷，這樣如果沒有text就會變成空字串。  

![](https://i.imgur.com/nt5jaq3.png)  

![](https://i.imgur.com/ImwcSSC.png)  

```
textinfo(){
	f1=$(echo "$chfile")
	f2=$(echo "$chfile"|sed 's/.$//')
	dialog --title "" --no-label "Edit" --yesno \
	" <File Name>: $(echo "$f1" | sed 's/.$//' | xargs -I % du -sh % |\
	awk '{print $2 }')\n <File Info>: $(echo "$f1"|\
	sed 's/.$//' | xargs -I @ file --mime-type @ | awk '{print $2}')\n\
	<File Size>: $(echo "$f2" | xargs -I size du -sh size|\
	awk '{print $1}')"\
	12 72
	ynresult=$?
	if [ $ynresult -eq 1 ] ; then
	$EDITOR $f2
	elif [ $ynresult -eq 255 ] ; then
	exit 255;
	fi
	file
	
}
```
判斷是文字檔就跑textinfo，這是一個yesno視窗，其實yes/no是可以改文字的，這裡將no改成Edit，接著是判斷輸入，使用者按Yes代表0、No代表2、esc代表255，所以讓按下Edit(本質為No)後呼叫`$EDITOR`，他會以目前環境變數設定的編輯器來開啟指定檔案。作業還需要印出檔案的大小，`du -sh *`是後來查到的一個蠻好用的指令，可以直接印出檔案的大小，`-h`可以用適合的單位印出，就不用像前面那樣使用while。


![](https://i.imgur.com/uiVs27r.png)
```
notextinfo(){
	fl=$(echo "$chfile")
	f2=$(echo "$chfile")
	dialog --title "" --msgbox \
	" <File Name>: $(echo "$fl" | sed 's/.$//' | xargs -I % du -sh % |\
	awk '{print $2 }')\n <File Info>: $(echo "$fl"|\
	sed 's/.$//' | xargs -I @ file --mime-type @ | awk '{print $2}')\n\
	<File Size>: $(echo "$f2" |sed 's/.$//' | xargs -I size du -sh size|\
	awk '{print $1}')"\
	12 72
	file
}

```
至於非文字檔案只要印出相關資訊就非常簡單，使用msgbox解決。  



```
cdto(){
	cd $chfile2
	file
}
```
作業還有要求如果是目錄的話，就要進到那個目錄再顯示該目錄底下的檔案，誤打誤撞發現直接寫一個新函數包一個`cd`就解決了。  

[作業連結](https://github.com/star32134212/CSA_note/tree/master/HW2/HW2-2)
