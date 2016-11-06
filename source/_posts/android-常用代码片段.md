title: "android 常用代码片段"
date: 2015-04-21 09:10:32
tags: Android
categories: Android
---


##前言
>Android开发过程可能需要用到的代码片段，一共`35`则。供需要时借鉴参考。
<!--more-->


###精确获取屏幕尺寸

>例如：3.5、4.0、5.0寸屏幕：

```java
public static double getScreenPhysicalSize(Activity ctx) {
        DisplayMetrics dm = new DisplayMetrics();
        ctx.getWindowManager().getDefaultDisplay().getMetrics(dm);
        double diagonalPixels = Math.sqrt(Math.pow(dm.widthPixels, 2) + Math.pow(dm.heightPixels, 2));
        return diagonalPixels / (160 * dm.density);
    }
```

###判断设备是否是平板

>一般是7存以上是平板，此为官方用法。

```java
public static boolean isTablet(Context context) {
        return (context.getResources().getConfiguration().screenLayout & Configuration.SCREENLAYOUT_SIZE_MASK) >= Configuration.SCREENLAYOUT_SIZE_LARGE;
    }
```

###文字根据状态栏更改颜色

>属性为:`android:textColor`
>此代码文件放置在项目的`res/color/`目录下:

```xml
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:color="#53c1bd" android:state_selected="true"/>
    <item android:color="#53c1bd" android:state_focused="true"/>
    <item android:color="#53c1bd" android:state_pressed="true"/>
    <item android:color="#777777"/>
</selector>
```

###背景色根据状态更改颜色

>属性为:`android:backgroup`,如果直接给背景色将会报错:

```xml
<selector xmlns:android="http://schemas.android.com/apk/res/android">

    <item android:state_selected="true"><shape>            
            <gradient android:angle="0" android:centerColor="#00a59f" android:endColor="#00a59f" android:startColor="#00a59f" />
        </shape></item>
    <item android:state_focused="true"><shape>
            <gradient android:angle="0" android:centerColor="#00a59f" android:endColor="#00a59f" android:startColor="#00a59f" />
        </shape></item>
    <item android:state_pressed="true"><shape>
            <gradient android:angle="0" android:centerColor="#00a59f" android:endColor="#00a59f" android:startColor="#00a59f" />
        </shape></item>
    <item><shape>
            <gradient android:angle="0" android:centerColor="#00ff00" android:endColor="00ff00" android:startColor="00ff00" />
        </shape></item>

</selector>
```

###启动App默认的Activity

```java
public static void startApkActivity(final Context ctx, String packageName) {
        PackageManager pm = ctx.getPackageManager();
        PackageInfo pi;
        try {
            pi = pm.getPackageInfo(packageName, 0);
            Intent intent = new Intent(Intent.ACTION_MAIN, null);
            intent.addCategory(Intent.CATEGORY_LAUNCHER);
            intent.setPackage(pi.packageName);

            List<ResolveInfo> apps = pm.queryIntentActivities(intent, 0);

            ResolveInfo ri = apps.iterator().next();
            if (ri != null) {
                String className = ri.activityInfo.name;
                intent.setComponent(new ComponentName(packageName, className));
                ctx.startActivity(intent);
            }
        } catch (NameNotFoundException e) {
            Log.e("startActivity", e);
        }
    }
```

###计算字宽

>如果设置了`textStyle`，还需要进一步设置`TextPaint`。

```java
public static float GetTextWidth(String text, float Size) {
        TextPaint FontPaint = new TextPaint();
        FontPaint.setTextSize(Size);
        return FontPaint.measureText(text);
    }
```

###获取应用程序下所有的Activity

```java
public static ArrayList<String> getActivities(Context ctx) {
      ArrayList<String> result = new ArrayList<String>();
      Intent intent = new Intent(Intent.ACTION_MAIN, null);
      intent.setPackage(ctx.getPackageName());
      for (ResolveInfo info : ctx.getPackageManager().queryIntentActivities(intent, 0)) {
          result.add(info.activityInfo.name);
      }
      return result;
  }
```

###检测字符串中是否包含汉字

```java
public static boolean checkChinese(String sequence) {
        final String format = "[\\u4E00-\\u9FA5\\uF900-\\uFA2D]";
        boolean result = false;
        Pattern pattern = Pattern.compile(format);
        Matcher matcher = pattern.matcher(sequence);
        result = matcher.find();
        return result;
    }
```

