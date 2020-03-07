+++
title = "STF之Provider系列五"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "STF"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/a358dbb7c05d)

## Mac环境下支持脚本自动部署 

需要Mac环境已经安装了XCode，因为环境编译时需要C++的编译器。

```
#!/bin/bash

# base on https://gist.github.com/aamnah/a62d30340de4f7ae98ea57b219d98d14
# NOTES
# `which` command will tell you a program is installed ONLY if it's in the $PATH
# so if your program is installed but the $PATH wasn't updated for whatever reason, it'll tell you program isn't installed
# `command -v foo` is a better alternative to `which foo`
# https://stackoverflow.com/a/46998376/890814
# Check if Homebrew is installed, install if we don't have it, update if we do
homebrew() {

	# $() is for command substitution, commands don't _return_ values, they _capture_ them
	# more here: https://stackoverflow.com/a/12137501/890814
	# [[ ]] is the newer _test command_ for evaluations, is more literal
	# more here: http://mywiki.wooledge.org/BashFAQ/031
	# the double [[ ]], and $() is important
	# `command -v brew` will output nothing if Homebrew is not installed
	if [[ $(command -v brew) == "" ]]; then
		echo "Installing Homebrew.. "
		ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
	else
		echo "Homebrew installed. "
	fi
}


# brew check 
brewcheck() {
  if brew list $1 >/dev/null ; then
    echo "$1 installed"
  else
    echo "$1 not installed "
	brew install $1
  fi
}


# check adb and link 
adbcheck() {
	adbversion=`adb version`
	if [[ $adbversion == *"version 1.0.40"* ]]; then
		echo $adbversion
	else
	    currentdir=`pwd`
		echo "use custom adb " $currentdir
		if [[ -f /usr/local/bin/adb ]]; then 
			rm /usr/local/bin/adb 
		fi 
		ln -s  $currentdir/tools/mac/adb /usr/local/bin/adb
	fi
}

# update environment file if exist
updateEnvFile() {
	if [[ -f $HOME/$1 ]]; then
		echo -e "export NVM_DIR=\"\$HOME/.nvm\"" >> $HOME/$1 
		# echo -e "[ -s \"\$NVM_DIR/nvm.sh\" ] && . \"\$NVM_DIR/nvm.sh\" " >> $HOME/$1 
		# echo -e "[ -s \"\$NVM_DIR/bash_completion\" ] && . \"\$NVM_DIR/bash_completion\" " >> $HOME/$1 
		source $HOME/$1 
	fi 
}

# node check
nodecheck() {
  nodeversion=`node -v`
  if [[ $nodeversion == "v8"* ]]; then
    echo $nodeversion
  else
    echo "install node carbon"
	if [[ $(command -v nvm) == "" ]]; then
		echo "Installing nvm first. "
		wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
		export NVM_DIR="$HOME/.nvm"
		[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
		[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
		updateEnvFile ".zshrc"
		updateEnvFile ".bashrc"
		updateEnvFile ".bash_profile"

	else
		echo "nvm already installed. "
	fi
    
    nvm install v8.16.0
    nvm use v8.16.0
  fi

}

# start here
if xcode-select -p > /dev/null ; then
   echo "XCode installed."
else
   echo "need xcode, please install XCode first. "
   exit 0
fi

# check brew
homebrew

declare -a array=(wget graphicsmagick zeromq protobuf yasm pkg-config)

# Install dependencies
for e in "${array[@]}"
do
   brewcheck $e
done

# check adb version and link adb 
adbcheck

# check node version 
nodecheck

echo "current node:" `node -v` " npm:" `npm -v`

nodeversion=`node -v`
if [[ $nodeversion == "v8"* ]]; then
	# start install 	
	echo "useRepository: " `npm config get registry`
	npm install 

	adbversion=`adb version`
	if [[ $adbversion == *"version 1.0.40"* ]]; then
		./provider.sh 
	else
		echo "adb version: " $adbversion
	fi
else
  echo "environment not fit. current node:" `node -v` " npm:" `npm -v`
fi 
```

[Linux下的正斜杠"/"和"\"的区别](https://blog.csdn.net/taiyang1987912/article/details/39551385)
>Windows 用反斜杠（“\”）的历史来自 DOS，而 DOS 的另一个传统是用斜杠（“/”）表示命令行参数，比如：
　　cd %SystemDrive%
　　dir /s /b shell32.dll
　　既然 DOS 这边斜杠被占用了，只好找一个最接近的。那就是它了。而在 UNIX 环境中，我们用减号（“-”）和双减号（“--”）表示命令行参数。
　　用斜杠表示命令行参数是兼容性原因。这个问题最初起源自 IBM。IBM 在最初加入 DOS 开发时贡献了大批工具，它们都是用斜杠处理命令行参数的。而这个传统源自于 **DEC/IBM**，比如当年的 VMS 就是用斜杠处理命令行参数，它的目录分隔符是美元符（“$”）。顺便说一句，这个传统也被部分地继承进了 DOS 和 Windows 体系，日文版的 Windows 就把反斜杠在屏幕上显示为“¥”，虽然实际上还是反斜杠。
　　如今的 Windows 内核在处理路径时确实可以同时支持斜杠和反斜杠。很多时候我们看到用斜杠时出错，是因为应用程序层面的原因。比如 cmd.exe 就不支持用斜杠表示路径，而PowerShell.exe 支持，也正因为这个原因，PowerShell 开始转而使用减号作为命令行参数的起始符。

[Why is the DOS path character ""?](https://blogs.msdn.microsoft.com/larryosterman/2005/06/24/why-is-the-dos-path-character/)

