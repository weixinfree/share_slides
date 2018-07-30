title: clean_code
speaker: 王伟

[slide]

# 编写清晰易懂的函数的一些技巧

[slide]

1. 单一抽象层次（SAL）
2. 最小缩进
3. 复合表达式命名
4. 作用域最小化
5. 最简单变化原则
6. 空格，留白的艺术

[slide]

# 单一抽象层次（SAL）

[slide]

## 一个函数/方法中所有操作处于相同逻辑层次。

```java
    void 放大象进入冰箱() {
        举起胳膊;
        伸手进冰箱把手;
        五个手指握紧;                              void 放大象进入冰箱() {
        手用例拉回一米;                                 打开冰箱门();
        while(冰箱门 < 90度) 手再度拉回20厘米;  ==>       推动大象();
        推动大象();                                    关闭冰箱门();
        ......;                                    } 
        if(门关不上){
            抬脚();
            用力踹大象皮鼓();
            用力关门();
        }
        .....
    }
```

[slide]

## 单一抽象层次 => 思维保持在同一层次上，不跳跃

[slide]
# 最小缩进（negative）

```java
if (TextUtils.isEmpty(creatorCity)) {
    return;
}
if (null != sameCitySwitchInfoModel && sameCitySwitchInfoModel.isOpen()) {
    if (!TextUtils.isEmpty(sameCitySwitchInfoModel.content)) {
        String[] whiteCity = sameCitySwitchInfoModel.content.split(",");
        if (null != whiteCity) {
            mCityWhiteNames.clear();
            for (String cityName : whiteCity) {
                if (TextUtils.isEmpty(cityName)) {
                    continue;
                }
                if (TextUtils.equals(cityName, creatorCity)) {
                    mCityWhiteNames.add(cityName);
                }
            }
        }
    }
}
```

[slide]
# 最小缩进（positive）

```java
if (TextUtils.isEmpty(creatorCity)) return;

final boolean isSameCitySwitchOpen = null != sameCitySwitchInfoModel && sameCitySwitchInfoModel.isOpen();
if (!isSameCitySwitchOpen) return;

final String content = sameCitySwitchInfoModel.content;
if (TextUtils.isEmpty(content)) return;

final String[] whiteCity = content.split(",");
if (whiteCity == null) return;


mCityWhiteNames.clear();
for (String cityName : whiteCity) {
    if (TextUtils.equals(cityName, creatorCity)) {
        mCityWhiteNames.add(cityName);
    }
}
```

[slide]

## 什么导致了缩进

1. 控制结构 `if for while`
2. 业务逻辑 
3. 卫语句(妨碍对业务逻辑的理解)

[slide]
# 最小缩进 => 凸显主干逻辑，减轻卫语句的思维负担

[slide]
# 复合表达式命名 （negative）

```java
if (null != mDiscoverCategoryModel 
	&& null != mDiscoverCategoryModel.lives 
	&& mDiscoverCategoryModel.lives.size() > 0 
	&& null != mDataList 
	&& mDataList.size() > 0) {

	...
}
```

[slide]
# 复合表达式命名 （positive）

```java
final boolean isLiveValid = null != mDiscoverCategoryModel && null != mDiscoverCategoryModel.lives 
	&& mDiscoverCategoryModel.lives.size() > 0;

final boolean isDataListValid = null != mDataList && mDataList.size() > 0;

if (isLiveValid && isDataListValid) {
	...
}
```

[slide]

# 最小, 最简单
1. 最小作用域
2. 最简单变化 (final, 多线程)
3. 最小的访问范围 
4. 离使用上下文最近

[slide]

# 例子 (原始代码)
```java
private void demo() {
	if (null != mDiscoverCategoryModel 
	&& null != mDiscoverCategoryModel.lives 
	&& mDiscoverCategoryModel.lives.size() > 0 
	&& null != mDataList && mDataList.size() > 0) {

    HallItemModel removeItem = null;
    for (HallItemModel hallItemModel : mDataList) {
        if (HallViewType.HallHot.TYPE_DISCOVER == hallItemModel.type) {
            removeItem = hallItemModel;
        }
    }
    if (null != removeItem) {
        mDataList.remove(removeItem);
    }
    HallItemModel hallItemModel = new HallItemModel();
    hallItemModel.type = HallViewType.HallHot.TYPE_DISCOVER;
    hallItemModel.discoverCategoryModel = mDiscoverCategoryModel;
    mDataList.add(1, hallItemModel);
}
```

[slide]
# 例子 (使用上面的技巧后)
```java
private void demo() {
	final DiscoverCategoryModel discoverCategoryModel = mDiscoverCategoryModel;
	final boolean isLiveValid = null != discoverCategoryModel 
		&& null != discoverCategoryModel.lives 
		&& discoverCategoryModel.lives.size() > 0;
	if (!isLiveValid) return;

	final List<HallItemModel> dataList = mDataList;
	final boolean isDataListValid = null != dataList && dataList.size() > 0;
	if (!isDataListValid) return;

	removeLastDiscoverItem(dataList);
	dataList.add(1, newDiscoverHallItem(discoverCategoryModel));
}
```

[slide]

# 例子 (使用上面的技巧后, 激进版本)

```java
private void demo() {
    final DiscoverCategoryModel discoverCategoryModel = mDiscoverCategoryModel;
    if (!isLiveValid(discoverCategoryModel)) return;

    final List<HallItemModel> dataList = mDataList;
    if (!isDataListValid(dataList)) return;

    removeLastDiscoverItem(dataList);
    dataList.add(1, newDiscoverHallItem(discoverCategoryModel));
}
```

[slide]

1. 代码是最重要的用途给人看的，可理解性非常重要
2. 代码维护时间 >> 调试时间 >> 开发时间
2. 编程是管理复杂性
3. kiss `keep simple, keep stupid`
4. 本质复杂性, 偶然复杂性
5. ...

[slide]

# `先清晰，后简单`

[slide]

# Thanks
