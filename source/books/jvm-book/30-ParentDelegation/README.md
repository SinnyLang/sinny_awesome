# 类加载器的双亲委派

## 类加载器和父加载器

每个类加载器都有父加载器，类加载器创建时必须要指定一个父加载器，保存在`ClassLoader.parent`中。子加载器加载一个类时会先把这个类委托给父加载器加载，父加载器再继续委托到父加载器的父类加载器，直到父加载器为`null`，也就是到**BootStrap**加载器。只有当委托加载失败时，加载器才会调用自己的`findClass()`方法加载类。

```java
// 此loadClass()选自 - android.jar!/java.lang.ClassLoader#loadClass()
// name是要加载的类的名字，resolve表示是否连接该类
protected Class<?> loadClass(String name, boolean resolve)  
    throws ClassNotFoundException  
{  
	// First, check if the class has already been loaded
	// 如果 name 被加载过，则不再重新加载。findLoadClass()是native方法。
	Class<?> c = findLoadedClass(name);  
	
	if (c == null) {  
		// 如果没被加载过，则首先使用父加载器加载
		try {  
			if (parent != null) {  
				c = parent.loadClass(name, false);  
			} else {  
				c = findBootstrapClassOrNull(name);  
			}  
		} catch (ClassNotFoundException e) {  
			// ClassNotFoundException thrown if class not found  
			// from the non-null parent class loader
			// 父类加载失败会抛出ClassNotFound异常，
			// 为了正常执行加载流程需要捕捉异常            
		}  

		if (c == null) {  
			// If still not found, then invoke findClass in order  
			// to find the class.     
			// 如果父类加载器没找到，则使用自己的findClass()方法加载类。           
			c = findClass(name);  
			// Android-removed: Android has no jvmstat.  
			// this is the defining class loader; record the stats                
			// PerfCounter.getParentDelegationTime().addTime(t1 - t0);                
			// PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);                
			// PerfCounter.getFindClasses().increment();            
		}  
	}  
	return c;  
}
```

有委托机制的存在，使得Java无法加载自定义核心类。例如`java.lang.String`，由于委托加载机制存在，该类只能被BootStrap加载器加载。

## 自定义类加载器

当要实现一个自定义类加载器时，必须继承ClassLoader类，并实现`ClassLoader.findClass()`方法。而`findClass()`定义了如何加载类。

示例代码：
```java
package dalvik.system;
// 选自 android-35\dalvik\system\BaseDexClassLoader.java # findClass()
// 该类自定义了一个类加载器，用于加载安卓程序的类。
// PS: 安卓程序的类都被压缩到Dex文件中，加载某个类时，需要从Dex文件中加载二进制文件
public class BaseDexClassLoader extends ClassLoader {
	@Override  
	protected Class<?> findClass(String name) throws ClassNotFoundException {  
	    // First, check whether the class is present in our shared libraries.  
	    if (sharedLibraryLoaders != null) {  
	        for (ClassLoader loader : sharedLibraryLoaders) {  
	            try {  
	                return loader.loadClass(name);  
	            } catch (ClassNotFoundException ignored) {  
	            }  
	        }  
	    }  
	    // Check whether the class in question is present in the dexPath that  
	    // this classloader operates on. 
	    // pathList保存了Dex文件的路径，从pathList中寻找名为name的类
	    // 追根到底，会发现最终使用了native方法寻找类 - dalvik.system.DexFile#defineClassNative()
	    // 由于嵌套层数多且对象结构复杂，这里不多展示。
	    List<Throwable> suppressedExceptions = new ArrayList<Throwable>();  
	    Class c = pathList.findClass(name, suppressedExceptions);  
	    if (c != null) {  
	        return c;  
	    }  
	    // Now, check whether the class is present in the "after" shared libraries.  
	    if (sharedLibraryLoadersAfter != null) {  
	        for (ClassLoader loader : sharedLibraryLoadersAfter) {  
	            try {  
	                return loader.loadClass(name);  
	            } catch (ClassNotFoundException ignored) {  
	            }  
	        }  
	    }  
	    // 如果都找不到名为name的类，则抛出异常
	    if (c == null) {  
	        ClassNotFoundException cnfe = new ClassNotFoundException(  
	                "Didn't find class \"" + name + "\" on path: " + pathList);  
	        for (Throwable t : suppressedExceptions) {  
	            cnfe.addSuppressed(t);  
	        }  
	        throw cnfe;  
	    }  
	    return c;  
	}
}
```

## 打破双亲委派机制

当自定义类加载器需要打破双亲委派机制时，需要方法重写`loadClass()`，`loadClass()`定义了类加载的顺序。

以下是在一个开源音乐软件（MusicFree安卓端）中编写的一个实例。**BootClassLoader**是安卓中加载基础类的加载器，类似JAVA中的BootStartup加载器。同样**PathClassLoader**类似JAVA中的ApplicationClassLoader，负责加载安卓程序MusicFree相关的类。正常的委托关系是**BootClassLoader <- PathClassLoader**，但是由于需要拦截目标类，现将委托关系修改如下：**BootClassLoader <- this Loader <- PathClassLoader <- baseAudioPlayerDexClassLoader**。

