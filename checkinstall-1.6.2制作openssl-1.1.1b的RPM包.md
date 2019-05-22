# checkinstall-1.6.2 制作 openssl-1.1.1b 的 RPM 包

## 一、编译并安装 openssl-1.1.1b

	[root@localhost ~]# cd /usr/src
	[root@localhost src]# wget https://www.openssl.org/source/openssl-1.1.1b.tar.gz
	[root@localhost src]# tar zxf openssl-1.1.1b.tar.gz 
	[root@localhost src]# cd openssl-1.1.1b
	[root@localhost openssl-1.1.1b]# ./config --prefix=/usr
	Operating system: i686-whatever-linux2
	Configuring OpenSSL version 1.1.1b (0x1010102fL) for linux-x86
	Using os-specific seed configuration
	Creating configdata.pm
	Creating Makefile
	
	**********************************************************************
	***                                                                ***
	***   OpenSSL has been successfully configured                     ***
	***                                                                ***
	***   If you encounter a problem while building, please open an    ***
	***   issue on GitHub <https://github.com/openssl/openssl/issues>  ***
	***   and include the output from the following command:           ***
	***                                                                ***
	***       perl configdata.pm --dump                                ***
	***                                                                ***
	***   (If you are new to OpenSSL, you might want to consult the    ***
	***   'Troubleshooting' section in the INSTALL file first)         ***
	***                                                                ***
	**********************************************************************
	[root@localhost openssl-1.1.1b]# make && make install

## 二、yum install cvs gettext rpm-build

## 三、下载并安装 checkinstall-1.6.2.tar.gz

### 1、下载源码并解压
	[root@localhost ~]# cd /usr/src
	[root@localhost src]# wget http://asic-linux.com.mx/~izto/checkinstall/files/source/checkinstall-1.6.2.tar.gz
	[root@localhost src]# tar zxf checkinstall-1.6.2.tar.gz
	[root@localhost src]# cd checkinstall-1.6.2

### 2、修改 installwatch/installwatch.c
	at line 101, change: 
	static int (*true_scandir)( const char *,struct dirent ***,
    	int (*)(const struct dirent *),
    	int (*)(const void *,const void *)); 
	to: 
	static int (*true_scandir)( const char *,struct dirent ***,
    	int (*)(const struct dirent *),
    	int (*)(const struct dirent **,const struct dirent **));
 
 
	at line 121, change: 
	static int (*true_scandir64)( const char *,struct dirent64 ***,
    	int (*)(const struct dirent64 *),
    	int (*)(const void *,const void *));
	to: 
	static int (*true_scandir64)( const char *,struct dirent64 ***,
    	int (*)(const struct dirent64 *),
    	int (*)(const struct dirent64 **,const struct dirent64 **));

	at line 2941, change:
	#if (GLIBC_MINOR <= 4)
	to:
	#if (0) 
	
	at line 3080, change: 
	int scandir( const char *dir,struct dirent ***namelist,
		int (*select)(const struct dirent *),
		int (*compar)(const void *,const void *) ) {  
	to: 
	int scandir( const char *dir,struct dirent ***namelist,
		int (*select)(const struct dirent *),
		int (*compar)(const struct dirent **,const struct dirent **) ) {
  	
	at line 3692, change: 
	int scandir64( const char *dir,struct dirent64 ***namelist,
		int (*select)(const struct dirent64 *),
		int (*compar)(const void *,const void *) ) {
	to: 
	int scandir64( const char *dir,struct dirent64 ***namelist,
		int (*select)(const struct dirent64 *),
		int (*compar)(const struct dirent64 **,const struct dirent64 **) ) {

### 3、编译
	[root@localhost checkinstall-1.6.2]# make

### 4、修改 checkinstall

	at line 495, change: CHECKINSTALLRC=${CHECKINSTALLRC:-${INSTALLDIR}/checkinstallrc}
	to:
	CHECKINSTALLRC=${CHECKINSTALLRC:-${INSTALLDIR}/lib/checkinstall/checkinstallrc}
	 
	at line 2466, change: $RPMBUILD -bb ${RPM_TARGET_FLAG}${ARCHITECTURE} "$SPEC_PATH" &> ${TMP_DIR}/rpmbuild.log
	to:
	$RPMBUILD -bb ${RPM_TARGET_FLAG}${ARCHITECTURE} --buildroot $BROOTPATH "$SPEC_PATH" &> ${TMP_DIR}/rpmbuild.log 

### 5、安装
	[root@localhost checkinstall-1.6.2]# make install

### 6、建立目录
	[root@localhost checkinstall-1.6.2]# mkdir -p /root/rpmbuild/SOURCES

### 7、为打包做准备
	/bin/cp /usr/lib/libcrypto.so.1.1 /usr/lib/libcrypto.so.1.1.new
	/bin/cp /usr/lib/libssl.so.1.1 /usr/lib/libssl.so.1.1.new
	/bin/cp /usr/lib/libcrypto.a /usr/lib/libcrypto.a.new
	/bin/cp /usr/lib/libssl.a /usr/lib/libssl.a.new
	/bin/cp /usr/lib/libcrypto.so.1.1 /usr/lib/libcrypto.so.1.1.new
	/bin/cp /usr/lib/libssl.so.1.1 /usr/lib/libssl.so.1.1.new
	/bin/cp /usr/lib/engines-1.1/capi.so /usr/lib/engines-1.1/capi.so.new
	/bin/cp /usr/lib/engines-1.1/padlock.so /usr/lib/engines-1.1/padlock.so.new
	
	/bin/cp /usr/bin/openssl /usr/bin/openssl.new
	/bin/cp /usr/bin/c_rehash /usr/bin/c_rehash.new
	
	/bin/cp /usr/ssl/misc/CA.pl /usr/ssl/misc/CA.pl.new
	/bin/cp /usr/ssl/misc/tsget.pl /usr/ssl/misc/tsget.pl.new
	/bin/cp /usr/ssl/openssl.cnf.dist /usr/ssl/openssl.cnf.new
	/bin/cp /usr/ssl/ct_log_list.cnf.dist /usr/ssl/ct_log_list.cnf.new
	/bin/cp /usr/ssl/openssl.cnf /usr/ssl/openssl.cnf.new

### 8、制作RPM包
	[root@localhost checkinstall-1.6.2]# cd /usr/src/openssl-1.1.1b
	[root@localhost openssl-1.1.1b]# checkinstall 
	
	checkinstall 1.6.2, Copyright 2009 Felipe Eduardo Sanchez Diaz Duran
           	This software is released under the GNU GPL.
	
	Please choose the packaging method you want to use.
	Slackware [S], RPM [R] or Debian [D]? R

	**************************************
	**** RPM package creation selected ***
	**************************************
	
	This package will be built according to these values: 

	1 -  Summary: [ openssl-1.1.1b ]
	2 -  Name:    [ openssl ]
	3 -  Version: [ 1.1.1b ]
	4 -  Release: [ 1 ]
	5 -  License: [ GPL ]
	6 -  Group:   [ Applications/System ]
	7 -  Architecture: [ i386 ]
	8 -  Source location: [ openssl-1.1.1b ]
	9 -  Alternate source location: [  ]
	10 - Requires: [  ]
	11 - Provides: [ openssl ]

	Enter a number to change any of them or press ENTER to continue: 

回车后耐心等待，openssl包含文件非常多，需要很长时间才能打包成功。