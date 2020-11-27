<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [概况](#%E6%A6%82%E5%86%B5)
      - [运行时核心代码](#%E8%BF%90%E8%A1%8C%E6%97%B6%E6%A0%B8%E5%BF%83%E4%BB%A3%E7%A0%81)
        - [编辑器核心代码](#%E7%BC%96%E8%BE%91%E5%99%A8%E6%A0%B8%E5%BF%83%E4%BB%A3%E7%A0%81)
- [文本](#%E6%96%87%E6%9C%AC)
    - [字符串池](#%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%B1%A0)
    - [Codekey](#codekey)
    - [PrefabKey](#prefabkey)
      - [导出](#%E5%AF%BC%E5%87%BA)
      - [如何本地化](#%E5%A6%82%E4%BD%95%E6%9C%AC%E5%9C%B0%E5%8C%96)
    - [CommandKey](#commandkey)
      - [导出](#%E5%AF%BC%E5%87%BA-1)
      - [本地化](#%E6%9C%AC%E5%9C%B0%E5%8C%96)
    - [TableKey](#tablekey)
      - [本地化](#%E6%9C%AC%E5%9C%B0%E5%8C%96-1)
  - [哈希冲突](#%E5%93%88%E5%B8%8C%E5%86%B2%E7%AA%81)
      - [解决办法](#%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95)
- [资源](#%E8%B5%84%E6%BA%90)
    - [资源目录介绍](#%E8%B5%84%E6%BA%90%E7%9B%AE%E5%BD%95%E4%BB%8B%E7%BB%8D)
    - [资源自动替换<font color="#ff0000">非常重要</font>](#%E8%B5%84%E6%BA%90%E8%87%AA%E5%8A%A8%E6%9B%BF%E6%8D%A2font-colorff0000%E9%9D%9E%E5%B8%B8%E9%87%8D%E8%A6%81font)
      - [备注](#%E5%A4%87%E6%B3%A8)
    - [资源管理](#%E8%B5%84%E6%BA%90%E7%AE%A1%E7%90%86)
    - [翻译](#%E7%BF%BB%E8%AF%91)
    - [资源加载](#%E8%B5%84%E6%BA%90%E5%8A%A0%E8%BD%BD)
  - [翻译流程](#%E7%BF%BB%E8%AF%91%E6%B5%81%E7%A8%8B)
  - [多语言切换流程](#%E5%A4%9A%E8%AF%AD%E8%A8%80%E5%88%87%E6%8D%A2%E6%B5%81%E7%A8%8B)
  - [翻译流程](#%E7%BF%BB%E8%AF%91%E6%B5%81%E7%A8%8B-1)
  - [编辑器预览](#%E7%BC%96%E8%BE%91%E5%99%A8%E9%A2%84%E8%A7%88)
- [操作文档](#%E6%93%8D%E4%BD%9C%E6%96%87%E6%A1%A3)
    - [配置(csv)](#%E9%85%8D%E7%BD%AEcsv)
    - [打表](#%E6%89%93%E8%A1%A8)
  - [查看bytes](#%E6%9F%A5%E7%9C%8Bbytes)
  - [触发器（200812补充）](#%E8%A7%A6%E5%8F%91%E5%99%A8200812%E8%A1%A5%E5%85%85)
  - [美术资源](#%E7%BE%8E%E6%9C%AF%E8%B5%84%E6%BA%90)
    - [图集](#%E5%9B%BE%E9%9B%86)
    - [Prefab](#prefab)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 概况

本地化包含以下内容

1. 文本
2. 资源
3. sdk（这块此处不作说明了）

##### 运行时核心代码

```
Localization/
├── Channel
│   ├── ChannelManager.cs 地区管理器
│   └── Channels
│       ├── BaseChannel.cs 地区基类
│       ├── ChinaChannel.cs 
│       ├── HongKongChannel.cs
│       ├── JapanChannel.cs
│       ├── KoreaChannel.cs
│       └── NorthAmericaChannel.cs
├── Component
│   ├── MLocalizationFontAsset.cs 多字体
│   ├── UIAtlasLocal.cs 图集本地化组件
│   ├── UILocalBase.cs 本地化组件基类
│   ├── UIStringLocal.cs 字符串本地化组件
│   └── UITextureLocal.cs 贴图本地化组件
├── Localization.cs 本地化池，大多数是代码key使用
├── ManualMap
│   ├── IManualMap.cs CodeKey基类
│   ├── MEditorManualMap.cs 编辑器用CodeKey解释器
│   └── MRuntimeManualMap.cs 运行时CodeKey解释器
└── StringPool
    ├── IStringPool.cs 字符串池基类
    ├── MEditorStringPool.cs 编辑器字符串池基类
    ├── MRuntimeStringPool.cs 运行时字符串池基类
    └── StringPoolManager.cs 字符串池管理器
```

###### 编辑器核心代码

```
Localization
├── KeyExporter
│   ├── CodeKeyExporter.cs CodeKey导出器
│   ├── CommandBlockHelper.cs 
│   ├── CommandBlockTreeExporter.cs 剧本上下文导出
│   ├── CommandScriptKeyExporter.cs 剧本Key导出器
│   ├── LocalizationKeyComment.cs 本地化Key描述文件，主要用于描述文本来源
│   ├── LocalizationKeyExporter.cs 
│   ├── TableKeyExporter.cs 表Key导出器
│   └── UIKeyExporter.cs PrefabKey导出器
├── LocalizationBridge.cs 本地化Bridge，调用本地化接口都走这里
├── Translater
│   ├── LocalizationMerger.cs 翻译合并
│   ├── LocalizationTranslater.cs 翻译
│   └── MiniStringPoolTranslater.cs ministring翻译
└── Utillity
    ├── LocalizationForBackEnd.cs 为后台使用本地化文件处理
    ├── LocalizationTextStringFile.cs 本地化文件
    ├── LocalizationTranslateTextStringFile.cs 本地化翻译文件
    └── TMPAssetExporter.cs TMP资源处理
```



## 文本

由于历史原因，我们项目将文本分为以下四块：

1. 代码文本，存储在(config/Assets/Resources/Localization/ManualLocalizationTable_Editor.txt)
2. Prefab文本，存储在prefab中(clientproj/Assets/artres/Resources/UI/Prefabs，以及对应的海外prefab)
3. 剧本文本，存储在(config/Assets/Resources/CommandScrip)
4. 表文本，存储在csv中(config/Table/CSV，以及对应的海外增量表中)

#### 字符串池

为了方便，在编辑器下和最终运行时使用的字符串池不一样，运行时所有字符串都会汇聚到一个与语言相关的bytes中，比如Chinese.bytes

#### Codekey

使用方式：

```c#
Localization.singleton.Get("AUTO_BATTLE_TIPS_4")
```

通过unity的本地化文本工具可进行快捷、方便的操作，存储为ManualLocalizationTable_Editor.txt（这里就是简单的字符串映射）

```
ATTR_BASE_NAME	素质
ATTR_CLEAR_TIPS	重置本属性页之后，本属性页上面的加点操作将会被完全取消，返还所有的素质点，需要你重新加点才会生效
ATTR_EQUIP_NAME	装备
...
```



由于运行时所有字符串是存储在字符串池中，所以在publish的时候我们会做一次转换，将上诉的值替换成哈希值，然后再通过哈希值去字符串池查询对应的结果（ManualLocalizationTable.txt）。

```
ATTR_BASE_NAME	1093192
ATTR_CLEAR_TIPS	2475109970
ATTR_EQUIP_NAME	1178220
...
```

这块转换代码在CodeKeyExporter类中实现，这块逻辑比较简单，遍历上诉的文本，将字符串值插入本地化文件，将对应的哈希值存储到文件ManualLocalizationTable.txt中，publish的时候加载这个文件，就能通过代码key获取到字符串哈希，进而从字符串池拿到结果。

#### PrefabKey

Prefab上的文本节点如果需要本地化，那么就要挂上UIStringLocal脚本（目前在导出prefab文本的时候，会自动刷上此组件）

##### 导出

遍历prefab上面所有的文本组件(Text、RichText、TextmeshPro)，提取文本，挂上本地化组件即可

##### 如何本地化

当前prefab上的本地化做的比较粗暴：无论如何都要让此节点初始化一次

```c#
public void MarkRefreshed()
{
    if (!_textHasRefreshed) _textHasRefreshed = true;
}

private void freshTxt()
{
    if (_textHasRefreshed)
    {
        return;
    }
    _textHasRefreshed = true;

    if (LocalKey == 0) return;

    var value = StringPoolManager.singleton.GetString(LocalKey);
    if (!string.IsNullOrEmpty(value))
    {
        if (_tmp) _tmp.text = value;
        else if (_txt) _txt.text = value;

        return;
    }
}
```



在UIStringLocal上有一个字段标记此节点是否被赋过值

1. 如果没有代码赋值，那么在节点OnEnable的时候，会自动按组件上的哈希值，去字符串拿一次结果，从而本地化
2. 代码对节点赋值的时候，会主动对此节点进行标记（这里要求都通过MLuaUICom组件进行操作）

根据以上规则，便能在节点显示出来之前保证被本地化一次，防止代码未赋值暴露原始测试文本或者非翻译文本的bug；这种做法一定程度上加大了文本导出量（原先是需要程序单独标记每一个节点文本是否不导出，但是费时费力，过于繁复）

#### CommandKey

[Command示例](./command.md)

剧本这块由于是我们自己实现的一套DSL，语法示例如下（摘取于CommandScript/Task/1013.txt）

```
tag@Main_1004610_yiwai
talk@false|这……我该说你心思纯良还是……咳，不知你还记不记得，她委托你送来一封信，这信件的内容就是告诉我，她已经调查到我委托她调查的东西了，并约我第二天在密室见面告诉我。
select@false|但她却在那天晚上死去……我想，她的死亡一定和我委托的东西有关系。|你委托她调查什么？|Main_1004610_weituo
quit

tag@Main_1004610_weituo
select@false|我委托她帮我调查城中出现的白面具……而且，你记不记得，随信送到我这里的，还有一个白面具，中途还被弄丢。所以此事和白面具脱不了关系……|露克希得知了白面具的秘密//露克希拿了白面具的东西|Main_1004610_sec,Main_1004610_thing
quit

tag@Main_1004610_sec
select@false|极有可能……是她得知了我委托事件中的真相，唉，这孩子冰雪聪明，我本来不想告诉她那么多，就是担心她的安危。|什么样的真相？|Main_1004610_truth
quit

tag@Main_1004610_thing
select@false|不排除这种可能，但我觉得更可能……是她得知了我委托事件中的真相，唉，这孩子冰雪聪明，我本来不想告诉她那么多，就是担心她的安危。|什么样的真相？|Main_1004610_truth
quit
```

剧本每一行都代表一个指令，并且约定了每个指令的格式，对此，有以下定义

```c#
namespace MoonClient.CommandSystem
{
    [AttributeUsage(AttributeTargets.Class, AllowMultiple = true)]
    public class CommandArgsAttribute : Attribute
    {
        public string ArgName { get; }
        public string ArgDescription { get; }
        public Type ArgType { get; }
        public int ArgIndex { get; }
        public bool Optional { get; }
        public bool NeedTranslate { get; }

        public CommandArgsAttribute(string argName, Type argType, int argIndex, bool optional = false, string argDescription = null, bool needTranslate = false)
        {
            ArgName = argName;
            ArgType = argType;
            ArgIndex = argIndex;
            Optional = optional;
            ArgDescription = argDescription;
            NeedTranslate = needTranslate;
        }
    }
}

namespace MoonClient.CommandSystem
{
    [CommandArgs("是否为玩家说话", typeof(bool), 0)]
    [CommandArgs("说话内容", typeof(string), 1, needTranslate: true)]
    [CommandArgs("音效配置", typeof(int), 2, true)]
    public class TalkCommand : BaseCommand
    {
        public override void HandleCommand()
        {
            base.HandleCommand();
            if (Args.Count > 2)
            {
                MAudioMgr.singleton.PlayCV(int.Parse(Args[2].Value));
            }
            IMLua luaEngine = MInterfaceMgr.singleton.GetInterface<IMLua>(MCommonFunctions.GetHash("MLua"));
            luaEngine.SendMessageToLua("ENUM_COMMAND_NPC_TLAK", this, bool.Parse(Args[0].Value), Args[1].Value);
        }

    }
}
```

CommandArgsAttribute属性定义了改指令的格式，其中我们关注ArgIndex和NeedTranslate即可

##### 导出

根据上面的属性标签可以得知那些参数被标记为需要本地化，将上诉文本导给本地化文件即可，这里利用了剧本原先的解释逻辑，然后获取字符串

```c#
foreach (var commandData in blockData.Commands)
{
    if (!CommandConst.NeedTranslateArgIndexs.TryGetValue(commandData.commandType, out var needTranslate))
    {
        continue;
    }
    
    var count = commandData.args.Count;
    for (var i = 0; i < count; i++)
    {
        if (needTranslate.Contains(i))
        {
            var key = ((CommandBlockStringArg) commandData.args[i]).OriginString;
            var location = $"Command{LocalizationKeyExporter.ParamConnectChar}{fileRelativeLocation}{LocalizationKeyExporter.ParamConnectChar}{commandData.lineNumber}{LocalizationKeyExporter.ParamConnectChar}{i}";
            LocalizationBridge.KeyExporter.InsertKey(key, source, location, ResolveConflict, filePath, commandData.lineNumber, i);
        }
    }
}
```

##### 本地化

依据剧本的规则，在解析文本的时候，将需要本地化的文本翻译即可，代码如下：

```c#
for (var i = 0; i < argOriginStrings.Length; i++)
{
    var origin = argOriginStrings[i];
    bool needTransltate = false;
    if (Application.isPlaying &&
        !string.IsNullOrEmpty(codeId) &&
        !MGameContext.singleton.IsMainLanguage &&
        CommandConst.NeedTranslateArgIndexs.TryGetValue(codeId, out var indexArr))
    {
        needTransltate = indexArr.Contains(i);
    }
    if (!ParseArg(origin, needTransltate, out var arg))
    {
        MDebug.singleton.AddErrorLog($"解析参数失败：{origin}");
        data = CommandData.Default;
        return false;
    }
    commandData.args.Add(arg);
}

private static bool ParseArg(string originArg, bool needTranslate, out BaseArg arg)
{
    if (needTranslate)
    {
        originArg = StringPoolManager.singleton.GetString(originArg);
    }
    arg = new CommandBlockStringArg().Init(originArg);
    return true;
}
```



#### TableKey

表本地化的导出也比较简单，根据json配置(NeedLocal字段)，可以知道那些字段是需要本地化的

```json

{
    "isNeedPre" : false,
    "isNeedPost" : false,
    "TableCodeTarget" : 4294967295,
    "TableBytesTarget" : 4294967295,
    "MainTableName"    : "ItemTable",
    "TableLocations"   : [
        {
            "ExcelPath" : "ItemTable.csv",
            "SheetName" : null
        },
        {
            "ExcelPath" : "ItemTableTask.csv",
            "SheetName" : null
        }
    ],
    "Fields"           : [
        {
            "FieldName" : "ItemID",
            "FieldTypeName" : "int",
            "DefaultValue"  : "",
            "ForClient"     : true,
            "ForServer"     : true,
            "ClientPosID"   : 0,
            "ServerPosID"   : 0,
            "EditorPosID"   : 0,
            "ClientPosTimeStamp" : 20190424154751,
            "ServerPosTimeStamp" : 20190424154751,
            "EditorPosTimeStamp" : 20190424154751,
            "IndexType"          : 1,
            "CheckInfos"         : null,
            "NeedLocal"          : false
        },
        {
            "FieldName" : "ItemName",
            "FieldTypeName" : "string",
            "DefaultValue"  : "",
            "ForClient"     : true,
            "ForServer"     : true,
            "ClientPosID"   : 1,
            "ServerPosID"   : 1,
            "EditorPosID"   : 1,
            "ClientPosTimeStamp" : 20190424154751,
            "ServerPosTimeStamp" : 20190424154751,
            "EditorPosTimeStamp" : 20190424154751,
            "IndexType"          : 0,
            "CheckInfos"         : [
            ],
            "NeedLocal"          : true
        },
...
```

那么根据表格配置，将对应需要本地化的列的字符串提取出来即可。

```c#
private void InsertKeyByTableName(int binCol, int searchCol, string tableName)
{
    var args = new List<object>();
    for (int iRow = 0, sz = _fields.Count; iRow < sz; iRow++)
    {
        var fieldStrs = _fields[iRow];
        for (int iCol = 0, len = fieldStrs.Count; iCol < len; iCol++)
        {
            var curField = _fieldFieldsToExport[iCol];
            if (!curField.NeedLocal) continue;

            var valueStr = fieldStrs[iCol];
            //空白部分使用占位符
            if (string.IsNullOrEmpty(valueStr))
            {
                valueStr = curField.DefaultValue;
            }

            var fieldTypeName = curField.FieldTypeName;
            var parser = ParserUtil.GetParser(fieldTypeName);
            var strLs = parser.GetStringValueFromString(valueStr);
            for (int i = 0; i < strLs.Count; i++)
            {
                if (string.IsNullOrEmpty(strLs[i])) continue;

                var column = -1;
                if (fieldIndexs != null && iCol < fieldIndexs.Length)
                {
                    column = fieldIndexs[iCol];
                }

                args.Clear();
                args.Add(column);
                args.Add(iRow);
                args.Add(tableName);
                string location = string.Empty;
                if (binCol >= 0) location = $"Table{LocalizationKeyExporter.ParamConnectChar}{tableName} id[{fieldStrs[binCol]}]row[{iRow}]col[{column}]";
                else if (searchCol >= 0) location = $"Table{LocalizationKeyExporter.ParamConnectChar}{tableName} search[{fieldStrs[searchCol]}]row[{iRow}]col[{column}]";
                LocalizationBridge.KeyExporter.InsertKey(strLs[i], ELocalizationKeyCommentSource.Table, location, ResolveConflict, args.ToArray());
            }
        }
    }
}
```

##### 本地化

csv转为bytes的时候，bytes中存储的是对应字符串的哈希值，生成的读表代码实际上是从bytes中拿到uint32的一个哈希值然后进行本地化

```c#
public string ItemName
{
    get
    {
#if DEBUG
        if (singleton.Size <= _rowNum)
        {
            MDebug.singleton.AddErrorLogF("ItemName out of index, rowNum={0}", _rowNum);
        }
#endif
        if(ItemNamePointer == null)
        {
            uint key = RowDataProxy.GetValueUInt(RowDataPtr, 1);
            ItemNamePointer = StringPoolManager.singleton.GetString(key);
        }
        return ItemNamePointer;
    }
}
```

### 哈希冲突

以上文本导出都存在一种情况，两个不同的字符串可能算出来的哈希值是一样的（我们使用的是快速哈希算法，最终得到一个uint32的数）

##### 解决办法

比如存在“血滴”和“衣橱”两个串：计算出来的哈希值都是1179444，将后存储的字符串加上一些冲突描述字段，比如衣橱改为“@衣橱##@”，此时哈希值为1940710554，然后将该字符串写回原先存在的位置，即冲突解决。

以下代码是收集本地化字符串的核心代码。

```c#
/// <summary>
/// 新增字符串(也可以是替换翻译)
/// </summary>
/// <param name="key"></param>
/// <param name="value"></param>
/// <param name="fakeInsert">不实际增加，用于hash冲突判定</param>
/// <returns></returns>
protected string InternalInsertNew(string key, string value = "", bool fakeInsert = false)
{
    if (!CollectKey && string.IsNullOrEmpty(value)) return key;

    key = key.GetOneLineString();
    var hash = MCommonFunctions.GetHash(key);
    var originalKey = key;
    // 获取原始key
    if (MCommonFunctions.HasConflictFlag(originalKey))
    {
        originalKey = MCommonFunctions.GetConflictOriginKey(originalKey);
    }
    
    // 通过原始key判断是否已经有记录
    if (ConflictPart.TryGetValue(originalKey, out var conflictHash) && KeyDict.TryGetValue(conflictHash, out var item))
    {
        hash = conflictHash;
        key = item.Key;
    }

    // 获取已经存储的串，如果发现已存在，需要判定两者是否一致
    if (KeyDict.TryGetValue(hash, out var cacheValue))
    {
        // 和当前已存储的串不一致，说明冲突
        if (key != cacheValue.Key)
        {
            // 未记录
            var newStr = MCommonFunctions.ConvertConflictKey(generateConflictDicCache(), key);
            var newHash = MCommonFunctions.GetHash(newStr);
            if (!fakeInsert)
            {
                ConflictPart.Add(originalKey, newHash);
            }
            key = newStr;
            hash = newHash;
        }
    }

    if (fakeInsert) return key;

    if (!KeyDict.TryGetValue(hash, out item))
    {
        item = new LocalizationTextStringFileItem {Hash = hash, Key = key, Value = value};
        KeyDict.Add(hash, item);
    }
    else
    {
        item.Value = value;
    }

    // 标记新的冲突串
    if (MCommonFunctions.HasConflictFlag(key) && !ConflictPart.ContainsKey(originalKey))
    {
        ConflictPart.Add(originalKey, hash);
    }

    Set.Add(key);

    return key;
}
```



## 资源

#### 资源目录介绍

最初设想的是以主分支（国内），其余海外目录是为对国内目录的修改

基于项目结构，对artres、clientproj、config新增了子目录

1. clientproj/Assets/overseas_clientproj. 主要是对打包相关资源（icon、launch...）
2. clientproj/Assets/artres/overseas_artres 主要是资源（font、texture、prefab、atlas...）
3. config/overseas_config 增量配置（csv、翻译）

```
overseas_clientproj
├── Japan
│   ├── Editor
│   ├── Editor.meta
│   ├── Icon
│   │   ├── common
│   │   │   ├── default_icon
│   │   │   │   ├── Icon-192.png
│   │   │   │   └── Icon-192.png.meta
│   │   │   ├── default_icon.meta
│   │   │   ├── mipmap-hdpi
│   │   │   │   ├── ic_launcher_background.png
│   │   │   │   ├── ic_launcher_background.png.meta
│   │   │   │   ├── ic_launcher_foreground.png
│   │   │   │   └── ic_launcher_foreground.png.meta
│   │   │   │   │   ...
│   │   └── common.meta
│   ├── Icon.meta
│   ├── Plugins
│   │   ├── Android
│   │   │   ├── res
│   │   │   │   ├── raw
│   │   │   │   │   ├── launch.mp4
│   │   │   │   │   └── launch.mp4.meta
│   │   │   │   └── raw.meta
│   │   │   └── res.meta
│   │   └── Android.meta
│   ├── Plugins.meta
│   ├── Resources
│   │   ├── Launch
│   │   │   ├── Logo.png
│   │   │   └── Logo.png.meta
│   │   ├── Launch.meta
│   │   ├── MiniStringPool
│   │   │   ├── JAPANESE_MINI_STRING_POOL.json
│   │   │   └── JAPANESE_MINI_STRING_POOL.json.meta
│   │   ├── MiniStringPool.meta
│   │   ├── SDKConfig
│   │   │   ├── ObbDownloader.json
│   │   │   └── ObbDownloader.json.meta
│   │   └── SDKConfig.meta
│   ├── Resources.meta
│   ├── Scripts
│   │   ├── Lua
│   │   │   ├── UI
│   │   │   │   ├── Ctrl
│   │   │   │   ├── Ctrl.meta
│   │   │   │   ├── Panel
│   │   │   │   └── Panel.meta
│   │   │   └── UI.meta
│   │   └── Lua.meta
│   ├── Scripts.meta
│   ├── StreamingAssets
│   │   ├── Movie
│   │   │   ├── launch.mp4
│   │   │   └── launch.mp4.meta
│   │   └── Movie.meta
│   └── StreamingAssets.meta
├── Japan.meta
├── Korea
│   ├── Editor
│   │   ├── XUPorter
│   │   │   ├── Mods
│   │   │   │   ├── RO.projmods
│   │   │   │   └── RO.projmods.meta
│   │   │   └── Mods.meta
│   │   └── XUPorter.meta
│   ├── Editor.meta
│   ├── Icon
│   │   ├── com.gravity.ragnarokorigin.one
│   │   │   ├── default_icon
│   │   │   │   ├── Icon-192.png
│   │   │   │   └── Icon-192.png.meta
│   │   │   ├── default_icon.meta
│   │   │   ├── mipmap-hdpi
│   │   │   │   ├── ic_launcher_background.png
│   │   │   │   ├── ic_launcher_background.png.meta
│   │   │   │   ├── ic_launcher_foreground.png
│   │   │   │   └── ic_launcher_foreground.png.meta
│   │   │   │   │   ...
│   │   ├── common
│   │   │   ├── default_icon
│   │   │   │   ├── Icon-192.png
│   │   │   │   └── Icon-192.png.meta
│   │   │   ├── default_icon.meta
│   │   │   ├── mipmap-hdpi
│   │   │   │   ├── ic_launcher_background.png
│   │   │   │   ├── ic_launcher_background.png.meta
│   │   │   │   ├── ic_launcher_foreground.png
│   │   │   │   └── ic_launcher_foreground.png.meta
│   │   │   ...
│   │   └── common.meta
│   ├── Icon.meta
│   ├── Plugins
│   │   ├── Android
│   │   │   ├── res
│   │   │   │   ├── raw
│   │   │   │   │   ├── launch.mp4
│   │   │   │   │   └── launch.mp4.meta
│   │   │   │   └── raw.meta
│   │   │   └── res.meta
│   │   └── Android.meta
│   ├── Plugins.meta
│   ├── Resources
│   │   ├── MiniStringPool
│   │   │   ├── KOREAN_MINI_STRING_POOL.json
│   │   │   └── KOREAN_MINI_STRING_POOL.json.meta
│   │   ├── MiniStringPool.meta
│   │   ├── SDKConfig
│   │   │   ├── ObbDownloader.json
│   │   │   └── ObbDownloader.json.meta
│   │   ├── SDKConfig.meta
│   │   ├── __Android
│   │   │   ├── Launch
│   │   │   │   ├── Logo.png
│   │   │   │   └── Logo.png.meta
│   │   │   └── Launch.meta
│   │   ├── __Android.meta
│   │   ├── __IOS
│   │   │   ├── Launch
│   │   │   │   ├── Logo.png
│   │   │   │   └── Logo.png.meta
│   │   │   └── Launch.meta
│   │   └── __IOS.meta
│   ├── Resources.meta
│   ├── Scripts
│   │   ├── Lua
│   │   │   ├── UI
│   │   │   │   ├── Ctrl
│   │   │   │   ├── Ctrl.meta
│   │   │   │   ├── Panel
│   │   │   │   └── Panel.meta
│   │   │   └── UI.meta
│   │   └── Lua.meta
│   ├── Scripts.meta
│   ├── StreamingAssets
│   │   ├── Movie
│   │   │   ├── 1.3.mp4
│   │   │   ├── BattleVideo.mp4
│   │   │   │   ...
│   │   │   └── launch.mp4
│   │   └── Movie.meta
│   └── StreamingAssets.meta
...
└── newchannel.rar

79 directories, 187 files

```

```
overseas_artres
├── Japan -- 日本地区
│   └── Resources
│       └── UI
│           └── Font
│               └── Common.ttf
├── Korea -- 韩国地区
│   ├── Korea.txt -- 编辑器下用海外资源标记
│   ├── Resources
│   │   └── UI
│   │       ├── Atlas -- 海外图集
│   │       │   └── FontSprite.png
│   │       ├── Font -- 海外字体
│   │       │   └── Common.ttf
│   │       ├── Prefabs -- 海外prefab
│   │       │   └── Login.prefab
│   │       ├── TMP -- 海外tmp
│   │       │   ├── Common\ SDF.asset
│   │       │   ├── Common_Achievement_Number.mat
│   │       │   │   ...
│   │       │   ├── Common_Yellowdge.mat
│   │       │   └── Korean.txt
│   │       └── Texture -- 海外贴图
│   │           ├── HowToPlay
│   │           │   ├── ui_Howtoplay_Dieying02.png
│   │           │   ├── ui_Howtoplay_Gonghuishoulie02.png
│   │           │   │   ...
│   │           │   └── ui_Howtoplay_huntplay2.png
│   │           ├── Others
│   │           │   ├── RO_Age.png
│   │           │   │   ...
│   │           │   └── UI_RoleInfo_Img1.jpg
│   │           ├── PlotIcon
│   │           │   ├── UI_PlotIcon_Image_27.png
│   │           │   │   ...
│   │           │   └── UI_mxts_Image_02.png
│   │           └── PrefabBg
│   │               ├── RO_Loading_Jinglianwu_L.png
│   │               │   ...
│   │               └── RO_Loading_Yinglingdian_R.png
│   ├── _FMod -- 海外fmod
│   │   └── Mobile
│   │       ├── AMB.bank
│   │       │   ...
│   │       └── VO_Korea.bank
│   ├── _GameRes -- 海外资源
│   │   └── Effects
│   │       └── Textures
│   │           └── SourceReplace -- 此目录资源会自动替换原始目录资源
│   │               └── Ui
│   │                   ├── Fx_CS_LoGo_BiYou_01.png
│   │                   │   ...
│   │                   └── c_ui_sb.png
│   └── _UI -- 海外图集原始贴图资源
│       ├── FontSprite
│       │   ├── UI_Activity_Mvp_Identification_Lucky.png
│       │   │   ...
│       │   └── ui_Tower_Ward05.png
│       └   ...
...
├── newchannel.zip
└── newchannel.zip.meta

```

```
overseas_config
├── Korea -- 地区目录
│   ├── Assets
│   │   ├── Editor
│   │   │   └── Table -- 编辑器使用海外bytes
│   │   │       ├── AccessoryTable.bytes
│   │   │       ├── AchievementBadgeTable.bytes
...
│   │   │       └── WorldEventTable.bytes
│   │   └── Resources
│   │       ├── Localization -- 本地化目录
│   │       │   ├── Export -- 导出目录
│   │       │   │   ├── COMMANDBLOCK_TREE_RECORD.xlsx
│   │       │   │   ├── Localization.xlsx
│   │       │   │   └── language.xlsx
│   │       │   ├── Import -- 导入目录
│   │       │   │   ├── COMMANDBLOCK_TREE_RECORD.xlsx
│   │       │   │   ├── Localization.xlsx
│   │       │   │   └── language.xlsx
│   │       │   ├── Korean -- old目录，ep2分支合并过来的
│   │       │   │   ├── EDITOR_STRING_POOL_CACHE.kvp
│   │       │   │   └── EDITOR_STRING_POOL_CONFLICT_CACHE.kvp
│   │       │   ├── Korean.bytes -- 韩语字符串池
│   │       │   ├── Korean.loc -- 韩语本地化文件
│   │       │   ├── Korean_ex.bytes -- 韩语字符串池补充
│   │       │   ├── _Temp -- 临时目录，用于方便策划git看差异
│   │       │   │   ├── COMMANDSCRIPT_NPC.kvp
│   │       │   │   ├── COMMANDSCRIPT_STORYTALK.kvp
│   │       │   │   ├── COMMANDSCRIPT_TASK.kvp
│   │       │   │   ├── MANUAL_KEY.kvp
│   │       │   │   ├── TABLE_KEY.kvp
│   │       │   │   └── UI_KEY.kvp
│   │       │   └── ko.json -- 后台使用json
│   │       └── Table -- 客户端海外bytes目录
│   │           ├── AccessoryTable.bytes
│   │           ├── AchievementBadgeTable.bytes
...
│   │           └── WorldEventTable.bytes
│   └── Table -- 增量表目录
│       ├── CSV -- 增量csv
│       │   ├── AwardPackTable.csv
│       │   ├── DailyActivitiesTable.csv
...
│       │   ├── WorldEventTable.csv
│       │   └── achievement_target.csv
│       ├── Excel -- 废弃
│       └── ServerBytes -- 服务器海外使用bytes目录
│           ├── AccessoryTable.bytes
│           ├── AchievementBadgeTable.bytes
...
│           └── WorldEventTable.bytes
├── NorthAmerica 北美地区。
...
```

#### 资源自动替换<font color="#ff0000">非常重要</font>

由于我们需要做海外版本或者多语言版本，那么针对各个地区或者各个语言会有资源上的差异，比如字体、美术字等等，正常流程应该是美术同学针对不同地区做不同的Prefab变体，通过加载不同的变体实现，但美术字使用比较多，如果都交给美术同学维护的话，比较繁琐也容易漏，我们使用了资源自动替换的方案，即保持原来资源的引用条件下，资源内容修改。

查看unity资源的meta文件可以得知第二行为文件的GUID

```
fileFormatVersion: 2
guid: 7e6642c6f9a0aec4285e7766a7a63a71
PrefabImporter:
  externalObjects: {}
  userData: 
  assetBundleName: 
  assetBundleVariant: 

```

那么如果我们将原资源文件替换，新资源的meta采用老资源的GUID，那么资源替换但资源引用也不会丢失

比如

1. clientproj/Assets/artres/Resources/UI/Atlas（简称目录1）存在FontSprite.png和FontSprite.png.meta
2. clientproj/Assets/artres/overseas_artres/Korea/Resources/UI/Atlas（简称目录2）存在FontSprite.png和FontSprite.png.meta
3. 将目录2的FontSprite.png直接替换目录1的FontSprite.png
4. 读取目录1的meta信息里面的GUID
5. 将目录2的meta文件覆盖目录1的meta文件并将GUID改为步骤4的到的结果

##### 备注

像Atlas的meta文件实际是存有sprite切割信息的，所以才会有上面替换meta的过程，如果对meta文件不敏感，其实字需要替换资源文件即可

#### 资源管理

由于存在一个地区多个资源的情况，分为以下几种处理情形，资源存在&处理情形不太一样

1. **中文**

   无需关注，默认即位中文资源

2. **韩文**

   资源存储：同名文件

   打包：将海外目录下的资源，通过前面的资源替换方案，直接替换国内资源，这样包里只会有一份资源

3. **<font color="#ff0000">中文</font>+日文**

   资源存储：同名文件

   打包：将海外目录资源文件名加上后缀"\_\_jp"（比如"FontSprite\_\_jp.png"）,然后拷贝到国内对应路径，包里有些资源会存在一份中文、一份日文资源，根据选择语言不通，加载不同的资源

4. **中文+<font color="#ff0000">日文</font>**

   和3一致

5. **<font color="#ff0000">英语</font>+葡萄牙语**

   资源存储：英文资源和国内资源同名，葡萄牙语资源在文件名后加后缀\_\_pr（比如FontSprite\_\_pr.png）

   打包：将英文资源替换国内资源，并且将葡萄牙语资源直接拷贝到国内目录

备注：

1. 海外资源的存储路径和国内相对一致，比如clientproj/Assets/artres/overseas_artres/Korea/Resources/UI/Atlas和clientproj/Assets/artres/Resources/UI/Atlas
2. 各个地区均以国内目录加上对应的海外目录作为基准目录
3. 各个地区目录的主语言资源均不带后缀，地区目录下的其他资源需要带后缀

而oversea_config就比前面目录简单很多，每个地区下包含该地区支持的所有语言文件

#### 翻译

我们前面提取出了所有的文本，将其存储到一个本地化文件中（LocalizationTextStringFile.cs）

```
2031659194	修复城中受损的拒马，为攻城战做好准备！	
2589418660	战前的武器准备	
1803329920	开采矿石获得了足够的材料，快回去交给士兵吧	
1308251679	在城中寻找矿点，开采到足够的矿石	
...
```

将其输出到本地化翻译文件中

<img src="https://raw.githubusercontent.com/haiyaojing/document/master/uPic/image-20201127152043952.png" alt="image-20201127152043952" style="zoom:33%;" />

翻译公司翻译此文件后，将其导入本地化翻译文件（LocalizationTranslateTextStringFile.cs）

```
1045497368	选择祝福对象	축복 대상 선택
1353620289	选择气氛	분위기 선택
4206811132	神秘的双人礼	신비한 2인용 선물
1516085321	迷人的香氛	매혹적인 향기
1026246441	洒下祝福	축복 뿌리기
1618949819	女孩的甜心礼	달콤한 선물
1090367452	玩家名称	test
931097689	抢到奖励	보상 빼앗기
3141333981	所有礼包价值	모든 선물 가치
...
```



#### 资源加载

前面已经根据地区或者语言区分了资源和文本，那么只需要在加载时做一点手脚即可（资源打包后都以hash为文件名，因此存在一个函数转换地址到哈希）

```C#
/// <summary>
/// 获取资源位置的Hash，如果有海外资源则先检查海外Hash，实现调用方不用关注是否存在海外资源的加载
/// </summary>
/// <param name="location"></param>
/// <param name="suffix"></param>
/// <returns></returns>
public uint GetLocationHash(ref string location, string suffix)
{
    // 原始hash
    uint hash = Hash(location, suffix);
    // 如果存在中文，则直接使用原始路径
    if (_gameLanguage == MGameLanguage.Chinese) return hash;
    // 如果当前是主语言并且不是中文，则也是用原始路径(这种情况主语言的资源会替换原始路径)
    if (_isMainLanguage && !_hasChinese) return hash;
    // 获取配置情况
    if (!MResMgr.Language2Extends.TryGetValue(_gameLanguage, out var ex)) return hash;

    var overseaLocation = $"{location}{ex}";
    var overseaHash = Hash(overseaLocation, suffix);

    // 非发布模式&编辑器下会尝试根据本地生成映射判定是否存在海外资源
    if (!MGameContext.singleton.IsPublish)
    {
#if UNITY_EDITOR
        if (_overseaAssetSet.Contains($"{overseaLocation}{suffix}"))
        {
            location = overseaLocation;
            return overseaHash;
        }
#endif
    }
    // 听过ab管理器判定是否存在海外资源
    if (ABManager.Instance.BundleInfoExist(overseaHash))
    {
        location = overseaLocation;
        return overseaHash;
    }

    return hash;
}
```

**上述函数存在优化内容：每次都会去判断是否存在海外资源，可提前在打包时做好映射，在运行时可省去额外一次拼接字符串计算哈希的消耗**

对于文本，因为我们整合到了一起，加载不同的字符串池即可（使用者都是通过哈希映射对应的文本）。

### 翻译流程



### 多语言切换流程

由于我们选择在游戏内切换语言，我们需要处理文本刷新和资源刷新，重载字符串池，清理当前的缓存，走一次内切场景是比较简单的做法

```c#
/// <summary>
/// 清理缓存、重载字符串
/// </summary>
/// <returns></returns>
private IEnumerator clearAll()
{
    MDebug.singleton.AddLog($"[ChannelManager] reload string");
    StringPoolManager.singleton.Reload();
    MDebug.singleton.AddLog($"[ChannelManager] reload c# table");
    MCppTableLoader.singleton.OnLanguageSwitch();
    MDebug.singleton.AddLog($"[ChannelManager] clear command cache");
    CommandBlockManager.singleton.ClearCache();
    MDebug.singleton.AddLog($"[ChannelManager] reload resloader");
    MResLoader.singleton.OnLanguageSwitch();
    MDebug.singleton.AddLog($"[ChannelManager] mark local component");
    Localization.IncreaseVersion();
    MDebug.singleton.AddLog($"[ChannelManager] global event");
    MEventMgr.singleton.FireGlobalEvent(MEventType.MGlobalEvent_Language_Switched);
    MDebug.singleton.AddLog($"[ChannelManager] clear all lua ui");
    MGame.singleton.LuaEngine.CallTableFunc("game", "OnSwitchLanguageBegin");
    MDebug.singleton.AddLog($"[ChannelManager] clear c# ui");
    MUIManager.singleton.DeActiveAll();
    MDebug.singleton.AddLog($"[ChannelManager] clear all fx");
    MFxMgr.singleton.ClearAll(true);
    MDebug.singleton.AddLog($"[ChannelManager] clear pool go");
    MResGoPool.singleton.ClearAllObjPools();
    yield return new WaitForSeconds(0.3f);
    MDebug.singleton.AddLog($"[ChannelManager] re preload");
    MPreLoadResources.singleton.Reload();
    MDebug.singleton.AddLog($"[ChannelManager] run tasks");
    var it = MPreloader.singleton.RunTasks();
    while (it.MoveNext()) yield return null;
    MDebug.singleton.AddLog($"[ChannelManager] notify lua preload");
    MGame.singleton.LuaEngine.CallTableFunc("game", "OnSwitchLanguageFinish");
    MDebug.singleton.AddLog($"[ChannelManager] switch scene for clear all");
    MStageMgr.singleton.SwitchTo(MStageMgr.singleton.CurrentStage.Stage, MScene.singleton.SceneID);
}
```

同样的，由于c#代码缓存，lua缓存，增加了生命周期

```c#
// AchievementBadgeTable 根据json自动生成
public void OnLanguageSwitch()
{
    AtlasPointer = null;
    IconPointer = null;
    NamePointer = null;
}
```

```lua
-- mgrmgr 这个函数可在对应管理器中重载以清理自身的lua文本缓存
function MgrMgr:OnSwitchLanguage()

    for mgrName, mgr in pairs(l_mgrTable) do
        if mgr.ReloadStringCache ~= nil then
            PCallAndDebug(mgr.ReloadStringCache)
        end
    end
end
```



[切换视频](https://github.com/haiyaojing/document/blob/master/video/switchlanguage.mp4)

### 翻译流程

翻译工具在“ROTools/Localization/打开翻译流程工具”。  参看LocalizationBridge.cs的代码即可

1. 导出key

![img](https://raw.githubusercontent.com/haiyaojing/document/master/uPic/exportkey.png)

2. 导出翻译

![img](https://raw.githubusercontent.com/haiyaojing/document/master/uPic/exporttranslate.png)

3. 导入翻译

![img](https://raw.githubusercontent.com/haiyaojing/document/master/uPic/importtranslate-20201127195822318.png)

4. 合并资源

![img](https://raw.githubusercontent.com/haiyaojing/document/master/uPic/peparepublish.png)

### 编辑器预览

选择地区后，点击工具“ROTools/Localization/编辑器预览海外资源”，会同步美术资源，就可以在游戏中预览海外资源了。记得使用完还愿git工程



## 操作文档

#### 配置(csv)

海外目录csv路径(以韩国为例):

config\overseas_config\Korea\Table\CSV\
我们海外的csv配置是以增量表的形式出现(即海外目录存在对应表，就会对内容进行增量修改)，比如海外目录存在表ActionTable.csv，那么海外目录表ActionTable.csv的内容会 <font color="#ff0000">按ID覆盖</font> 国内ActionTable.csv里面的内容
约束：

1. <font color="#ff0000">海外目录的csv不能增删ID，只能对国内config库的内容进行修改(即如果海外需增加ID，必须现在国内config增加ID，然后海外目录即可拥有对应ID的配置)</font>
2. <font color="#ff0000">海外目录的csv不能修改需本地化的文本，会导致哈希值对不上(要改文本，直接改翻译就好了)</font>
3. <font color="#ff0000">海外目录的csv字段格式，顺序必须与原表相同(若国内表增加、删除字段，需手动同步对应字段到对应的海外目录csv里)</font>
4. <font color="#ff0000">海外目录的csv相对目录需一致，比如config\Table\CSV\TaskTable\TaskAcceptTable.csv，那么对应海外目录的csv目录为config\overseas_config\Korea\Table\CSV\TaskTable\TaskAcceptTable.csv</font>

特例：
1、 有些表可以不走增量表的形式，直接打对应海外目录的csv表的bytes

备注：

1. 海外目录的csv一定是存的修改项，无修改的内容不要复制过去~
2. 母子表也只在海外目录存放需要修改的对应csv过去，不需要把母表也拷贝过去

备注：

1. 配置文件位于config/Table/Tools/BatTools/CSVExporterConfig.json可以配置打表地区、全量表、不检查主键表

示例:
国内表:config\Table\CSV\GlobalTableSystem.csv 

| Category           | Name                    | Comment                                                      | Value |
| ------------------ | ----------------------- | ------------------------------------------------------------ | ----- |
|                    | RedPacketTextMaxLen     | 红包寄语字数                                                 | 30    |
|                    | RedPacketPasswordMaxLen | 红包口令字数                                                 | 30    |
| 货币兑换分地区相关 | ExchangeDiamondMail     | 第一个参数控制直接兑换，第二个控制补差额兑换货币兑换是否走邮件（0为不走邮件直接发送给玩家，1为走邮件） | 1=0   |
|                    | ...                     |                                                              |       |
|                    | ...                     |                                                              |       |



海外表:config\overseas_config\Korea\Table\CSV\GlobalTableSystem.csv

| Category           | Name                    | Comment                                                      | Value |
| ------------------ | ----------------------- | ------------------------------------------------------------ | ----- |
|                    | RedPacketTextMaxLen     | 红包寄语字数                                                 | 1     |
|                    | RedPacketPasswordMaxLen | 红包口令字数                                                 | 1     |
| 货币兑换分地区相关 | ExchangeDiamondMail     | 第一个参数控制直接兑换，第二个控制补差额兑换货币兑换是否走邮件（0为不走邮件直接发送给玩家，1为走邮件） | 2=0   |



那么最终生成bytes里GlobalTable的以下项内容为

| RedPacketTextMaxLen     | 1    |
| ----------------------- | ---- |
| RedPacketPasswordMaxLen | 1    |
| ExchangeDiamondMail     | 2=0  |

#### 打表

打表有两种方式

1. BatTools

   原config\Table\Tools\BatTools目录的批处理仍然可用，根据配置文件(CSVExporterConfig.json)，打多个地区的bytes

2. Unity里的TableTools
   对应面板上都有可以选择地区的下拉框，然后生成bytes即可
   如图:
   
3. 张叔叔开发的csv配置工具
   config\Tools\CsvGen\CsvGen.exe 

### 查看bytes

1. 打开Unity，选择TableDump
   可以详细查看bytes内容，该工具已经优化过了，支持翻页，支持显示本地化内容，支持显示嵌套数据结构
2. Clientproj/Tools/TableDumper 脱离于unity的工具

### 触发器（200812补充）

触发器里的中文（比如交互按钮上的文字）， 需要将中文内容放到ManualLocalization_Editor.txt里，然后在触发器里填入对应的key ，否则无法走本地化流程。

### 美术资源

如果需要对资源做本地化，在海外美术目录相同路径放同名文件即可

比如：

1. clientproj/Assets/artres/Resources/UI/Atlas（简称目录1）存在FontSprite.png和FontSprite.png.meta
2. clientproj/Assets/artres/overseas_artres/Korea/Resources/UI/Atlas（简称目录2）存在FontSprite.png和FontSprite.png.meta

#### 图集

由于图集需要保留引用，这里写下海外图集的制作方法

为了方便替换图集，海外库有一个目录存放图集资源，在打包时会自动替换国内的图集资源，比如国内多个prefab引用了一个美术字图集，但不想每个prefab做一个变体引用新的美术字图集比如目录为clientproj\Assets\artres\overseas_artres\Korea\Resources\UI\Atlas\SourceReplace此目录的图集最终会替换clientproj\Assets\artres\Resources\UI\Atlas里面的资源( meta信息会同步动态的修改，所以这个图集是需要通过Atlas工具生成的，而不是直接替换图)
<font color="#FF0000">制作步骤( 非常重要 )</font>:（以制作FontAtlas为例）

1. 上传新图到对应的海外目录，比如\clientproj\Assets\artres\Korea_UI\FontSprite
2. .将FontSprite目录所有的图拷贝至clientproj\Assets\artres_UI\FontSprite( 只拷贝图，不要拷贝其他meta等文件 )
3. 打开Unity，使用Unity的AtlasMaker，选中clientproj\Assets\artres_UI\FontSprite，并生成图集
4. 将第三步生成的图集(clientproj\Assets\artres\Resources\UI\Atlas\FontSprite.png和clientproj\Assets\artres\Resources\UI\Atlas\FontSprite.png.meta)拷贝回clientproj\Assets\artres\overseas_artres\Korea\Resources\UI\Atlas\SourceReplace目录
5. 上传海外目录overseas_artres下的内容即可

#### Prefab

一般prefab不做区分。