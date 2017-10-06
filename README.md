# Clang Cross Compile For RasberryPi on Redhat 7

## 1. References 
https://llvm.org/docs/HowToCrossCompileLLVM.html 

https://medium.com/@zw3rk/making-a-raspbian-cross-compilation-sdk-830fe56d75ba

## 2. Steps for setup 
### 2.1. Select where to install clang cross compile SDK 
Example:

  `export BASE=/opt`  
  `mkdir -p $BASE/raspbian-sdk/{prebuilt,sysroot,tmp}`
  
### 2.2 Download pre-build clang+llvm
From http://releases.llvm.org/download.html, dowload clang+llvm-3.4.2-x86_64-fedora20.xz
	
  `cd $BASE/raspbian-sdk/prebuilt`
  
  `wget http://releases.llvm.org/3.4.2/clang+llvm-3.4.2-x86_64-fedora20.xz -O $BASE/raspbian-sdk/prebuilt/clang+llvm-3.4.2-x86_64-fedora20.xz`
	
Then extract this to `$BASE/raspbian-sdk/prebuilt`

	cd $BASE/raspbian-sdk/prebuilt
	tar -xvf clang+llvm-3.4.2-x86_64-fedora20.xz
	rm -f clang+llvm-3.4.2-x86_64-fedora20.xz
  
### 2.3 Setup Linker and Other Utilities
  `cd $BASE/raspbian-sdk/tmp 
  
  wget https://ftp.gnu.org/gnu/binutils/binutils-2.28.tar.bz2 
  
	tar -xvf binutils-2.28.tar.bz2 
  
	cd binutils-2.28 
  
	./configure --prefix="${BASE}/raspbian-sdk/prebuilt" \  
            --target=arm-linux-gnueabihf \            
            --enable-gold=yes \            
            --enable-ld=yes \            
            --enable-targets=arm-linux-gnueabihf \            
            --enable-multilib \            
            --enable-interwork \            
            --disable-werror \            
            --quiet
            
  make && make install`

### 2.4 Copy Headers and Libraries from RasberryPi to cross-compile machine (Redhat 7) 
  `cd $BASE/raspbian-sdk
  
  rsync -rzLR --safe-links \
      pi@raspberrypi_address:/usr/lib/arm-linux-gnueabihf \
      pi@raspberrypi_address:/usr/lib/gcc/arm-linux-gnueabihf \
      pi@raspberrypi_address:/usr/include \
      pi@raspberrypi_address:/lib/arm-linux-gnueabihf \
      ./sysroot/`

Notes: On RasberryPi we also need to install 'rsync' before running above command. 

  `sudo apt-get install rsync`
  
### 2.5 Remove temp folder 
  'cd $BASE/raspbian-sdk
  
  rm -rf tmp`

After doing all above steps, now the sdk folder layout is like below. 

    ├── prebuilt
	|   |── clang+llvm-3.4.2-x86_64-fedora20
	│   ├── arm-linux-gnueabihf
	│   ├── bin
	│   ├── include
	│   ├── lib
	│   ├── libexec
	│   └── share
	└── sysroot
		├── lib
		└── usr
        
### 2.6. Create shell wrappers
Do below commands before create shell wrappers 

	ln -s $BASE/raspbian-sdk/prebuilt/clang+llvm-3.4.2-x86_64-fedora20/bin/clang /usr/bin/clang
 	
	ln -s /usr/bin/clang $BASE/raspbian-sdk/prebuilt/bin/clang

#### 2.6.1 Create file CC script "arm-linux-gnueabihf-clang"
	cd $BASE
	vi arm-linux-gnueabihf-clang
	
	#------------------------------------------------------------------------
	#!/bin/bash
	BASE="/opt/raspbian-sdk"
	SYSROOT="${BASE}/sysroot"
	TARGET=arm-linux-gnueabihf
	COMPILER_PATH="${SYSROOT}/usr/lib/gcc/${TARGET}/4.9"
	exec env COMPILER_PATH="${COMPILER_PATH}" \
		 "${BASE}/prebuilt/bin/clang" --target=${TARGET} \
		 --sysroot="${SYSROOT}" \
		 -isysroot "${SYSROOT}" \
		 --gcc-toolchain="${BASE}/prebuilt/bin" \
		 "$@"
	#------------------------------------------------------------------------`
	
	
#### 2.6.2 Create file LD script "arm-linux-gnueabihf-clang-ld"
	cd $BASE
	vi arm-linux-gnueabihf-clang-ld
	
	#------------------------------------------------------------------------
	#!/bin/bash
	BASE="/opt/raspbian-sdk"
	SYSROOT="${BASE}/sysroot"
	TARGET=arm-linux-gnueabihf
	COMPILER_PATH="${SYSROOT}/usr/lib/gcc/${TARGET}/4.9"
	exec env COMPILER_PATH="${COMPILER_PATH}" \
		 "${BASE}/prebuilt/bin/clang" --target=${TARGET} \
		 --sysroot="${SYSROOT}" \
		 -isysroot "${SYSROOT}" \
		 -L"${COMPILER_PATH}" \
		 --gcc-toolchain="${BASE}/prebuilt/bin" \
		 "$@"	
	#------------------------------------------------------------------------
	
#### 2.6.3. Create soft link
	cd $BASE
	chmod +x arm-linux-gnueabihf-clang
	chmod +x arm-linux-gnueabihf-clang-ld 
	ln -s $BASE/arm-linux-gnueabihf-clang /usr/bin/arm-linux-gnueabihf-clang
	ln -s $BASE/arm-linux-gnueabihf-clang-ld /usr/bin/arm-linux-gnueabihf-clang-ld

### 2.7 Test
Create file hello.c 

	#include <stdio.h>
	int
	main(int argc, char ** argv) {
	  printf("Hello World!\n");
	  return 0;
	}

Compile 'hello.c': 

	arm-linux-gnueabihf-clang-ld -o hello hello.c
	
Copy execurable file "hello" to Raspberry machine for execution. 