###检测字符串中只能包含:中文、数字、下划线(_)、横线(-)

```java
public static boolean checkNickname(String sequence) {
        final String format = "[^\\u4E00-\\u9FA5\\uF900-\\uFA2D\\w-_]";
        Pattern pattern = Pattern.compile(format);
        Matcher matcher = pattern.matcher(sequence);
        return !matcher.find();
    } 
```

###检查又没有应用程序来接受处理你发出的intent

```java
public static boolean isIntentAvailable(Context context, String action) {
        final PackageManager packageManager = context.getPackageManager();
        final Intent intent = new Intent(action);
        List<ResolveInfo> list = packageManager.queryIntentActivities(intent, PackageManager.MATCH_DEFAULT_ONLY);
        return list.size() > 0;
    }
```

###使用`TransitionDrawable`实现渐变效果

>比使用`AlphaAnimation`效果要好，可以避免出现闪烁问题。

```java
private void setImageBitmap(ImageView imageView, Bitmap bitmap) {
        // Use TransitionDrawable to fade in.
        final TransitionDrawable td = new TransitionDrawable(new Drawable[] { new ColorDrawable(android.R.color.transparent), new BitmapDrawable(mContext.getResources(), bitmap) });
        //noinspection deprecation
            imageView.setBackgroundDrawable(imageView.getDrawable());
        imageView.setImageDrawable(td);
        td.startTransition(200);
    }
```

###扫描指定文件

>用途:从本软件新增、修改、删除图片、文件某一个文件（音频、视频）徐奥更新系统媒体库时使用。不必扫描整个SD卡。

```java
sendBroadcast(new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE, uri));
```

###Dip转px

>用途：难免在Activity代码中设置位置、大小等，本方法就很有用了！ 

```java
public static int dipToPX(final Context ctx, float dip) {
        return (int)TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, dip, ctx.getResources().getDisplayMetrics());
    }
```

###获取已经安装的APK路径

>代码:

```java
PackageManager pm = getPackageManager();

        for (ApplicationInfo app : pm.getInstalledApplications(0)) {
             Log.d("PackageList", "package: " + app.packageName + ", sourceDir: " + app.sourceDir);
        }
```
>将会获得以下输出:

```java
package: com.tmobile.thememanager, sourceDir: /system/app/ThemeManager.apk
        package: com.touchtype.swiftkey, sourceDir: /data/app/com.touchtype.swiftkey-1.apk
```

###多进程`Preferences`数据共享

```java
public static void putStringProcess(Context ctx, String key, String value) {
        SharedPreferences sharedPreferences = ctx.getSharedPreferences("preference_mu", Context.MODE_MULTI_PROCESS);
        Editor editor = sharedPreferences.edit();
        editor.putString(key, value);
        editor.commit();
    }

    public static String getStringProcess(Context ctx, String key, String defValue) {
        SharedPreferences sharedPreferences = ctx.getSharedPreferences("preference_mu", Context.MODE_MULTI_PROCESS);
        return sharedPreferences.getString(key, defValue);
    }
```

###泛型`ArrayList`转数组

```java
@SuppressWarnings("unchecked")
    public static <T> T[] toArray(Class<?> cls, ArrayList<T> items) {
        if (items == null || items.size() == 0) {
            return (T[]) Array.newInstance(cls, 0);
        }
        return items.toArray((T[]) Array.newInstance(cls, items.size()));
    }
```

###保存回复ListView当前为位置

>可以保存在`Preference`中或者是数据库中，数据加载完后再设置：

```java
private void saveCurrentPosition() {
        if (mListView != null) {
            int position = mListView.getFirstVisiblePosition();
            View v = mListView.getChildAt(0);
            int top = (v == null) ? 0 : v.getTop();
            //保存position和top
        }
    }
    
    private void restorePosition() {
        if (mFolder != null && mListView != null) {
            int position = 0;//取出保存的数据
            int top = 0;//取出保存的数据
            mListView.setSelectionFromTop(position, top);
        }
    }
```

###调用`便携式热点`和`数据共享`设置

```java
public static Intent getHotspotSetting() {
        Intent intent = new Intent();
        intent.setAction(Intent.ACTION_MAIN);
        ComponentName com = new ComponentName("com.android.settings", "com.android.settings.TetherSettings");
        intent.setComponent(com);
        return intent;
    }
```

