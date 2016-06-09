## Reverse me (250)
### Tools
- jadx-gui
- Genymotion
- Xposed Framework

Looking at the `MainActivity.java` with `jadx-gui`, we can find a call to the class `o.vaehekua`.

```java
public void Clicked(View view) {
  try {
    Class.forName("o.vaehekua").getMethod("weicighi", new Class[]{Activity.class}).invoke(null, new Object[]{this});
  } catch (Throwable th) {
    Throwable th2 = th2.getCause();
  }
}
```

A lot of code could not be recreated by jadx-gui, so we had to deal with some instructions. But we can see, that the method `weicighi(int, int, int)` is called multiple times.

```
r2 = weicighi(r2, r2, r3);	 Catch:{ all -> 0x000f }
```

Looking at the contract, we see that there is some string output.
```
private static java.lang.String weicighi(int r6, int r7, int r8)
```

Instead of patching the apk, we decided to use Xposed to inject some code, which generates us some logs with the return value of our function.

```java
if(lpparam.packageName.equals("de.ecspride.reverseme")) {
  findAndHookMethod("o.vaehekua", lpparam.classLoader, "weicighi", int.class, int.class, int.class, new XC_MethodHook() {
    @Override
    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
      Log.d("foo", (String) param.getResult());
    }
  });
}
```

We can see, that there are some class- and method names. Because there had been some obfuscation techniques in the qualifying round, it was pretty clear that these are used in here too. Also we can see, that a file is going to be created and deleted.
```
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: prefix
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: .apk
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: java.io.File
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: createTempFile
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: java.io.File
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: prefix
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: extension
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: java.io.File
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: createTempFile
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: java.io.File
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: java.io.File
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: delete
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: java.io.File
06-01 14:29:41.942 1249-1249/de.ecspride.reverseme D/foo: mkdir
06-01 14:29:41.998 1249-1249/de.ecspride.reverseme D/foo: java.io.File
06-01 14:29:41.998 1249-1249/de.ecspride.reverseme D/foo: getAbsolutePath
06-01 14:29:41.998 1249-1249/de.ecspride.reverseme D/foo: java.io.File
06-01 14:29:41.998 1249-1249/de.ecspride.reverseme D/foo: getAbsolutePath
06-01 14:29:41.998 1249-1249/de.ecspride.reverseme D/foo: dalvik.system.DexClassLoader
06-01 14:29:42.046 1249-1249/de.ecspride.reverseme D/foo: A
06-01 14:29:42.046 1249-1249/de.ecspride.reverseme D/foo: dalvik.system.DexClassLoader
```

Because we are curious, we replace the `delete` method of `java.io.File` with a simple `true` value, so the file won't be deleted. We are doing this using Xposed.

```java
if(lpparam.packageName.equals("de.ecspride.reverseme")) {
  findAndHookMethod("java.io.File", lpparam.classLoader, "delete", (XC_MethodReplacement)XC_MethodReplacement.returnConstant((Boolean)true));
}
```
We found out, that an apk is written into `/data/data/de.ecspride.reverseme/`. By pulling it with `adb pull /data/data/de.ecspride.reverseme/cache/prefix-183764452.apk prefix.apk`, we can have a look at it in jadx-gui. The code could not be recreated again, but there is a md5 value that looks suspicious.

```
22BA039C77C8AEEFBE9DF63F76050ACE
```

By googling it, we got the plain value `Cool`, which is our flag.
