+++
title = "STF之Provider一键部署一"
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



[link on JianShu](https://www.jianshu.com/p/86e493ccf13a)

[STF之Provider系列五](https://www.jianshu.com/p/a358dbb7c05d)里的前提条件是已经获取到里源码。在源码目录执行上面到脚本。

问题： 
- 需要有获取到源码到权限
- 公司网络使用brew安装依赖时过慢，可能导致安装失败
- npm install时bower获取github上到资源时大概率会遇到失败情况，需要多次执行

[STF之Provider系列六：brew](https://www.jianshu.com/p/1a52b09203b9)里提到brew可以支持本地安装，以及如何获取brew应用到安装包、依赖包到方法。

方案：
- 提供依赖包的合集压缩包`stftools.tar.gz`
- 编译后的STF项目压缩包`remoteDevice.tar.gz `
- 使用线上脚本方式 `oneshell.sh`

这种条件下，Mac环境下只需要执行`curl  -o- http://inner.domain.com/path/oneshell.sh | bash` 就可以完成Provider的一键部署工作。 

依赖包大概57MB + 通过 `npm install`直接安装的话包大概600MB
依赖包大概57MB + 编译好的STF项目压缩后大概220MB 
这样整体下载的内容更小，并且只依赖本地网络。

```
#!/bin/bash

# download dependencies
brewCache() {
	if [ -f stftools.tar.gz ]
	then
		echo "stftools.tar.gz exist"
		rm stftools.tar.gz
	else
		echo "no stftools.tar.gz, download first."
	fi

	curl http://inner.domain.com/path/stftools.tar.gz -o stftools.tar.gz

	if [ -d stftools ]
	then
		echo "stftools dir exist"
		rm -rf stftools
	else
		echo "no stftools dir"
	fi

	tar -zxvf stftools.tar.gz

	for f in `ls stftools`
	do
		dorf=`pwd`/stftools/$f
		if [  -f $dorf ]
		then
			echo "file: " $f
			mv $dorf $(brew --cache)
		else
			echo "ignore dir: " $f
		fi
	done
        rm -rf stftools
	rm stftools.tar.gz
}

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

	for i in 1 2 3 4 5
	do
		echo "Looping ... number $i"
		if brew list $1 >/dev/null ; then
			echo "$1 installed"
			break
		else
			echo "$1 not installed "
			brew install $1
		fi
	done

	if brew list $1 >/dev/null ; then
		echo "final check: $1 installed. good \n"
	else
		echo "\n\n $1 cannot be installed. make sure your network condition is good enough and try it later... \n\n"
		exit 0
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
			curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
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

# download compiled source code
getSourceCode () {

	if [ -f remoteDevice.tar.gz ]
	then
		echo "remoteDevice.tar.gz file exist"
		rm remoteDevice.tar.gz
	fi

	curl http://inner.domain.com/path/remoteDevice.tar.gz -o remoteDevice.tar.gz

	if [ -d RemoteDevice ]
	then
		echo "RemoteDevice dir exist"
		rm -rf RemoteDevice
	fi

	tar -zxvf remoteDevice.tar.gz
	rm remoteDevice.tar.gz
}

# check brew
brewCheckAgain() {

	if [[ $(command -v brew) == "" ]]; then
		echo "\n\n brew cannot be installed. make sure your network condition is good enough and try it later... \n\n"
		exit 0
	else
		echo "Homebrew already installed. "
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
brewCheckAgain
brewCache

declare -a array=(graphicsmagick zeromq protobuf yasm pkg-config)

# Install dependencies
for e in "${array[@]}"
do
	brewcheck $e
done

getSourceCode


if [ -d RemoteDevice ]
then
	echo "go to RemoteDevice dir "
	cd  RemoteDevice

        # check adb version and link adb 
        adbcheck
	# check node version
	nodecheck

	echo "current node:" `node -v` " npm:" `npm -v`

	nodeversion=`node -v`
	if [[ $nodeversion == "v8"* ]]; then

		adbversion=`adb version`
		if [[ $adbversion == *"version 1.0.40"* ]]; then
			./provider.sh
		else
			echo "adb version: " $adbversion
		fi
	else
		echo "environment not fit. current node:" `node -v` " npm:" `npm -v`
		cd ..

	fi
else 
       echo "RemoteDevice dir not found."
fi
```