###格式化输出IP地址

```java
public static String getIp(Context ctx) {
        return Formatter.formatIpAddress((WifiManager) ctx.getSystemService(Context.WIFI_SERVICE).getConnectionInfo().getIpAddress());
    }
```

###文件夹排序

>先文件夹排序，后文件排序

```java
public static void sortFiles(File[] files) {
        Arrays.sort(files, new Comparator<File>() {

            @Override
            public int compare(File lhs, File rhs) {
                //返回负数表示o1 小于o2，返回0 表示o1和o2相等，返回正数表示o1大于o2。 
                boolean l1 = lhs.isDirectory();
                boolean l2 = rhs.isDirectory();
                if (l1 && !l2)
                    return -1;
                else if (!l1 && l2)
                    return 1;
                else {
                    return lhs.getName().compareTo(rhs.getName());
                }
            }
        });
    }
```

###发送不重复的通知（Notification）

>  关键点在这个`requestCode`，这里使用的是当前系统时间，巧妙的保证了每次都是一个新的`Notification`产生。

```java
public static void sendNotification(Context context, String title,
            String message, Bundle extras) {
        Intent mIntent = new Intent(context, FragmentTabsActivity.class);
        mIntent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
        mIntent.putExtras(extras);

        int requestCode = (int) System.currentTimeMillis();

        PendingIntent mContentIntent = PendingIntent.getActivity(context,
                requestCode, mIntent, 0);

        Notification mNotification = new NotificationCompat.Builder(context)
                .setContentTitle(title).setSmallIcon(R.drawable.app_icon)
                .setContentIntent(mContentIntent).setContentText(message)
                .build();
        mNotification.flags |= Notification.FLAG_AUTO_CANCEL;
        mNotification.defaults = Notification.DEFAULT_ALL;

        NotificationManager mNotificationManager = (NotificationManager) context
                .getSystemService(Context.NOTIFICATION_SERVICE);

        mNotificationManager.notify(requestCode, mNotification);
    }
```

###代码设置`TextView`的样式

> 使用过`自定义Dialog`可能马上会想到用如下代码：

```java
 new TextView(this,null,R.style.text_style);
```
>  但你运行这代码你会发现毫无作用！正确用法：

```java
 new TextView(new ContextThemeWrapper(this, R.style.text_style));
```

###ip地址转成8位16进制串

> ip:192.168.68.128 16 =>hex :c0a84480

```java
 /** ip转16进制 */
    public static String ipToHex(String ips) {
        StringBuffer result = new StringBuffer();
        if (ips != null) {
            StringTokenizer st = new StringTokenizer(ips, ".");
            while (st.hasMoreTokens()) {
                String token = Integer.toHexString(Integer.parseInt(st.nextToken()));
                if (token.length() == 1)
                    token = "0" + token;
                result.append(token);
            }
        }
        return result.toString();
    }

    /** 16进制转ip */
    public static String texToIp(String ips) {
        try {
            StringBuffer result = new StringBuffer();
            if (ips != null && ips.length() == 8) {
                for (int i = 0; i < 8; i += 2) {
                    if (i != 0)
                        result.append('.');
                    result.append(Integer.parseInt(ips.substring(i, i + 2), 16));
                }
            }
            return result.toString();
        } catch (NumberFormatException ex) {
            Logger.e(ex);
        }
        return "";
    }
```

###`WebView`保留缩放功能但隐藏缩放控件

>注意：`setDisplayZoomControls`是在API Level 11中新增。

```java
 mWebView.getSettings().setSupportZoom(true);
        mWebView.getSettings().setBuiltInZoomControls(true);
        if (DeviceUtils.hasHoneycomb())
              mWebView.getSettings().setDisplayZoomControls(false);
```

###获取网络类型名称

