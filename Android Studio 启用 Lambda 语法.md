# Android Studio 启用 Lambda 语法

## 安装JDK 8

## Open Module Settings

	SDK Location -> JDK location -> 选择JDK 8路径

	Modules -> app -> Source Compatibility -> 1.8

	Modules -> app -> Targer Compatibility -> 1.8

## build.grandle(Project)

	buildscript {
	    repositories {
	        mavenCentral()
	        google()
	        jcenter()
	        
	    }
	    dependencies {
	        classpath 'com.android.tools.build:gradle:3.4.2'
			// 增加 gradle-retrolambda
	        classpath 'me.tatarka:gradle-retrolambda:3.7.1'
	        // NOTE: Do not place your application dependencies here; they belong
	        // in the individual module build.gradle files
	    }
	}
	
	allprojects {
	    repositories {
	        google()
	        jcenter()
	        mavenCentral()
	    }
	}
	
	task clean(type: Delete) {
	    delete rootProject.buildDir
	}


## build.grandle(Module)

	apply plugin: 'com.android.application'
	// 增加 retrolambda
	apply plugin: 'me.tatarka.retrolambda'
	
	android {
	    compileSdkVersion 28
	    defaultConfig {
	        applicationId "cn.suzhou.xj107359.geoquiz"
	        minSdkVersion 23
	        targetSdkVersion 28
	        versionCode 1
	        versionName "1.0"
	        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
	    }
	    buildTypes {
	        release {
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
	        }
	    }
	    compileOptions {
	        targetCompatibility = '1.8'
	        sourceCompatibility = '1.8'
	    }
	}
	dependencies {
	    implementation fileTree(dir: 'libs', include: ['*.jar'])
	    implementation 'com.android.support:appcompat-v7:28.0.0'
	    implementation 'com.android.support.constraint:constraint-layout:2.0.0-beta2'
	    testImplementation 'junit:junit:4.12'
	    androidTestImplementation 'com.android.support.test:runner:1.0.2'
	    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
	}
	
## code

	mTrueButton = findViewById(R.id.true_button);
    mTrueButton.setOnClickListener((v) -> {
        Toast toast = Toast.makeText(QuizActivity.this,
                R.string.correct_toast,
                Toast.LENGTH_SHORT);
        toast.setGravity(Gravity.TOP, 0, 0);
        toast.show();
    });
