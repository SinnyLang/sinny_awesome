# 安卓自动化脚本检测

24年冬接触到了一个刷广告赚钱APP（以下简称为APP1），只要看广告就能加金币，然后把几千金币兑换成几块钱人民币。套路和某果差不多，不同的是红果可以边看剧边刷，APP1就只是单纯刷广告。

广告刷完之后需要手动点击关闭，才能拿到金币。这就需要一直手动操作，但这会占用大部分个人时间，于是想使用AutoJSx写一个自动脚本刷广告，解放人力。但不曾想APP1的开发者已经料到了这一点，在APP1里内置了反自动化脚本检测程序，防止你自动刷广告。其实这个脚本不难绕过，先看看这个APP1是如何检测的自动化脚本。最后的最后给出我的绕过方法。

```alert type=note
提取这段程序的步骤使用安卓静态反编译得到smali，然后使用jadx转换成易读的java代码，过程不在此描述。
```

## 无障碍服务检测

```java
public final String isAccessibilityEnabled(Context context) throws RuntimeException {
	List listOf = CollectionsKt.listOf(new String[]{"com.zf.changdou", "com.kildare.autor", "com.kildare.nautor", "org.autojs.autojs", "com.qian.shuashua", "com.weiwei", "com.yuequ.auto", "com.kildare.rguard", "com.oasisfeng.greenify", "com.play4u.luabox", "com.play4u.luabox", "com.zdanjian.zdanjian", "com.granules", "com.zidongdianji", "com.zidongdianji", "com.yuedu.kgd", "com.yuecai.kgd", "org.autojs.autojspro"});
	Object systemService = context.getSystemService("accessibility");
	List<AccessibilityServiceInfo> enabledAccessibilityServiceList = ((AccessibilityManager) systemService).getEnabledAccessibilityServiceList(16);
	
    if (enabledAccessibilityServiceList.isEmpty()) {
		return "";
	}
	
    Iterator<T> it = enabledAccessibilityServiceList.iterator();
	
    while (it.hasNext()) {
		ServiceInfo serviceInfo = ((AccessibilityServiceInfo) it.next()).getResolveInfo().serviceInfo;
		String str = serviceInfo.name;
		
        if (str != null) {
			if (StringsKt.contains$default(str, "autojs", false, 2, (Object) null)) {
	    		return "无障碍功能" + serviceInfo.packageName + '[' + ((Object) serviceInfo.applicationInfo.nonLocalizedLabel) + ']';
			}
		}
		
        String str2 = serviceInfo.packageName;
		
        if (str2 != null && listOf.contains(str2)) {
			return "无障碍功能" + serviceInfo.packageName + '[' + ((Object) serviceInfo.applicationInfo.nonLocalizedLabel) + ']';
		}
	}
	return "";

}
```

这段代码检查了开启的无障碍应用，首先无障碍应用名称不应含有**autojs**字符，其次包名也不应含有下列中的任何一个。因为我使用的是autojsx，所以校验不通过。

```C
{
"com.zf.changdou", "com.kildare.autor", "com.kildare.nautor", 
"org.autojs.autojs", "com.qian.shuashua", "com.weiwei", 
"com.yuequ.auto", "com.kildare.rguard", "com.oasisfeng.greenify", 
"com.play4u.luabox", "com.play4u.luabox", "com.zdanjian.zdanjian", 
"com.granules", "com.zidongdianji", "com.zidongdianji", 
"com.yuedu.kgd", "com.yuecai.kgd", "org.autojs.autojspro"
}
```

继续，下面代码展示了程序捕捉了正在运行的屏幕阅读器，检查设备上是否有无障碍服务（如 TalkBack、Voice Assistant 等）处于活跃状态。屏幕阅读器在这里指的是注册了`android.accessibilityservice.AccessibilityService`和`android.accessibilityservice.category.FEEDBACK_SPOKEN`的服务。