```java
 public static String getNetworkTypeName(Context context) {
        if (context != null) {
            ConnectivityManager connectMgr = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
            if (connectMgr != null) {
                NetworkInfo info = connectMgr.getActiveNetworkInfo();
                if (info != null) {
                    switch (info.getType()) {
                    case ConnectivityManager.TYPE_WIFI:
                        return "WIFI";
                    case ConnectivityManager.TYPE_MOBILE:
                        return getNetworkTypeName(info.getSubtype());
                    }
                }
            }
        }
        return getNetworkTypeName(TelephonyManager.NETWORK_TYPE_UNKNOWN);
    }

    public static String getNetworkTypeName(int type) {
        switch (type) {
        case TelephonyManager.NETWORK_TYPE_GPRS:
            return "GPRS";
        case TelephonyManager.NETWORK_TYPE_EDGE:
            return "EDGE";
        case TelephonyManager.NETWORK_TYPE_UMTS:
            return "UMTS";
        case TelephonyManager.NETWORK_TYPE_HSDPA:
            return "HSDPA";
        case TelephonyManager.NETWORK_TYPE_HSUPA:
            return "HSUPA";
        case TelephonyManager.NETWORK_TYPE_HSPA:
            return "HSPA";
        case TelephonyManager.NETWORK_TYPE_CDMA:
            return "CDMA";
        case TelephonyManager.NETWORK_TYPE_EVDO_0:
            return "CDMA - EvDo rev. 0";
        case TelephonyManager.NETWORK_TYPE_EVDO_A:
            return "CDMA - EvDo rev. A";
        case TelephonyManager.NETWORK_TYPE_EVDO_B:
            return "CDMA - EvDo rev. B";
        case TelephonyManager.NETWORK_TYPE_1xRTT:
            return "CDMA - 1xRTT";
        case TelephonyManager.NETWORK_TYPE_LTE:
            return "LTE";
        case TelephonyManager.NETWORK_TYPE_EHRPD:
            return "CDMA - eHRPD";
        case TelephonyManager.NETWORK_TYPE_IDEN:
            return "iDEN";
        case TelephonyManager.NETWORK_TYPE_HSPAP:
            return "HSPA+";
        default:
            return "UNKNOWN";
        }
    }
```

###Android解压Zip包

```java
/**
     * 解压一个压缩文档 到指定位置
     * 
     * @param zipFileString 压缩包的名字
     * @param outPathString 指定的路径
     * [url=home.php?mod=space&uid=2643633]@throws[/url] Exception
     */
    public static void UnZipFolder(String zipFileString, String outPathString) throws Exception {
        java.util.zip.ZipInputStream inZip = new java.util.zip.ZipInputStream(new java.io.FileInputStream(zipFileString));
        java.util.zip.ZipEntry zipEntry;
        String szName = "";

        while ((zipEntry = inZip.getNextEntry()) != null) {
            szName = zipEntry.getName();

            if (zipEntry.isDirectory()) {

                // get the folder name of the widget
                szName = szName.substring(0, szName.length() - 1);
                java.io.File folder = new java.io.File(outPathString + java.io.File.separator + szName);
                folder.mkdirs();

            } else {

                java.io.File file = new java.io.File(outPathString + java.io.File.separator + szName);
                file.createNewFile();
                // get the output stream of the file
                java.io.FileOutputStream out = new java.io.FileOutputStream(file);
                int len;
                byte[] buffer = new byte[1024];
                // read (len) bytes into buffer
                while ((len = inZip.read(buffer)) != -1) {
                    // write (len) byte from buffer at the position 0
                    out.write(buffer, 0, len);
                    out.flush();
                }
                out.close();
            }
        }//end of while

        inZip.close();

    }//end of func
```

###从`assets`中读取文本和图片资源

```java
 /** 从assets 文件夹中读取文本数据 */
    public static String getTextFromAssets(final Context context, String fileName) {
        String result = "";
        try {
            InputStream in = context.getResources().getAssets().open(fileName);
            // 获取文件的字节数
            int lenght = in.available();
            // 创建byte数组
            byte[] buffer = new byte[lenght];
            // 将文件中的数据读到byte数组中
            in.read(buffer);
            result = EncodingUtils.getString(buffer, "UTF-8");
            in.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
    
    /** 从assets 文件夹中读取图片 */
    public static Drawable loadImageFromAsserts(final Context ctx, String fileName) {
        try {
            InputStream is = ctx.getResources().getAssets().open(fileName);
            return Drawable.createFromStream(is, null);
        } catch (IOException e) {
            if (e != null) {
                e.printStackTrace();
            }
        } catch (OutOfMemoryError e) {
            if (e != null) {
                e.printStackTrace();
            }
        } catch (Exception e) {
            if (e != null) {
                e.printStackTrace();
            }
        }
        return null;
    }
```

###展开、收起状态栏

>用途：可用于点击`Notifacation`之后收起状态栏

