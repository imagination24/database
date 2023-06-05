# Android-如何获取用户实时应用使用时长




## 背景
&emsp;&emsp;从何说起呢？如你所知，我是一个Flutter开发工程师，有Android原生应用的开发背景。在之前的一个项目中，我负责Android端的功能实现，然后我遇到一个获取用户实时使用时长的需求。一番调研之后，发现Android有提供原生的Api，而且Flutter社区上已经有实现好的插件[（usage_stats插件地址）](https://pub.dev/packages/usage_stats)，使用下图这个方法即可获取原生Api的返回数据
![image](https://i.imgur.com/6Xpjxj5.png)


但简单使用过后，通过检查结果发现这个Api的返回数据并不准确，或者说有大概一天的延迟，是的足足有一天。在Stack-OverFlow搜索发现这个问题是普遍的，也就是说所有人对这个Api调用的返回结果都有延迟，显然这是Api的问题，之后我查阅文档后发现这个一段说明
![image](https://i.imgur.com/D9oLa6R.png)

根据结果推测，我设定的开始时间和结束时间应该是被改了，又经过无数次尝试后，我放弃使用这个Api，非常遗憾。但这个功能该如何实现？感谢stack-overflow，我从中得到了思路：通过使用得知
![image](https://i.imgur.com/BrGojto.png)


上图这个方法的返回数据是无误差无延时的，那么我们可以**通过计算前台启动数据来累计应用的使用时间**。

## 实现

* 权限

**(至少需要 API 级别 22！)**
首先我们需要获取手机的使用记录访问权限。
什么是使用记录访问权限呢？这是在Android5.0（Api level 21）新添加的，通过该权限我们可以查看设备上其它应用使用情况的统计信息等。

添加权限到AndroidManifest中
```
<uses-permission  android:name="android.permission.PACKAGE_USAGE_STATS" tools:ignore="ProtectedPermissions" />
```

权限申请方法
```
  private boolean isUsagePermission(){
    if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.KITKAT) {
      AppOpsManager appOpsManager = (AppOpsManager) context.getSystemService(Context.APP_OPS_SERVICE);
      int mode = appOpsManager.checkOpNoThrow(AppOpsManager.OPSTR_GET_USAGE_STATS,Process.myUid(), context.getPackageName());
      if(mode == AppOpsManager.MODE_ALLOWED){
        return true;
      }
    }
    return false;
  }

  private void grantUsagePermission(){
    if(!isUsagePermission()){
      Intent intent = new Intent(Settings.ACTION_USAGE_ACCESS_SETTINGS);
      intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
      context.startActivity(intent);
    }
  }
```

如果需要在Flutter中使用，仅需实现插件并调用方法即可。需要注意的是如果要使用Flutter在后台查询应用使用数据,需要把权限申请和查询应用使用数据分离，因为后台不能允许任何和UI有关的应用，事实上是任何需要使用到Activity提供的Context的程序都不能在后台运行。

* 实现
```
///查询需要获取context,如果要做后台使用可以搭配无障碍，无障碍全局提供context
///beginTime:开始时间
///endTime:结束时间
public static List<Map<String, Object>> queryUsageTime(Context context, long beginTime, long endTime){
        ///记录所有的包的使用数据
        HashMap<String,Integer> packageMap = new HashMap<>();
        ///记录当前事件
        UsageEvents.Event currentEvent;
        ///所有事件
        List<UsageEvents.Event> allEvents = new ArrayList<>();
        HashMap<String, Integer> appUsageMap = new HashMap<>();
        ///结果
        List<Map<String,Object>> result = new ArrayList<>();
        UsageStatsManager usageStatsManager = (UsageStatsManager) context.getSystemService(Context.USAGE_STATS_SERVICE);
        ///获取所有事件
        UsageEvents usageEvents = usageStatsManager.queryEvents(beginTime, endTime);
        ///获取运行过的全部包
        while (usageEvents.hasNextEvent()){
            currentEvent = new UsageEvents.Event();
            usageEvents.getNextEvent(currentEvent);
            if (currentEvent.getEventType()==UsageEvents.Event.ACTIVITY_PAUSED||currentEvent.getEventType()==UsageEvents.Event.ACTIVITY_RESUMED){
                if (packageMap.isEmpty()||packageMap.get(currentEvent.getPackageName())==null){
                    packageMap.put(currentEvent.getPackageName(),0);
                }
            }
        }
        ///计算单个包名的使用时间
        for (String packageName:packageMap.keySet()){
            //这里有个特别的点，本来以为查一次获取全部数据就够了，但跑起来发现如果只差一次，获取的usage不知为何到第二次循环时，内容为空，因此每次都查
            UsageStatsManager usageStatsManagers = (UsageStatsManager) context.getSystemService(Context.USAGE_STATS_SERVICE);
            UsageEvents usageEvent = usageStatsManagers.queryEvents(beginTime, endTime);
            ///获取有关包名的全部事件
            while (usageEvent.hasNextEvent()) {
                currentEvent = new UsageEvents.Event();
                usageEvent.getNextEvent(currentEvent);
                if(currentEvent.getPackageName().equals(packageName) || packageName == null) {
                    if (currentEvent.getEventType() == UsageEvents.Event.ACTIVITY_RESUMED
                            || currentEvent.getEventType() == UsageEvents.Event.ACTIVITY_PAUSED) {
                        allEvents.add(currentEvent);
                        String key = currentEvent.getPackageName();
                        if (appUsageMap.get(key) == null)
                            appUsageMap.put(key, 0);
                    }
                }
            }
            ///遍历包名的全部事件
            for (int i = 0; i < allEvents.size() - 1; i++) {
                UsageEvents.Event E0 = allEvents.get(i);
                UsageEvents.Event E1 = allEvents.get(i + 1);
                ///如果事件类型为恢复或者终止，则计算时间差
                if (E0.getEventType() == UsageEvents.Event.ACTIVITY_RESUMED
                        && E1.getEventType() == UsageEvents.Event.ACTIVITY_PAUSED
                        && E0.getClassName().equals(E1.getClassName())) {
                    int diff = (int)(E1.getTimeStamp() - E0.getTimeStamp());
                    diff /= 1000;
                    Integer prev = appUsageMap.get(E0.getPackageName());
                    if(prev == null) prev = 0;
                    appUsageMap.put(E0.getPackageName(), prev + diff);
                }
                if ((i+1)==(allEvents.size()-1)&&(allEvents.get(i+1).getEventType())==(UsageEvents.Event.ACTIVITY_RESUMED)){
                    UsageEvents.Event lastEvent = allEvents.get(allEvents.size() - 1);///未关闭的事件的时间
                    int diff = (int)System.currentTimeMillis() - (int)lastEvent.getTimeStamp();
                    diff /= 1000;
                    Integer prev = appUsageMap.get(lastEvent.getPackageName());
                    if(prev == null) prev = 0;
                    appUsageMap.put(lastEvent.getPackageName(), prev + diff);
                }
            }
            ///加入结果
            if (appUsageMap.get(packageName)!=null&&packageName!=null){
                result.add(Map.of("packageName",packageName,"duration",appUsageMap.get(packageName)));
            }
            allEvents.clear();
        }
        return result;
    }
```
如果不想写，可以直接引入我已经实现好的插件
[Github插件地址](https://github.com/imagination24/usage_state)

