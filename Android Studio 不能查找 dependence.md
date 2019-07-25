# Android Studio 不能查找 dependence

## build.gradle

	buildscript {
	    repositories {
			// 增加 mavenCentral
	        mavenCentral()
	        google()
	        jcenter()
	        
	    }
	    ……
	}
	
	allprojects {
	    repositories {
			// 增加 mavenCentral
	        mavenCentral()
	        google()
	        jcenter()
	    }
	}

## Open Module Settings

	Dependencies -> <All Modules> -> Add Library Dependency

就可以查找模块了