```java
public static final void collapseStatusBar(Context ctx) {
        Object sbservice = ctx.getSystemService("statusbar");
        try {
            Class<?> statusBarManager = Class.forName("android.app.StatusBarManager");
            Method collapse;
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
                collapse = statusBarManager.getMethod("collapsePanels");
            } else {
                collapse = statusBarManager.getMethod("collapse");
            }
            collapse.invoke(sbservice);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static final void expandStatusBar(Context ctx) {
        Object sbservice = ctx.getSystemService("statusbar");
        try {
            Class<?> statusBarManager = Class.forName("android.app.StatusBarManager");
            Method expand;
            if (Build.VERSION.SDK_INT >= 17) {
                expand = statusBarManager.getMethod("expandNotificationsPanel");
            } else {
                expand = statusBarManager.getMethod("expand");
            }
            expand.invoke(sbservice);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

###获取状态栏高度

```java
public static int getStatusBarHeight(Context context){
        Class<?> c = null;
        Object obj = null;
        Field field = null;
        int x = 0, statusBarHeight = 0;
        try {
            c = Class.forName("com.android.internal.R$dimen");
            obj = c.newInstance();
            field = c.getField("status_bar_height");
            x = Integer.parseInt(field.get(obj).toString());
            statusBarHeight = context.getResources().getDimensionPixelSize(x);
        } catch (Exception e1) {
            e1.printStackTrace();
        }
        return statusBarHeight;
    }
```

###`ListView`使用`ViewHolder`极简写法
>用起来非常简练，将ViewHolder隐于无形。

```java
public static <T extends View> T getAdapterView(View convertView, int id) {
        SparseArray<View> viewHolder = (SparseArray<View>) convertView.getTag();
        if (viewHolder == null) {
            viewHolder = new SparseArray<View>();
            convertView.setTag(viewHolder);
        }
        View childView = viewHolder.get(id);
        if (childView == null) {
            childView = convertView.findViewById(id);
            viewHolder.put(id, childView);
        }
        return (T) childView;
    }
```
>用法:

```java
@Override
    public View getView(int position, View convertView, ViewGroup parent) {
        if (convertView == null) {
            convertView = LayoutInflater.from(getActivity()).inflate(R.layout.fragment_feed_item, parent, false);
        }

        ImageView thumnailView = getAdapterView(convertView, R.id.video_thumbnail);
        ImageView avatarView =  getAdapterView(convertView, R.id.user_avatar);
        ImageView appIconView = getAdapterView(convertView, R.id.app_icon);
```

###设置Activity透明

> 说明：AppBaseTheme一般是你application指定的android:theme是啥这里就是啥，否则Activity内部的空间风格可能不一致。
>用途：用于模拟Dialog效果，比如再Service中没法用Dialog，就可以用Activity来模拟。

```xml
    <style name="TransparentActivity" parent="AppBaseTheme">
        <item name="android:windowBackground">@android:color/transparent</item>
        <item name="android:colorBackgroundCacheHint">@null</item>
        <item name="android:windowIsTranslucent">true</item>
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowContentOverlay">@null</item>
    </style>
```

###代码切换全屏

>注意：切换到全屏时，底部的虚拟按键仍然是显示的。次方法可多次调用用于切换.
>用途：播放器界面经常会用到.

```java
//切换到全屏
        getWindow().clearFlags(WindowManager.LayoutParams.FLAG_FORCE_NOT_FULLSCREEN);
        getActivity().getWindow().addFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);

        //切换到非全屏
        getWindow().clearFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);
        getWindow().addFlags(WindowManager.LayoutParams.FLAG_FORCE_NOT_FULLSCREEN);
```

###调用开发者选项中显示触摸位置功能

> 设置1显示，设置0不显示。

```java
android.provider.Settings.System.putInt(getContentResolver(), "show_touches", 1);
```

###获取设备上已安装并且可启动的应用列表

>注意：使用`getInstalledApplications`会返回很多无法启动甚至没有图标的系统应用。`ResolveInfo.activityInfo.applicationInfo`也能取到你想要的数据。

```java
Intent intent = new Intent(Intent.ACTION_MAIN);
            intent.addCategory(Intent.CATEGORY_LAUNCHER);

            List<ResolveInfo> activities = getPackageManager().queryIntentActivities(intent, 0)
```



<font size=1>本文摘抄自[eoe论坛](http://www.eoeandroid.com/thread-570919-1-1.html),非本博原创。</font>