那么现在正常加载一个类的流程变成了：
1. 程序运行，发现缺少name类，使用`PathClassLoader`加载name。
2. `PathClassLoader`委托给`MediaSessionHookClassLoader`。
3. `MediaSessionHookClassLoader`发现`name.startsWith("com.doublesymmetry.kotlinaudio.R")`，返回`null`，`PathClassLoader`发现父类加载不了，则自己加载。如果`MediaSessionHookClassLoader`发现`name.startsWith("com.doublesymmetry.kotlinaudio")`，则使用`baseAudioPlayerDexClassLoader.loadClass(name)`加载name。
4. `baseAudioPlayerDexClassLoader`是一个`DexClassLoader`，它能从after-MusicFree.app.dex中加载目标类。但是仅加载**com.doublesymmetry.kotlinaudio.\*** 的类。
5. `baseAudioPlayerDexClassLoader`遇到不属于自己加载范围的类，交给`PathClassLoader`加载。

> 第5步存在是有意义的。考虑如下情况，`com.doublesymmetry.kotlinaudio.A`中使用`String`类，但是在`com.doublesymmetry.kotlinaudio.A`加载之前，`String`并未被加载到内存，这个时候需要加载`String`，而此时加载`String`的加载器是`baseAudioPlayerDexClassLoader`，无法正常被加载，需要指定使用`PathClassLoader`加载。

```alert type=warning
截止2025-07月，该部分并未commit，所以仓库中找不到。代码并未粘贴全部。
```

```java
package fun.upup.musicfree.equalizerUtil.hook;
// 委托关系 BootClassLoader <- this Loader <- PathClassLoader <- baseAudioPlayerDexClassLoader
// this Loader 拦截目标类； baseAudioPlayerDexClassLoader 加载目标类  
// this Loader 直接继承 DexClassLoader 可省略 baseAudioPlayerDexClassLoader
public class MediaSessionHookClassLoader extends ClassLoader{
	@Override  
	protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {  	    
	    if (name.startsWith("com.doublesymmetry.kotlinaudio.R")){  
	        // 拒绝加载R类的委托  
	        // 由于dex分包机制，com.doublesymmetry.kotlinaudio.R和com.doublesymmetry.kotlinaudio.*  
	        // 不在同一dex文件，导致baseAudioPlayerDexClassLoader找不到R类，最终导致循环加载R类  
	        // 具体表现为  
	        // baseAudioPlayerDexClassLoader找不到R类，调用pathLoader.loadClass()加载R类，  
	        // pathLoader.loadClass()调用MediaSessionHookClassLoader.loadClass()加载R类，  
	        // MediaSessionHookClassLoader.loadClass()使用baseAudioPlayerDexClassLoader加载R类  
	        // ...  
	        // MediaSessionHookClassLoader.loadClass() 返回null可以直在第二步终止循环  
	        return null;  
	    }  
	    
	    if (name.startsWith("com.doublesymmetry.kotlinaudio")) {  
	        try {  
	            String outputDexPath = context.getFilesDir().getAbsolutePath() + File.separator;  
	            String outputDexPathname = outputDexPath + "after-MusicFree.app.dex"; // Path to the modified dex file  
	            File modifiedDexFile = new File(outputDexPathname);  
	  
	            // 如果修改后的dex文件已经存在则不再重新修改dex  
	            if (!modifiedDexFile.exists()) {  
	                // 修改dex文件并保存到 outputDexPathname 
	                copyDex(outputDexPath);  
	                // 使用dexlib修改BaseAudioPlayer类  
	                modifyDex(outputDexPathname);  
	            }  
	            if (baseAudioPlayerDexClassLoader == null) {  
	                // 使用 DexClassLoader加载新的Dex  
	                baseAudioPlayerDexClassLoader = new DexClassLoader(  
	                        outputDexPath + "after-MusicFree.app.dex",  
	                        null,  
	                        null,  
	                        this.getClass().getClassLoader() // 加载this类的loader是PathClassLoader  
	                ) {  
	                    // 重写loadClass 破坏委托加载机制，  
	                    @Override  
	                    public Class<?> loadClass(String name) throws ClassNotFoundException {  
	                        Class<?> c = findLoadedClass(name);  
	                        if (c == null) {  
	                            try {  
	                                // 仅加载 com.doublesymmetry.kotlinaudio.*  
	                                if (name.startsWith("com.doublesymmetry.kotlinaudio"))  
	                                    c = findClass(name);  
	                            } catch (ClassNotFoundException e) {  
	                                // 为了保证加载流程正常执行，需要捕捉异常  
	                            }  
	                        }  
	                        if (c == null) {  
	                            ClassLoader loader = getParent();  
	                            Log.i("FindClass","自定义 load class: 在修改的dex中没找到: " + name);  
	                            Log.i("FindClass","自定义 load class: 尝试通过父加载器加载: " + loader);  
	                            c = loader.loadClass(name);  
	                        } else {  
	                            Log.i("FindClass","自定义 load class: 在修改的dex中找到" + name + "，结果: " + c);  
	                        }  
	  
	                        if (c == null) {  
	                            throw new ClassNotFoundException("Didn't find class \"" + name);  
	                        }  
	                        return c;  
	                    }  
	                };  
	            }  
	  
	            Log.i("FindClass", "自定义 load class: " + name);  
	            Class<?> targetClass = baseAudioPlayerDexClassLoader.loadClass(name);  
	  
	            if(targetClass == null){  
	                Log.e("FindClass", "自定义 load class error: " + name);  
	            } else {  
	                return targetClass;  
	            }  
	        } catch (Exception e) {  
	            e.printStackTrace();  
	        }  
	    }  
	    
	    // DexClassLoader 加载类失败  
	    return super.loadClass(name, resolve);  
	}
}
```

## 打破双亲委派机制--线程上下文类加载器与SPI机制

**服务发现机制（SPI）** 也打破了双亲委派机制。SPI常被用于驱动加载等。





