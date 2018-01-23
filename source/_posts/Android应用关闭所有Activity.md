title: "Android应用关闭所有Activity"
date: 2015-04-05 23:45:54
categories: Android
tags:
- Android
- Activity
---
### 前言
>在Android开发中，常常想在某一个Activity或者Service中关闭所有打开的Activity。一直没有十分统一可行的方法。下面结合网络和自己的在项目中遇到的情况，给出一种实现方法。
><!-- more -->
- 创建一个类CloseAllActivity继承`Application`类。
- 在该类内部创建一个`List`集合，用于保存打开的Activity。
- 创建一个`public void addActivity(Activity activity)`方法,用于在Activity的`onCreate(Bundle bundle)`中调用。使该Activity被添加到List集合中。
- 创建一个静态对象，并将其实例化一次，避免每次使用都得实例化。
- 在需要添加到List集合中的Activity中，调用。
- 编写一个exit()函数，关闭List集合中的所有Activity。

>CloseAllActivity类具体代码如下:

```java
public class CloseAllActivity extends Application {

		private List<Activity> activityList = new LinkedList<Activity>();
		private static CloseAllActivity instance;// 创建的静态对象，避免每次使用的实例化

		private CloseAllActivity() {
			// 空构造函数
		}

		// 实例化一次
		public synchronized static CloseAllActivity getInstance() {
			if (instance == null) {
				instance = new CloseAllActivity();
			}
			return instance;
		}

		/**
		 * 
		 * @param activity
		 *            将当前activity添加到activity集合中
		 */
		public void addActivity(Activity activity) {
			activityList.add(activity);
		}

		public void exit() {
			try {
				for (Activity activity : activityList) {
					if (activity != null) {
						activity.finish();
					}
				}
			} catch (Exception e) {
				Log.e("xiamo", "关闭activity错误：" + e.toString());
			} finally {
				System.exit(0);
			}
		}

	}
```

>需要添加到List集合中关闭的Activity中，引用的方法为:
```java
public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        CloseAllActivity.getInstance().addActivity(this);
          ....
              ....
	}
```