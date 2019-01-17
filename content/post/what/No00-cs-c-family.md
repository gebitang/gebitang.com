+++
title = "CS course"
description = "make up lessons for C Family"
tags = [
    "cs",
    "c"
]
date = "2018-05-09"
topics = [ 
    "cs",
    "c"
]
toc = true
+++


### 双感叹号

两个!是为了把非0值转换成1,而0值还是0。
在C语言中，所以非0值都表示真。所以!非0值 = 0，而!0 = 1。
所以!!非0值 = 1，而!!0 = 0


```
http://journals.ecs.soton.ac.uk/java/tutorial/native1.1/stepbystep/index.html
http://www.stat.ucla.edu/~dinov/courses_students.dir/04/Winter/STAT130D.dir/STAT130D_notes.dir/JNI_C_example.html

data
	四角码、偏旁部首组合 解析规则
	sqlilte 文件？
	
openCV 版本？ 
	编译方法：
	

tessdata 3.05.2版本 ——4.0
	公共的文件 
	
https://github.com/tesseract-ocr/tessdata


debug、release如何区别？各自的作用？: 生成不同的文件夹下的内容 debug下有pdb符号表
目前使用的为release版本 -G "Visual Studio 15 2017 Win64" 参数指定，不使用，则默认是32位的

G:/work/release/32/OCRUtils/cmakeStaticRelease
G:/work/release/64/OCRUtils/cmakeStaticRelease
32、64的区别由 “”

编译过程中输出的检查项作用？not found 的结果是否有影响？

demo工程编译：
debug方式调试——


windows: cmake执行后需要理由vs进行 “生成解决方案”
mac\linux: cmake执行完成后，直接执行 make即可

不指定64位"Visual Studio 15 2017 Win64"，会编译为默认的32位和MD（动态连接的格式？）
使用的是64位 & MT（静态编译的格式）
通用的应该为：
cmake -G "Visual Studio 15 2017 Win64" -DZ_PREFIX=1 -DCMAKE_USER_MAKE_RULES_OVERRIDE="D:/workplaces/cplusplus/OCRUtils/cmake/compiler_flags_overrides.cmake" ../


https://blog.csdn.net/qq_20408397/article/details/77803215
链接提示无法解析的外部符号 __imp__fopen
#pragma comment(lib,"msvcrt.lib")

https://docs.microsoft.com/en-us/cpp/c-runtime-library/crt-library-features?view=vs-2017
https://docs.microsoft.com/zh-cn/cpp/build/reference/md-mt-ld-use-run-time-library?view=vs-2017
md mt ld

tiff： 只要生成了 tiff.lib和tiffxx.lib即可。 其他的失败可以忽略

默认使用的构建目录定位为buildR
构建顺序：
1. zlib-1.2.11
cmake -G "Visual Studio 15 2017 Win64" -DZ_PREFIX=1 -DCMAKE_USER_MAKE_RULES_OVERRIDE="D:/workplaces/cplusplus/OCRUtils/cmake/compiler_flags_overrides.cmake" ../


2. jpeg-9c 3. giflib-5.1.4
cmake -G "Visual Studio 15 2017 Win64" -DSTATIC=TRUE ../

4. libpng-1.6.34  5. tiff-4.0.9
cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="D:/workplaces/cplusplus/OCRUtils/cmakeStaticRelease" -DCMAKE_USER_MAKE_RULES_OVERRIDE="D:/workplaces/cplusplus/OCRUtils/cmake/compiler_flags_overrides.cmake" ../

6. leptonica
cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="D:/workplaces/cplusplus/OCRUtils/cmakeStaticRelease" -DSTATIC=TRUE -DCMAKE_USER_MAKE_RULES_OVERRIDE="D:/workplaces/cplusplus/OCRUtils/cmake/compiler_flags_overrides.cmake" ../

7. tesseract
cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="D:/workplaces/cplusplus/OCRUtils/cmakeStaticRelease" -DSTATIC=TRUE -DLeptonica_DIR="D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/" -DSTATIC=TRUE -DCMAKE_USER_MAKE_RULES_OVERRIDE="D:/workplaces/cplusplus/OCRUtils/cmake/compiler_flags_overrides.cmake" ../

8. ocr （在主工程目录的build文件夹下进行cmake）
cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="D:/workplaces/cplusplus/OCRUtils/cmakeStaticRelease" -DSTATIC=TRUE -DCMAKE_USER_MAKE_RULES_OVERRIDE="D:/workplaces/cplusplus/OCRUtils/cmake/compiler_flags_overrides.cmake" ../

自动构建
Building for: Visual Studio 15 2017


cmake -DZ_PREFIX=1 -DCMAKE_USER_MAKE_RULES_OVERRIDE="D:/workplaces/cplusplus/OCRUtils/cmake/compiler_flags_overrides.cmake" ../


CMAKE_MODULE_PATH=cmakeStaticRelease文件夹路径
CMAKE_USER_MAKE_RULES_OVERRIDE="D:/workplaces/cplusplus/OCRUtils/cmake/compiler_flags_overrides.cmake"



-------------------------------------------------------------------
git clone git@10.10.1.18:Automation/OCRUtils.git
________________________________________
linux
1.zlib-1.2.11
cmake -DZ_PREFIX=1 ../

2.jpeg-9c  3.gif
cmake -DSTATIC=TRUE ../

4.png 5.tiff
cmake -DCMAKE_MODULE_PATH="/home/testin/cxj/release/OCRUtils/cmakeStaticRelease" ../

6.leptonica   8.ocr
cmake -DCMAKE_MODULE_PATH="/home/testin/cxj/release/OCRUtils/cmakeStaticRelease" -DSTATIC=TRUE ../

7.tesseract
cmake -DCMAKE_MODULE_PATH="/home/testin/cxj/release/OCRUtils/cmakeStaticRelease" -DSTATIC=TRUE -DLeptonica_DIR="/home/testin/cxj/release/OCRUtils/thirdLibs/leptonica-1.76.0" ../

________________________________________
Mac
zlib
cmake -DZ_PREFIX=1 -DCMAKE_MACOSX_RPATH=1 ../

jpeg  gif
cmake -DCMAKE_MACOSX_RPATH=1 -DSTATIC=TRUE ../

png tiff
cmake -DCMAKE_MODULE_PATH="/Users/testin/cxj/ocr/OCRUtils/cmakeStaticRelease" -DCMAKE_MACOSX_RPATH=1 ../

leptonica   ocr
cmake -DCMAKE_MODULE_PATH="/Users/testin/cxj/ocr/OCRUtils/cmakeStaticRelease" -DCMAKE_MACOSX_RPATH=1 -DSTATIC=TRUE ../

tesseract
cmake -DCMAKE_MODULE_PATH="/Users/testin/cxj/ocr/OCRUtils/cmakeStaticRelease" -DCMAKE_MACOSX_RPATH=1 -DSTATIC=TRUE -DLeptonica_DIR="/Users/testin/cxj/ocr/OCRUtils/thirdLibs/leptonica-1.76.0" ../

________________________________________
Windows
zlib
cmake -DZ_PREFIX=1 -DCMAKE_USER_MAKE_RULES_OVERRIDE=$overrides_cmake_path ../
cmake -DZ_PREFIX=1 ../
cmake -G "Visual Studio 15 2017 Win64" -DZ_PREFIX=1 ../


jpeg gif
cmake -DSTATIC=TRUE ../
cmake -G "Visual Studio 15 2017 Win64" -DSTATIC=TRUE ../

png tiff
cmake -DCMAKE_MODULE_PATH="G:/work/release/32/OCRUtils/cmakeStaticRelease" -DCMAKE_USER_MAKE_RULES_OVERRIDE=$overrides_cmake_path ../
cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="G:/work/release/64/OCRUtils/cmakeStaticRelease" -DCMAKE_USER_MAKE_RULES_OVERRIDE=$overrides_cmake_path ../

cmake -DCMAKE_MODULE_PATH="G:/work/release/32d/OCRUtils/cmakeStaticRelease" -DCMAKE_USER_MAKE_RULES_OVERRIDE=$overrides_cmake_path -DDEBUG_ON=1 ../

leptonica   ocr
cmake -DCMAKE_MODULE_PATH="G:/work/release/32/OCRUtils/cmakeStaticRelease" -DSTATIC=TRUE -DCMAKE_USER_MAKE_RULES_OVERRIDE=$overrides_cmake_path ../
cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="G:/work/release/64/OCRUtils/cmakeStaticRelease" -DSTATIC=TRUE -DCMAKE_USER_MAKE_RULES_OVERRIDE=$overrides_cmake_path ../

tesseract
cmake  -DCMAKE_MODULE_PATH="G:/work/release/32/OCRUtils/cmakeStaticRelease" -DLeptonica_DIR="G:/work/release/32/OCRUtils/thirdLibs/leptonica-1.76.0/" -DSTATIC=TRUE -DCMAKE_USER_MAKE_RULES_OVERRIDE=$overrides_cmake_path ../
cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="G:/work/release/64/OCRUtils/cmakeStaticRelease" -DLeptonica_DIR="G:/work/release/64/OCRUtils/thirdLibs/leptonica-1.76.0/" -DSTATIC=TRUE -DCMAKE_USER_MAKE_RULES_OVERRIDE=$overrides_cmake_path ../

============================================

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils (master)
$ cd thirdLibs/

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs (master)
$ ls
giflib-5.1.4/  leptonica-1.76.0/  tesseract-3.05.02/  zlib-1.2.11/
jpeg-9c/       libpng-1.6.34/     tiff-4.0.9/

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs (master)
$ cd zlib-1.2.11/

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11                                                 (master)
$ cd buildR/

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/                                                buildR (master)
$ rm -rf *

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/                                                buildR (master)
$ ll
total 0

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/                                                buildR (master)
$ cmake -G "Visual Studio 15 2017 Win64" -DZ_PREFIX=1 -DCMAKE_USER_MAKE_RULES_OV                                                ERRIDE="D:/workplaces/cplusplus/OCRUtils/cmake/compiler_flags_overrides.cmake" .                                                ./
-- The C compiler identification is MSVC 19.15.26729.0
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_C_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_C_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_C_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_C_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Looking for sys/types.h
-- Looking for sys/types.h - found
-- Looking for stdint.h
-- Looking for stdint.h - found
-- Looking for stddef.h
-- Looking for stddef.h - found
-- Check size of off64_t
-- Check size of off64_t - failed
-- Looking for fseeko
-- Looking for fseeko - not found
-- Looking for unistd.h
-- Looking for unistd.h - not found
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR (master)
$ cd ../../jpeg-9c/buildR/

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR (master)
$ rm -rf *

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR (master)
$ cmake -G "Visual Studio 15 2017 Win64" -DSTATIC=TRUE ../
-- The C compiler identification is MSVC 19.15.26729.0
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Looking for unistd.h
-- Looking for unistd.h - not found
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR (master)
$ cd ../../giflib-5.1.4/bu
BUGS       build.asc  buildR/

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR (master)
$ cd ../../giflib-5.1.4/buildR/

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR (master)
$ rm -rf *

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR (master)
$ cmake -G "Visual Studio 15 2017 Win64" -DSTATIC=TRUE ../
-- The C compiler identification is MSVC 19.15.26729.0
-- The CXX compiler identification is MSVC 19.15.26729.0
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/H
-- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/H
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR (master)
$ cd ../../libpng-1.6.34/buildR/

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR (master)
$ rm -rf *

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR (master)
$ cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="D:/workplaces/cplusplus/OCRUtils/cmakeStaticRelease" -DCMAKE_USER_ags_overrides.cmake" ../
-- The C compiler identification is MSVC 19.15.26729.0
-- The ASM compiler identification is MSVC
-- Found assembler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hostx86/x64/cl.e
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_C_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_C_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_C_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_C_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- Find Zlib at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11 ------------------.
-- FOUND zconf.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR
-- FOUND zlib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11
-- FOUND libz.a;zlibstatic.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib at D:/w/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release
-- Found libz.a;zlibstatic.lib : D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib
-- Performing Test HAVE_LD_VERSION_SCRIPT
-- Performing Test HAVE_LD_VERSION_SCRIPT - Failed
-- Performing Test HAVE_SOLARIS_LD_VERSION_SCRIPT
-- Performing Test HAVE_SOLARIS_LD_VERSION_SCRIPT - Failed
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR (master)
$ cd ../../tiff-4.0.9/buildR/

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR (master)
$ cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="D:/workplaces/cplusplus/OCRUtils/cmakeStaticRelease" -DCMAKE_USER_ags_overrides.cmake" ../
-- Building tiff version 4.0.9
-- libtiff library version 5.3.0
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_C_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_C_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_C_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_C_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- CMAKE_HOST_SYSTEM_PROCESSOR set to AMD64
-- HOST_FILLORDER set to FILLORDER_MSB2LSB
-- HOST_BIG_ENDIAN set to 0
-- HAVE_IEEEFP set to 1
-- Find Zlib at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11 ------------------.
-- FOUND zconf.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR
-- FOUND zlib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11
-- FOUND libz.a;zlibstatic.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib at D:/w/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release
-- Found libz.a;zlibstatic.lib : D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib
-- Find jpeg at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c ------------------.
-- FOUND jpeglib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c
-- FOUND libjpeg.a;jpeg.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Found libjpeg.a;jpeg.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_CXX_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_CXX_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_CXX_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_CXX_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Could NOT find GLUT (missing: GLUT_glut_LIBRARY GLUT_INCLUDE_DIR)
--
-- Libtiff is now configured for
--
--   Installation directory:             C:/Program Files/tiff
--   Documentation directory:            C:/Program Files/tiff/share/doc/tiff
--   C compiler:                         C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726
--   C++ compiler:                       C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726
--   Build shared libraries:             OFF
--   Enable linker symbol versioning:    FALSE
--   Support Microsoft Document Imaging: ON
--   Use win32 IO:                       TRUE
--
--  Support for internal codecs:
--   CCITT Group 3 & 4 algorithms:       ON
--   Macintosh PackBits algorithm:       ON
--   LZW algorithm:                      ON
--   ThunderScan 4-bit RLE algorithm:    ON
--   NeXT 2-bit RLE algorithm:           ON
--   LogLuv high dynamic range encoding: ON
--
--  Support for external codecs:
--   ZLIB support:                       ON (requested) TRUE (availability)
--   Pixar log-format algorithm:         ON (requested) TRUE (availability)
--   JPEG support:                       ON (requested) TRUE (availability)
--   Old JPEG support:                   ON (requested) TRUE (availability)
--   JPEG 8/12 bit dual mode:            ON (requested) FALSE (availability)
--   ISO JBIG support:                   OFF (requested) FALSE (availability)
--   LZMA2 support:                      OFF (requested)  (availability)
--
--   C++ support:                        ON (requested) TRUE (availability)
--
--   OpenGL support:                     FALSE
--
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR (master)
$ ll
total 251
-rw-r--r-- 1 joechin 197121 33140 10月 19 15:29 ALL_BUILD.vcxproj
-rw-r--r-- 1 joechin 197121   308 10月 19 14:40 ALL_BUILD.vcxproj.filters
-rw-r--r-- 1 joechin 197121   165 10月 19 14:41 ALL_BUILD.vcxproj.user
drwxr-xr-x 1 joechin 197121     0 10月 19 15:29 build/
-rw-r--r-- 1 joechin 197121  3095 10月 19 14:40 cmake_install.cmake
-rw-r--r-- 1 joechin 197121 25731 10月 19 14:40 CMakeCache.txt
drwxr-xr-x 1 joechin 197121     0 10月 19 15:29 CMakeFiles/
drwxr-xr-x 1 joechin 197121     0 10月 19 15:29 contrib/
-rw-r--r-- 1 joechin 197121   475 10月 19 14:40 CTestTestfile.cmake
drwxr-xr-x 1 joechin 197121     0 10月 19 15:29 html/
-rw-r--r-- 1 joechin 197121 11519 10月 19 14:40 INSTALL.vcxproj
-rw-r--r-- 1 joechin 197121   551 10月 19 14:40 INSTALL.vcxproj.filters
drwxr-xr-x 1 joechin 197121     0 10月 19 15:29 libtiff/
-rw-r--r-- 1 joechin 197121   326 10月 19 14:40 libtiff-4.pc
drwxr-xr-x 1 joechin 197121     0 10月 19 15:29 man/
drwxr-xr-x 1 joechin 197121     0 10月 19 15:29 port/
-rw-r--r-- 1 joechin 197121 11278 10月 19 14:40 RUN_TESTS.vcxproj
-rw-r--r-- 1 joechin 197121   553 10月 19 14:40 RUN_TESTS.vcxproj.filters
drwxr-xr-x 1 joechin 197121     0 10月 19 15:29 test/
-rw-r--r-- 1 joechin 197121 47041 10月 19 14:40 tiff.sln
drwxr-xr-x 1 joechin 197121     0 10月 19 15:29 tools/
drwxr-xr-x 1 joechin 197121     0 10月 19 14:41 x64/
-rw-r--r-- 1 joechin 197121 39030 10月 19 15:29 ZERO_CHECK.vcxproj
-rw-r--r-- 1 joechin 197121   552 10月 19 14:40 ZERO_CHECK.vcxproj.filters

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR (master)
$ cd libtiff

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff (master)
$ ll
total 127
-rw-r--r-- 1 joechin 197121  8763 10月 19 14:40 cmake_install.cmake
drwxr-xr-x 1 joechin 197121     0 10月 19 14:41 CMakeFiles/
-rw-r--r-- 1 joechin 197121   348 10月 19 14:40 CTestTestfile.cmake
-rw-r--r-- 1 joechin 197121 12747 10月 19 14:40 INSTALL.vcxproj
-rw-r--r-- 1 joechin 197121   551 10月 19 14:40 INSTALL.vcxproj.filters
drwxr-xr-x 1 joechin 197121     0 10月 19 14:41 Release/
-rw-r--r-- 1 joechin 197121 12506 10月 19 14:40 RUN_TESTS.vcxproj
-rw-r--r-- 1 joechin 197121   553 10月 19 14:40 RUN_TESTS.vcxproj.filters
-rw-r--r-- 1 joechin 197121  7600 10月 19 14:40 tif_config.h
drwxr-xr-x 1 joechin 197121     0 10月 19 14:41 tiff.dir/
-rw-r--r-- 1 joechin 197121 27057 10月 19 14:40 tiff.vcxproj
-rw-r--r-- 1 joechin 197121  8151 10月 19 14:40 tiff.vcxproj.filters
-rw-r--r-- 1 joechin 197121  3553 10月 19 14:40 tiffconf.h
drwxr-xr-x 1 joechin 197121     0 10月 19 14:41 tiffxx.dir/
-rw-r--r-- 1 joechin 197121 22761 10月 19 14:40 tiffxx.vcxproj
-rw-r--r-- 1 joechin 197121   962 10月 19 14:40 tiffxx.vcxproj.filters

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff (master)
$ ls R
Release/                   RUN_TESTS.vcxproj          RUN_TESTS.vcxproj.filters

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff (master)
$ ls R
Release/                   RUN_TESTS.vcxproj          RUN_TESTS.vcxproj.filters

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff (master)
$ ls Release/tiff
ls: cannot access 'Release/tiff': No such file or directory

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff (master)
$ ls Release/tiff*
Release/tiff.lib  Release/tiffxx.lib

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff (master)
$ cd ../../../leptonica-1.76.0/buildR/

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR (master)
$ rm -rf *

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR (master)
$ cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="D:/workplaces/cplusplus/OCRUtils/cmakeStaticRelease" -DSTATIC=TRUEke/compiler_flags_overrides.cmake" ../
-- The C compiler identification is MSVC 19.15.26729.0
-- The CXX compiler identification is MSVC 19.15.26729.0
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_C_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_C_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_C_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_C_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_CXX_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_CXX_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_CXX_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_CXX_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/H
-- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/H
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Find Gif at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4 ------------------.
-- FOUND gif_lib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/lib
-- FOUND libgif.a;gif.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Found libgif.a;gif.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Find jpeg at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c ------------------.
-- FOUND jpeglib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c
-- FOUND libjpeg.a;jpeg.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Found libjpeg.a;jpeg.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Find Png at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34 ------------------.
-- FOUND pnglibconf.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR
-- FOUND png.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34
-- FOUND libpng16.a;libpng16_static.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_stildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release
-- Found libpng16.a;libpng16_static.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_static
-- Find Tiff at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9 ------------------.
-- FOUND tiff.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/libtiff
-- FOUND tif_config.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff
-- FOUND libtiff.a;tiff.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
-- Found libtiff.a;tiff.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
-- Find Zlib at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11 ------------------.
-- FOUND zconf.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR
-- FOUND zlib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11
-- FOUND libz.a;zlibstatic.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib at D:/w/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release
-- Found libz.a;zlibstatic.lib : D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib
-- Could NOT find PkgConfig (missing: PKG_CONFIG_EXECUTABLE)
-- Looking for include file dlfcn.h
-- Looking for include file dlfcn.h - not found
-- Looking for include file inttypes.h
-- Looking for include file inttypes.h - found
-- Looking for include file memory.h
-- Looking for include file memory.h - found
-- Looking for include file stdint.h
-- Looking for include file stdint.h - found
-- Looking for include file stdlib.h
-- Looking for include file stdlib.h - found
-- Looking for include file strings.h
-- Looking for include file strings.h - not found
-- Looking for include file string.h
-- Looking for include file string.h - found
-- Looking for include file sys/stat.h
-- Looking for include file sys/stat.h - found
-- Looking for include file sys/types.h
-- Looking for include file sys/types.h - found
-- Looking for include file unistd.h
-- Looking for include file unistd.h - not found
-- Looking for include file openjpeg-2.0/openjpeg.h
-- Looking for include file openjpeg-2.0/openjpeg.h - not found
-- Looking for include file openjpeg-2.1/openjpeg.h
-- Looking for include file openjpeg-2.1/openjpeg.h - not found
-- Looking for include file openjpeg-2.2/openjpeg.h
-- Looking for include file openjpeg-2.2/openjpeg.h - not found
-- Looking for include file openjpeg-2.3/openjpeg.h
-- Looking for include file openjpeg-2.3/openjpeg.h - not found
-- Looking for fmemopen
-- Looking for fmemopen - not found
-- Looking for fstatat
-- Looking for fstatat - not found
-- Check if the system is big endian
-- Searching 16 bit integer
-- Looking for stddef.h
-- Looking for stddef.h - found
-- Check size of unsigned short
-- Check size of unsigned short - done
-- Using unsigned short
-- Check if the system is big endian - little endian
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR (master)
$ cd ../../tesseract-3.05.02/buildR/

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR (master)
$ rm -rf *

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR (master)
$ cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="D:/workplaces/cplusplus/OCRUtils/cmakeStaticRelease" -DSTATIC=TRUE1.76.0/" -DSTATIC=TRUE -DCMAKE_USER_MAKE_RULES_OVERRIDE="D:/workplaces/cplusplus/OCRUtils/cmake/compiler_flags_overrides.cmake"
-- The C compiler identification is MSVC 19.15.26729.0
-- The CXX compiler identification is MSVC 19.15.26729.0
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_C_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_C_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_C_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_C_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_CXX_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_CXX_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_CXX_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_CXX_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/H
-- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/H
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Find Leptonica at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0 ------------------.
-- FOUND allheaders.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/src
-- FOUND endianness.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src
-- FOUND libleptonica.a;leptonica-1.76.0.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Releasetonica-1.76.0/buildR/src;D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release
-- Found libleptonica.a;leptonica-1.76.0.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release/lep
-- Looking for include file dlfcn.h
-- Looking for include file dlfcn.h - not found
-- Looking for include file inttypes.h
-- Looking for include file inttypes.h - found
-- Looking for include file limits.h
-- Looking for include file limits.h - found
-- Looking for include file malloc.h
-- Looking for include file malloc.h - found
-- Looking for include file memory.h
-- Looking for include file memory.h - found
-- Looking for include file stdbool.h
-- Looking for include file stdbool.h - found
-- Looking for include file stdint.h
-- Looking for include file stdint.h - found
-- Looking for include file stdlib.h
-- Looking for include file stdlib.h - found
-- Looking for include file strings.h
-- Looking for include file strings.h - not found
-- Looking for include file string.h
-- Looking for include file string.h - found
-- Looking for include file sys/ipc.h
-- Looking for include file sys/ipc.h - not found
-- Looking for include file sys/shm.h
-- Looking for include file sys/shm.h - not found
-- Looking for include file sys/stat.h
-- Looking for include file sys/stat.h - found
-- Looking for include file sys/types.h
-- Looking for include file sys/types.h - found
-- Looking for include file sys/wait.h
-- Looking for include file sys/wait.h - not found
-- Looking for include file unistd.h
-- Looking for include file unistd.h - not found
-- Looking for include file cairo/cairo-version.h
-- Looking for include file cairo/cairo-version.h - not found
-- Looking for include file CL/cl.h
-- Looking for include file CL/cl.h - not found
-- Looking for include file OpenCL/cl.h
-- Looking for include file OpenCL/cl.h - not found
-- Looking for include file pango-1.0/pango/pango-features.h
-- Looking for include file pango-1.0/pango/pango-features.h - not found
-- Looking for include file tiffio.h
-- Looking for include file tiffio.h - not found
-- Looking for include file unicode/uchar.h
-- Looking for include file unicode/uchar.h - not found
-- Looking for getline
-- Looking for getline - not found
-- Looking for snprintf
-- Looking for snprintf - not found
-- Looking for stddef.h
-- Looking for stddef.h - found
-- Check size of long long int
-- Check size of long long int - done
-- Check size of off_t
-- Check size of off_t - done
-- Check size of mbstate_t
-- Check size of mbstate_t - done
-- Check size of wchar_t
-- Check size of wchar_t - done
-- Check size of _Bool
-- Check size of _Bool - done
-- Check if the system is big endian
-- Searching 16 bit integer
-- Check size of unsigned short
-- Check size of unsigned short - done
-- Using unsigned short
-- Check if the system is big endian - little endian
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR (master)
$ cd ../../../build/

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/build (master)
$ rm -rf *

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/build (master)
$ cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="D:/workplaces/cplusplus/OCRUtils/cmakeStaticRelease" -DSTATIC=TRUEke/compiler_flags_overrides.cmake" ../
-- The C compiler identification is MSVC 19.15.26729.0
-- The CXX compiler identification is MSVC 19.15.26729.0
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_C_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_C_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_C_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_C_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_CXX_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_CXX_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_CXX_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_CXX_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/H
-- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/H
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Find Leptonica at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0 ------------------.
-- FOUND allheaders.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/src
-- FOUND endianness.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src
-- FOUND libleptonica.a;leptonica-1.76.0.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Releasetonica-1.76.0/buildR/src;D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release
-- Found libleptonica.a;leptonica-1.76.0.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release/lep
-- Find Tesseract at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02 ------------------.
-- FOUND opencl_device_selection.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/opencl
-- FOUND thresholder.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccmain
-- FOUND ccutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccutil
-- FOUND ccstruct.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccstruct
-- FOUND baseapi.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/api
-- FOUND svutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/viewer
-- FOUND gettimeofday.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/vs2010/port
-- FOUND liblibtesseract.a;tesseract305.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tes.05.02/buildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release
-- Found liblibtesseract.a;tesseract305.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tessera
-- Find Zlib at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11 ------------------.
-- FOUND zconf.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR
-- FOUND zlib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11
-- FOUND libz.a;zlibstatic.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib at D:/w/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release
-- Found libz.a;zlibstatic.lib : D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib
-- Find Gif at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4 ------------------.
-- FOUND gif_lib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/lib
-- FOUND libgif.a;gif.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Found libgif.a;gif.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Find jpeg at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c ------------------.
-- FOUND jpeglib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c
-- FOUND libjpeg.a;jpeg.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Found libjpeg.a;jpeg.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Find Png at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34 ------------------.
-- FOUND pnglibconf.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR
-- FOUND png.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34
-- FOUND libpng16.a;libpng16_static.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_stildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release
-- Found libpng16.a;libpng16_static.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_static
-- Find Tiff at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9 ------------------.
-- FOUND tiff.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/libtiff
-- FOUND tif_config.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff
-- FOUND libtiff.a;tiff.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
-- Found libtiff.a;tiff.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
-- Found JNI: C:/Program Files/Java/jdk1.8.0_181/lib/jawt.lib
------------------------C:/Program Files/Java/jdk1.8.0_181/include--------------------------
------------------------C:/Program Files/Java/jdk1.8.0_181/include/win32--------------------------
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workplaces/cplusplus/OCRUtils/build

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/build (master)
$

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/build (master)
$ rm -rf *

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/build (master)
$ cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="D:/workplaces/cplusplus/OCRUtils/cmakeStaticRelease" -DSTATIC=TRUEke/compiler_flags_overrides.cmake" ../
-- The C compiler identification is MSVC 19.15.26729.0
-- The CXX compiler identification is MSVC 19.15.26729.0
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_C_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_C_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_C_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_C_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hos
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_CXX_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_CXX_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_CXX_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_CXX_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/H
-- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/H
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Find Leptonica at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0 ------------------.
-- FOUND allheaders.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/src
-- FOUND endianness.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src
-- FOUND libleptonica.a;leptonica-1.76.0.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Releasetonica-1.76.0/buildR/src;D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release
-- Found libleptonica.a;leptonica-1.76.0.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release/lep
-- Find Tesseract at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02 ------------------.
-- FOUND opencl_device_selection.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/opencl
-- FOUND thresholder.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccmain
-- FOUND ccutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccutil
-- FOUND ccstruct.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccstruct
-- FOUND baseapi.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/api
-- FOUND svutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/viewer
-- FOUND gettimeofday.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/vs2010/port
-- FOUND liblibtesseract.a;tesseract305.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tes.05.02/buildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release
-- Found liblibtesseract.a;tesseract305.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tessera
-- Find Zlib at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11 ------------------.
-- FOUND zconf.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR
-- FOUND zlib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11
-- FOUND libz.a;zlibstatic.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib at D:/w/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release
-- Found libz.a;zlibstatic.lib : D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib
-- Find Gif at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4 ------------------.
-- FOUND gif_lib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/lib
-- FOUND libgif.a;gif.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Found libgif.a;gif.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Find jpeg at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c ------------------.
-- FOUND jpeglib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c
-- FOUND libjpeg.a;jpeg.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Found libjpeg.a;jpeg.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Find Png at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34 ------------------.
-- FOUND pnglibconf.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR
-- FOUND png.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34
-- FOUND libpng16.a;libpng16_static.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_stildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release
-- Found libpng16.a;libpng16_static.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_static
-- Find Tiff at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9 ------------------.
-- FOUND tiff.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/libtiff
-- FOUND tif_config.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff
-- FOUND libtiff.a;tiff.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
-- Found libtiff.a;tiff.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
CMake Error at src/testInOcr/CMakeLists.txt:243 (add_dependencies):
  add_dependencies called with incorrect number of arguments


-- Configuring incomplete, errors occurred!
See also "D:/workplaces/cplusplus/OCRUtils/build/CMakeFiles/CMakeOutput.log".

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/build (master)
$ cmake -G "Visual Studio 15 2017 Win64" -DCMAKE_MODULE_PATH="D:/workplaces/cplusplus/OCRUtils/cmakeStaticRelease" -DSTATIC=TRUEke/compiler_flags_overrides.cmake" ../
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_C_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_C_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_C_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_C_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_CXX_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_CXX_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_CXX_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_CXX_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Find Leptonica at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0 ------------------.
-- FOUND allheaders.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/src
-- FOUND endianness.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src
-- FOUND libleptonica.a;leptonica-1.76.0.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Releasetonica-1.76.0/buildR/src;D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release
-- Found libleptonica.a;leptonica-1.76.0.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release/lep
-- Find Tesseract at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02 ------------------.
-- FOUND opencl_device_selection.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/opencl
-- FOUND thresholder.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccmain
-- FOUND ccutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccutil
-- FOUND ccstruct.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccstruct
-- FOUND baseapi.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/api
-- FOUND svutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/viewer
-- FOUND gettimeofday.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/vs2010/port
-- FOUND liblibtesseract.a;tesseract305.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tes.05.02/buildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release
-- Found liblibtesseract.a;tesseract305.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tessera
-- Find Zlib at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11 ------------------.
-- FOUND zconf.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR
-- FOUND zlib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11
-- FOUND libz.a;zlibstatic.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib at D:/w/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release
-- Found libz.a;zlibstatic.lib : D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib
-- Find Gif at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4 ------------------.
-- FOUND gif_lib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/lib
-- FOUND libgif.a;gif.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Found libgif.a;gif.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Find jpeg at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c ------------------.
-- FOUND jpeglib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c
-- FOUND libjpeg.a;jpeg.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Found libjpeg.a;jpeg.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Find Png at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34 ------------------.
-- FOUND pnglibconf.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR
-- FOUND png.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34
-- FOUND libpng16.a;libpng16_static.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_stildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release
-- Found libpng16.a;libpng16_static.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_static
-- Find Tiff at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9 ------------------.
-- FOUND tiff.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/libtiff
-- FOUND tif_config.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff
-- FOUND libtiff.a;tiff.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
-- Found libtiff.a;tiff.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workplaces/cplusplus/OCRUtils/build

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/build (master)
$ cmake ../
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_C_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_C_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_C_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_C_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_CXX_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_CXX_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_CXX_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_CXX_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Find Leptonica at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0 ------------------.
-- FOUND allheaders.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/src
-- FOUND endianness.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src
-- FOUND libleptonica.a;leptonica-1.76.0.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Releasetonica-1.76.0/buildR/src;D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release
-- Found libleptonica.a;leptonica-1.76.0.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release/lep
-- Find Tesseract at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02 ------------------.
-- FOUND opencl_device_selection.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/opencl
-- FOUND thresholder.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccmain
-- FOUND ccutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccutil
-- FOUND ccstruct.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccstruct
-- FOUND baseapi.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/api
-- FOUND svutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/viewer
-- FOUND gettimeofday.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/vs2010/port
-- FOUND liblibtesseract.a;tesseract305.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tes.05.02/buildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release
-- Found liblibtesseract.a;tesseract305.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tessera
-- Find Zlib at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11 ------------------.
-- FOUND zconf.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR
-- FOUND zlib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11
-- FOUND libz.a;zlibstatic.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib at D:/w/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release
-- Found libz.a;zlibstatic.lib : D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib
-- Find Gif at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4 ------------------.
-- FOUND gif_lib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/lib
-- FOUND libgif.a;gif.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Found libgif.a;gif.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Find jpeg at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c ------------------.
-- FOUND jpeglib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c
-- FOUND libjpeg.a;jpeg.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Found libjpeg.a;jpeg.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Find Png at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34 ------------------.
-- FOUND pnglibconf.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR
-- FOUND png.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34
-- FOUND libpng16.a;libpng16_static.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_stildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release
-- Found libpng16.a;libpng16_static.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_static
-- Find Tiff at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9 ------------------.
-- FOUND tiff.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/libtiff
-- FOUND tif_config.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff
-- FOUND libtiff.a;tiff.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
-- Found libtiff.a;tiff.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workplaces/cplusplus/OCRUtils/build

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/build (master)
$ cmake ../
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_C_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_C_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_C_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_C_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_CXX_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_CXX_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_CXX_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_CXX_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Find Leptonica at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0 ------------------.
-- FOUND allheaders.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/src
-- FOUND endianness.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src
-- FOUND libleptonica.a;leptonica-1.76.0.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Releasetonica-1.76.0/buildR/src;D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release
-- Found libleptonica.a;leptonica-1.76.0.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release/lep
-- Find Tesseract at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02 ------------------.
-- FOUND opencl_device_selection.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/opencl
-- FOUND thresholder.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccmain
-- FOUND ccutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccutil
-- FOUND ccstruct.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccstruct
-- FOUND baseapi.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/api
-- FOUND svutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/viewer
-- FOUND gettimeofday.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/vs2010/port
-- FOUND liblibtesseract.a;tesseract305.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tes.05.02/buildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release
-- Found liblibtesseract.a;tesseract305.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tessera
-- Find Zlib at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11 ------------------.
-- FOUND zconf.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR
-- FOUND zlib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11
-- FOUND libz.a;zlibstatic.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib at D:/w/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release
-- Found libz.a;zlibstatic.lib : D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib
-- Find Gif at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4 ------------------.
-- FOUND gif_lib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/lib
-- FOUND libgif.a;gif.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Found libgif.a;gif.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Find jpeg at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c ------------------.
-- FOUND jpeglib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c
-- FOUND libjpeg.a;jpeg.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Found libjpeg.a;jpeg.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Find Png at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34 ------------------.
-- FOUND pnglibconf.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR
-- FOUND png.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34
-- FOUND libpng16.a;libpng16_static.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_stildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release
-- Found libpng16.a;libpng16_static.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_static
-- Find Tiff at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9 ------------------.
-- FOUND tiff.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/libtiff
-- FOUND tif_config.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff
-- FOUND libtiff.a;tiff.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
-- Found libtiff.a;tiff.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
-- Found JNI: C:/Program Files/Java/jdk1.8.0_181/lib/jawt.lib
------------------------C:/Program Files/Java/jdk1.8.0_181/include--------------------------
------------------------C:/Program Files/Java/jdk1.8.0_181/include/win32--------------------------
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workplaces/cplusplus/OCRUtils/build

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/build (master)
$ cmake ../
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_C_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_C_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_C_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_C_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_CXX_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_CXX_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_CXX_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_CXX_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Find Leptonica at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0 ------------------.
-- FOUND allheaders.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/src
-- FOUND endianness.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src
-- FOUND libleptonica.a;leptonica-1.76.0.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Releasetonica-1.76.0/buildR/src;D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release
-- Found libleptonica.a;leptonica-1.76.0.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release/lep
-- Find Tesseract at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02 ------------------.
-- FOUND opencl_device_selection.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/opencl
-- FOUND thresholder.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccmain
-- FOUND ccutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccutil
-- FOUND ccstruct.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccstruct
-- FOUND baseapi.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/api
-- FOUND svutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/viewer
-- FOUND gettimeofday.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/vs2010/port
-- FOUND liblibtesseract.a;tesseract305.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tes.05.02/buildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release
-- Found liblibtesseract.a;tesseract305.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tessera
-- Find Zlib at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11 ------------------.
-- FOUND zconf.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR
-- FOUND zlib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11
-- FOUND libz.a;zlibstatic.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib at D:/w/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release
-- Found libz.a;zlibstatic.lib : D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib
-- Find Gif at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4 ------------------.
-- FOUND gif_lib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/lib
-- FOUND libgif.a;gif.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Found libgif.a;gif.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Find jpeg at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c ------------------.
-- FOUND jpeglib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c
-- FOUND libjpeg.a;jpeg.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Found libjpeg.a;jpeg.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Find Png at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34 ------------------.
-- FOUND pnglibconf.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR
-- FOUND png.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34
-- FOUND libpng16.a;libpng16_static.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_stildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release
-- Found libpng16.a;libpng16_static.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_static
-- Find Tiff at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9 ------------------.
-- FOUND tiff.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/libtiff
-- FOUND tif_config.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff
-- FOUND libtiff.a;tiff.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
-- Found libtiff.a;tiff.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
------------------------C:/Program Files/Java/jdk1.8.0_181/include--------------------------
------------------------C:/Program Files/Java/jdk1.8.0_181/include/win32--------------------------
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workplaces/cplusplus/OCRUtils/build

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/build (master)
$ cmake ../
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_C_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_C_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_C_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_C_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- link to static  C and C++ runtime lirbary(/MT /MTd)
-- CMAKE_CXX_FLAGS_DEBUG_INIT: /MTd /Zi /Ob0 /Od /RTC1
-- CMAKE_CXX_FLAGS_RELEASE_INIT: /MT /O2 /Ob2 /DNDEBUG
-- CMAKE_CXX_FLAGS_MINSIZEREL_INIT: /MT /O1 /Ob1 /DNDEBUG
-- CMAKE_CXX_FLAGS_RELWITHDEBINFO_INIT: /MT /Zi /O2 /Ob1 /DNDEBUG
-- Find Leptonica at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0 ------------------.
-- FOUND allheaders.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/src
-- FOUND endianness.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src
-- FOUND libleptonica.a;leptonica-1.76.0.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Releasetonica-1.76.0/buildR/src;D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release
-- Found libleptonica.a;leptonica-1.76.0.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/leptonica-1.76.0/buildR/src/Release/lep
-- Find Tesseract at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02 ------------------.
-- FOUND opencl_device_selection.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/opencl
-- FOUND thresholder.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccmain
-- FOUND ccutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccutil
-- FOUND ccstruct.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/ccstruct
-- FOUND baseapi.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/api
-- FOUND svutil.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/viewer
-- FOUND gettimeofday.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/vs2010/port
-- FOUND liblibtesseract.a;tesseract305.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tes.05.02/buildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release
-- Found liblibtesseract.a;tesseract305.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tesseract-3.05.02/buildR/Release/tessera
-- Find Zlib at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11 ------------------.
-- FOUND zconf.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR
-- FOUND zlib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11
-- FOUND libz.a;zlibstatic.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib at D:/w/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release
-- Found libz.a;zlibstatic.lib : D:/workplaces/cplusplus/OCRUtils/thirdLibs/zlib-1.2.11/buildR/Release/zlibstatic.lib
-- Find Gif at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4 ------------------.
-- FOUND gif_lib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/lib
-- FOUND libgif.a;gif.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Found libgif.a;gif.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/giflib-5.1.4/buildR/lib/Release/gif.lib
-- Find jpeg at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c ------------------.
-- FOUND jpeglib.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c
-- FOUND libjpeg.a;jpeg.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Found libjpeg.a;jpeg.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/jpeg-9c/buildR/Release/jpeg.lib
-- Find Png at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34 ------------------.
-- FOUND pnglibconf.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR
-- FOUND png.H  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34
-- FOUND libpng16.a;libpng16_static.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_stildR;D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release
-- Found libpng16.a;libpng16_static.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/libpng-1.6.34/buildR/Release/libpng16_static
-- Find Tiff at dir:D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9 ------------------.
-- FOUND tiff.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/libtiff
-- FOUND tif_config.h  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff
-- FOUND libtiff.a;tiff.lib LIB  D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
-- Found libtiff.a;tiff.lib: D:/workplaces/cplusplus/OCRUtils/thirdLibs/tiff-4.0.9/buildR/libtiff/Release/tiff.lib
------------------------C:/Program Files/Java/jdk1.8.0_181/include--------------------------
------------------------C:/Program Files/Java/jdk1.8.0_181/include/win32--------------------------
-- Configuring done
-- Generating done
-- Build files have been written to: D:/workplaces/cplusplus/OCRUtils/build

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/build (master)
$ git staus
git: 'staus' is not a git command. See 'git --help'.

The most similar command is
        status

joechin@Gebitang MINGW64 /d/workplaces/cplusplus/OCRUtils/build (master)
$ 
```

整数类型是最基础的数据类型，从一方面看，它表达的是不带小数部分的数，是自然数；但是，从另一方面看，它是C语言用来表达计算机内部存储内容的一种方式。整数有char、short、int、long long等多种不同的类型，每一种有不同的范围，表达内存中不同大小的存储空间。为了能准确表达计算机，C语言的整数类型还可以加上unsigned修饰符，来保证纯二进制运算。能否正确和深入的理解整数类型，看似简单初步，却能反映你是否真正理解计算机本身。



浮点类型就是生活中习以为常的带小数点的数，但是在计算机内部，浮点数的处理是很特别的。整数是以二进制的形式存在于计算机内部的，而浮点数不是。浮点数是编码的数字，因此两个浮点数不能在计算机内部直接以二进制的方式进行计算，通常需要专门的硬件支持。

浮点数的计算机内部表达形式造成了浮点数的很多有意思的现象。

C语言本没有逻辑类型，在内部计算中使用整数表达关系运算和逻辑运算的结果，0表示false，而非0的值表示true。

在C99中，也没有固有的逻辑类型，但是通过一个头文件定义了可以直接使用的true和false这两个值，以及bool这个类型。

逻辑运算则是C语言固有的成分。