```java
private final boolean isScreenReaderActive(Context context) {
	Intent intent = new Intent(SCREEN_READER_INTENT_ACTION);
	intent.addCategory(SCREEN_READER_INTENT_CATEGORY);
	boolean z = false;
	List<ResolveInfo> queryIntentServices = context.getPackageManager().queryIntentServices(intent, 0);

	if (queryIntentServices.size() <= 0) {return false;}

	if (Build.VERSION.SDK_INT >= 26) {
		for (ResolveInfo resolveInfo : queryIntentServices) {
			z |= isAccessibilitySettingsOn(context, resolveInfo.serviceInfo.packageName + '/' + resolveInfo.serviceInfo.name);
		}
	} else {
		ArrayList arrayList = new ArrayList();
		Object systemService = context.getSystemService("activity");
		Iterator<ActivityManager.RunningServiceInfo> it = ((ActivityManager) systemService).getRunningServices(Integer.MAX_VALUE).iterator();

		while (it.hasNext()) {
			String packageName = it.next().service.getPackageName();
			arrayList.add(packageName);
		}

		Iterator<ResolveInfo> it2 = queryIntentServices.iterator();

		while (it2.hasNext()) {
			if (arrayList.contains(it2.next().serviceInfo.packageName)) {
				z |= true;
			}
		}
	}
	return z;
}

public final boolean isAccessibilitySettingsOn(Context context, String service) {
	TextUtils.SimpleStringSplitter simpleStringSplitter = new TextUtils.SimpleStringSplitter(':');
	String string = Settings.Secure.getString(StubApp.getOrigApplicationContext(context.getApplicationContext()).getContentResolver(), "enabled_accessibility_services");

	if (string == null) {return false;}

	simpleStringSplitter.setString(string);
	while (simpleStringSplitter.hasNext()) {

		String next = simpleStringSplitter.next();
		if (StringsKt.equals(next, service, true)) {
			return true;
		}
	}
	return false;
}
```

## 处理Xposed框架

X通过实例化一个Xposed类来判断Xposed框架是否存在，这是APP1用来检测手机是否安装xposed的方法。如果 `ClassNotFoundException`（类未找到），返回 `false`（Xposed 不存在）。如果 `IllegalAccessException` 或 `InstantiationException`（访问/实例化异常），返回 `true`（可能是 Xposed 存在但被干扰）。APP1中欲实例化的两个类是：
- `de.robv.android.xposed.XposedHelpers`    
- `de.robv.android.xposed.XposedBridge`
```java
public final boolean isXposedExists() {
	try {
		ClassLoader.getSystemClassLoader().loadClass(XPOSED_HELPERS).newInstance();
		try {
			ClassLoader.getSystemClassLoader().loadClass(XPOSED_BRIDGE).newInstance();
			return true;
		} catch (ClassNotFoundException unused) {
			return false;
		} catch (IllegalAccessException | InstantiationException unused2) {
			return true;
		}
	} catch (ClassNotFoundException unused3) {
		return false;
	} catch (IllegalAccessException | InstantiationException unused4) {
		return true;
	}
}
```

检测Xposed的另一个方法是手动抛出异常，检测异常递归链，若是出现`de.robv.android.xposed.XposedBridge`则表示手机安装了Xposed。
```java
public final boolean isEposedExistByThrow() {
	try {
		throw new Exception("gg");
	} catch (Exception e) {
		StackTraceElement[] stackTrace = e.getStackTrace();
		for (StackTraceElement stackTraceElement : stackTrace) {
			String className = stackTraceElement.getClassName();
			if (StringsKt.contains$default(className, XPOSED_BRIDGE, false, 2, (Object) null)) {
				return true;
			}
		}
		return false;
	}
}
```

APP1还会通过反射关闭正在运行的Xposed。
```java
public final void antiXposedInject() {
	Field declaredField = ClassLoader.getSystemClassLoader().loadClass(XPOSED_BRIDGE).getDeclaredField("disableHooks");
	declaredField.setAccessible(true);
	declaredField.get(null);
	declaredField.set(null, Boolean.TRUE);
}
```

## 检测其它非法内容

- 无障碍检测列表之外的的自动化应用（如**自阅**）。通过方法名可以大致判断`e.h()`获取了已安装的应用程序列表。
```java
public final String isUserInsertIllegalApp() {
	List mutableListOf = CollectionsKt.mutableListOf(new String[]{"自阅"});
	List<e.a> h = e.h();
	String str = "";
	for (e.a aVar : h) {
		if (mutableListOf.contains(aVar.c())) {
			str = aVar.c();
		}
	}
	return str;
}
```

- 检测usb调试环境。
```java
public final boolean usbStatus(Context mContext) {
	return Settings.Secure.getInt(mContext.getContentResolver(), "adb_enabled", 0) > 0;
}
```

# 绕过

接下来是娱乐时间。

当绕过不成功时会出现图示警告。
![adb-warning.png](adb-warning.png width="150px")

我这里仅仅使用了AutoJsX，所以需要考虑绕过无障碍检测部分。既然是字符串检测，我就修改包名和应用名就好了。最终结果也证明了我的方案是正确的，但是就在我欣喜之际……

![id-error.png](id-error.png width="150px")

![silly](emoji-silly.png)
