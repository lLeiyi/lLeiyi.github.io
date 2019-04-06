---
title: ArcEngine中IMap的选择集刷新问题
tags:
- ArcEngine
- ISelectEvents
categories:
- ArcEngine
- 记录&&踩过的坑
---

# 1.问题描述
    通过以下方式可以很便捷的往选择集内添加要素，但是却无法触发AxMapControl下的OnSelectionChanged事件。
```C#{.line-numbers}
public static void SelectFeatures(IFeatureLayer featureLayer, int[] OIDs)
{
    if (pFeatureLayer == null || OIDs == null)
    {
        return;
    }
    IFeatureSelection featureSelection = featureLayer as IFeatureSelection;
    if (featureSelection != null)
    {
        IGeoDatabaseBridge2 geoDatabaseBridge2 = new GeoDatabaseHelperClass();
        geoDatabaseBridge2.AddList(featureSelection.SelectionSet, ref OIDs);
    }
}
```
# 2.通过ISelectEvents接口解决
    可以通过ISelectEvents接口来解决这个问题,该接口可由IMap接口QI。
```C#{.line-numbers}
ISelectionEvents selectEvents = map as ISelectionEvents;
if (selectEvents != null)
{
    selectEvents.SelectionChanged();
}
```
