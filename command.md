

## 概况

[DSL](https://blog.csdn.net/hq354974212/article/details/72240447/)的一种实现（作者http://www.resetoter.cn/），以下是对原wiki的拷贝和补充。

### 配置表的变化


 __目前挂载剧情脚本的地方有任务以及NPC的默认对话__  ###任务表

在接受任务以及完成任务的时候也会有相应的剧本数据，填写的是剧本的Id以及剧本中相应的tag

![img](https://raw.githubusercontent.com/haiyaojing/document/master/uPic/tapd_20332331_base64_1521103928_5.png)

![img](https://raw.githubusercontent.com/haiyaojing/document/master/uPic/tapd_20332331_base64_1521103931_77.png)


例如上图跳转到对应剧本的main_1_accept标签

### NPC表


在Npc表中也存在两个字段

![img](https://raw.githubusercontent.com/haiyaojing/document/master/uPic/tapd_20332331_base64_1521103964_12.png)


分别表示的是，NPC没有功能以及任务时出现的默认脚本

## 剧本脚本介绍


语法只有一个


>函数名@参数1|参数2...


例如：


>log@输出信息


策划在配置的时候也可以使用“~”来替代上一次使用的函数


例如

![img](https://raw.githubusercontent.com/haiyaojing/document/master/uPic/tapd_20332331_base64_1521104166_17.png)


第二句调用的函数与上一句相同

或者也可以是通过“*”来表示上一次使用的对应位置的参数

![img](https://raw.githubusercontent.com/haiyaojing/document/master/uPic/tapd_20332331_base64_1521104282_32.png)


与上一句的第一个参数完全相同


通过{{ArgN}}来表示第N个代码块参数（通过代码调用或者脚本调用子脚本传入）


通过{{变量名}}来表示该块或者全局的变量（通过Local与Gloabl命令设置）


//代表注释不会被运行


&lt;&lt;宏名称&gt;&gt;代表可以被替换的宏，例如角色名，角色等级等等


现在宏已经支持表达式，


>类似于&lt;&lt;PlayerLv&gt;30andPlayerLv&lt;60&gt;&gt;


就会返回true或者false,如果解析失败则一律返回false。


LUA&lt;&lt;Lua代码&gt;&gt;表示需要执行的Lua求值代码


>例如：LUA&lt;&lt;1+1&gt;&gt;


下面语句表示判断任务是否完成，如果是则进入tagtest1否则进入tagtest2


>例如：if@LUA&lt;&lt;CommandMacro.CheckTaskFinished(100001)&gt;&gt;|test1|test2


则会返回2，或者Lua可以做的在脚本中都可以直接进行嵌入


通过if命令、switch命令或者condition命令可以做到分支功能


### 触发器系统


目前剧本已经支持触发器系统，


和tag相同，可以在剧本的任意位置添加trigger


例如：


>trigger@test|ON_TAKE_PHOTO|true|testTrigger

其中test为触发器名，ON_TAKE_PHOTO为事件名，第三个参数为条件，可以使用宏或者Lua脚本实现，最后一个就是要跳转的tag，跳转也支持跨剧本文件，要写成剧本名:tag 例如：Trigger/Test:testtrigger


触发器可以由程序埋点触发，也可以由剧本自行触发，调用triggerevent命令。


触发器支持使用剧本动态添加与删除，有addtrigger方法与removetrigger方法。


### 剧本的生命周期


在System/Global.txt中有一个tag叫做oninitglobal


这个剧本在游戏开始的时候就会执行


而在每个剧本读入的时候也会执行一个oninit的tag（如果该tag存在）


### 下面是完整的例子

![img](https://raw.githubusercontent.com/haiyaojing/document/master/uPic/tapd_20332331_base64_1521104359_86.png)

![img](https://raw.githubusercontent.com/haiyaojing/document/master/uPic/tapd_20332331_base64_1521104367_44.png)


Tag表示跳转到的tag，与表中对应


Talk表示与角色进行对话，要求对话框已经打开，第一个参数为是否是玩家，第二个参数为对话内容


StopTalk表示取消对角色的聚焦，并且关闭对话框（如果想继续做别的事情可以不进行调用）


Quit表示退出这段脚本

## 目前已经支持的函数：

|~函数名|描述|用法||
| ---- | ---- | ---- | ---- |
|starttalk|开始对话|starttalk@NpcId||
|talk|对话|talk@是否为玩家(true\false)\|内容\|cvid||
|stoptalk|结束对话|stoptalk||
|select|出现分支|select\|是否为玩家说话\|说话内容\|选项1，选项2，选项3\|跳转tag1，跳转tag2，跳转tag3\|cvid\|取消选项\|true 备注解释：说话角色\|说话内容\|选项\|选项参数\|cvid\|取消选项\|是否使用剧情分支界面||
|showselect|展示选择框（策划自行决定选择框出现时机 不要滥用此命令，需要显示对话框按钮的才能使用，不然会触发无用的界面刷新逻辑 ）|showselect@是否为玩家说话\|说话内容||
|addcancelselect|添加取消按钮|addcancelselect@按钮名称||
|clearselect|清除选择|clearselect||
|log|在Unity中输出日志|log@输出内容||
|wait|等待时间|wait@时间（秒）||
|gototag|跳转到标签|gototag@标签名||
|csevent|发送C#消息|暂未使用||
|luaevent|发送lua消息|luaevent@事件名\|任意数量参数\|...||
|waitluaevent|发送lua消息并等待|waitluaevent@事件名\|任意数量参数\|...||
|runlua|运行lua脚本|暂未使用||
|quit|停止运行脚本|quit||
|randomselect|随机跳转到某个tag|randomselect@tag1,tag2,tag3||
|openui|打开UI|openui@EdenTask||
|showtip|显示Tip|showtip@内容||
|bubble|显示对话气泡|bubble@npcid或者UID\|内容\|播放时间\|小剧场标记(true/false)||
|showaction|显示动作|showaction@npcid或者UID\|动作名\|播放后续剧本的等待时间||
|emotion|显示面部表情|emotion@npcid或者UID\|ShowFaceExpressionTable的Rowid\|播放时间||
|headexpression|头顶表情气泡|headexpression@npcid或者UID\|ShowExpressionTable的Rowid\|播放时间\|小剧场标记(true/false)||
|addtaskbtn|动态添加对话功能按钮|addtaskbtn@按钮名\|require的文件\|函数名\|点击后是否隐藏对话框||
|addfunc|动态添加功能按钮|addfunc@功能Id\|下一个tag（不填就执行下一句）||
|openfunc|执行配置在OpenSystemTable中的功能函数|openfunc@功能Id||
|local|添加块变量|local@变量名\|变量内容||
|global|添加全局变量|global@变量名\|变量内容||
|usenpc|声明剧本使用的npc|usenpc@npcid\|npcid...||
|scenariosetpos|ui层剧本设置显示位置(x坐标,有坐标)|scenariosetpos@100\|100||
|scenariochat|ui层剧本显示对话(文本内容,显示时长)|scenariochat@一条内容\|5||
|scenarioemoji|ui层剧本显示表情(表情ID,显示时长)|scenarioemoji@2\|5||
|scenariofontsize|ui层剧本设置文本字体大小(字号)|scenariofontsize@30||
|showmodelalarm|剧本中播放喊话|showmodelalarm@喊话主体（1：玩家；2：NPC；3：怪）\| &lt; &lt;ID&gt;&gt;（玩家填0，NPC或怪物填对应ID）\|喊话内容\|时间||
|posfx|在某个位置显示特效|posfx@特效id\|x,y,z\|时间||
|rolefx|在某个角色位置显示特效|rolefx@特效id\|角色Id\|时间||
|focus2npc|镜头看npc|focus2npc@镜头id\|npcid\|是否要隐藏玩家true/false||
|focus2player|镜头看玩家|focus2player@镜头id\|是否要隐藏玩家true/false||
|focus2pos|镜头看点|focus2pos@镜头id\|位置Vector|方向float\|是否要隐藏玩家true/false|
|resetcam|重置镜头|resetcam||
|gotonpc|玩家跑向Npc|gotonpc@场景id\|npcid||
|gotopos|玩家跑到某个地点|gotopos@场景id\|x,y,z\|朝向（0~360）||
|multiplestart|并行执行下列命令（动作指令）|multiplestart@并行执行行数||
|npcmove|Npc跑到某个地点（单机）|npcmove@npcid\|目标点Vector3\|速度float\|面朝方向0~360\|范围float||
|npcrotate|NPC旋转方向|npcrotate@npcid\|方向0~360||
|coupleaction|玩家与npc做双人交互动作（包含移动行为）|coupleaction@动作id\|NpcId||
|netaievent|发送网络AI事件|netaievent@事件名||
|netevent|发送网络事件|netevent@npcid\|事件名\|参数1,参数2,参数3||
|findelf|找到波利|findelf@触发的波利id\|NpcId||
|initblock|初始化剧本|initblock@剧本名 例如:Trigger/Test||
|addlocalnpc|添加本地npc|addlocalnpc@场景Id\|NpcId\|位置\|朝向||
|rmlocalnpc|移除本地npc|rmlocalnpc@场景Id\|NpcId||
|camerastate|切换相机状态|camerastate@相机状态（目前有Trigger（自动状态）、Normal（普通状态） 其他状态不建议使用，就先不写了）||
|cameratarget|切换相机目标|cameratarget@Npcid或角色id或位置\|offset(x,y,z)\|距离\|X轴角度\|Y轴角度\|fov\|是否可以旋转||
|cameraspeed|更新相机旋转速度（在Trigger状态下有效）|cameraspeed@距离速度\|X轴旋转速度\|Y轴旋转速度\|Offset更新速度||
|cameratonormal|自动相机模式调整到普通模式的状态|cameratonormal||
|trystoptalk|尝试停止对话，如果没有下一句话则停止|trystoptalk||
|playcv|播放CV|playcv@cvid(AudioStoryTable)||
|stopcv|停止当前CV|stopcv||
|blackcurtain|播黑幕|blackcurtain@黑幕id(BlackCurtainTable)||
|playsound|剧本中调用音效|playsound@音效资源（示例：playsound@event:/UI/SkillPanelAddPoint）||
|playtheater|剧本中调用小剧场镜头组|playtheater@ID （此ID为 TheaterTable 内的小剧场ID）||
|cutscene|剧本中调用CutScene|cutscene@ID （CutSceneTable）||
| randomcv | 剧本中调用CutScene | randomcv@id1,id2,id3...[p1,p2,p3] | id为列表，第二个参数为概率(可选) |
|backtrack|指定切换场景时，未执行完剧本跳转行，用于相机状态清理|backtrack@5|即如果当前剧本未执行到该指令往后五条命令时，强制执行往后第五条指令|







### 触发器相关

| ~函数名       | 描述       | 用法                                                   |                                |
| ------------- | ---------- | ------------------------------------------------------ | ------------------------------ |
| addtrigger    | 添加触发器 | addtrigger@触发器名\|事件名\|条件\|tag名 或 地址:tag名 | 例如(Trigger/Test:tesrtrigger) |
| removetrigger | 删除触发器 | removetrigger@触发器名                                 |                                |
| triggerevent  | 触发触发器 | triggerevent@事件名\|参数                              | 例如 arg1:aaa,arg2:bbb         |

### 分支语句

| ~表达式    | 描述                                          | 用法                                                         |      |
| ---------- | --------------------------------------------- | ------------------------------------------------------------ | ---- |
| if         | 双路条件判断                                  | if@&lt;&lt;条件表达式>>\|如果为真的进路tag\|如果为false的进路tag |      |
| switch     | 通过值判断                                    | switch@&lt;&lt;值表达式&gt;&gt;\|值1,值2,etc...\|tag1,tag2,etc... |      |
| condition  | 多路条件判断,如果不满足所有条件则继续往下执行 | condition@&lt;&lt;条件表达式1&gt;&gt;\|进路1\|<<条件表达式2>>\|进路2\|etc... |      |
| plotbranch | 开启分支选项(不需NPC、对话框)                 | plotbranch@按钮名1,按钮名2,按钮名3\|tag1,tag2,tag3\|emoj1,emoj2,emoj3\|end |      |

### 现在已经支持的宏（不进行维护，最新请见CommandMacro.lua）

| ~宏名                    | 描述                           |
| ------------------------ | ------------------------------ |
| PlayerName               | 玩家名称                       |
| PlayerLv                 | 玩家等级                       |
| PlayerUID                | 玩家UID                        |
| PlayerJobLv              | 玩家职业等级                   |
| NPCName                  | NPC名称                        |
| SceneId                  | 当前场景ID                     |
| NPCId                    | NPCId                          |
| CurrentTemperature       | 当前温度                       |
| FashionGetTheme          | 当前时尚评分的主题，返回数字   |
| FashionGetPoint          | 当前时尚评分的分数，返回数字   |
| FashionGetMaxPoint       | 当前已评最高分，返回数字       |
| FashionGetTheoryMaxPoint | 当前主题的理论最高数，返回数字 |
| FashionGetCount          | 当前评分次数，返回数字         |
| Arg0                     | 传的第一个参数                 |

### 现在已经支持的触发器事件

| ~事件名              | 描述           | 参数                                |
| -------------------- | -------------- | ----------------------------------- |
| ON_START_COLLECT     | 开始采集       | CollectId：采集物Id                 |
| ON_COLLECT_SUCC      | 采集成功       | CollectId：采集物Id                 |
| ON_START_USE_ITEM    | 使用道具       | ItemId:道具Id                       |
| ON_FINISH_USE_ITEM   | 成功使用道具后 | ItemId:道具Id                       |
| ON_DAMAGED_BY_PLAYER | 被玩家攻击时   | EnemyId:受击角色的UID               |
| ON_KILLED_BY_PLAYER  | 被玩家杀害     | EnemyId:被杀害角色的UID             |
| ON_NPC_CREATE        | NPC被创建      | NPCId：创建的NpcId                  |
| ON_NPC_DESTROY       | NPC被销毁      | NPCId：销毁的NPCId                  |
| ON_ENTER_DUNGEONS    | 进入副本       | DungeonsId：副本id，SceneId：场景Id |

### 策划常用Lua函数

||~方法名||描述||使用方法（LUA &lt; &lt;xxxxx&gt;&gt;）||
||CheckTaskFinished||检查任务是否完成||CommandMacro.CheckTaskFinished(任务id)||
||CheckSystemOpen||检查系统是否打开||CommandMacro.CheckSystemOpen(opensystem系统id)||

| ~方法名           | 描述             | 使用方法(LUA<<xxxx>>) |
| ----------------- | ---------------- | --------------------- |
| CheckTaskFinished | 检查任务是否完成 |                       |
| CheckSystemOpen   | 检查系统是否打开 |                       |

### 剧本测试工具


ROTools/剧本工具


可以通过剧本工具调试剧本，以及重载剧本

![图片描述](https://raw.githubusercontent.com/haiyaojing/document/master/uPic/tapd_20332331_base64_1553611384_95-20201204103620417.png)


## 程序相关


策划无需关心在CommandScript下的bytes文件，这些是通过打包是进行生成的二进制文件。


对于程序而言，需要知道在编辑器下将直接使用文本进行翻译，而在运行时则是使用pb。

### 代码目录

```
CommandSystem
├── Attribute 属性定义和检查
│   ├── CommandArgsAttribute.cs
│   └── CommandCheckAttribute.cs
├── Checker 剧本的格式检查
│   └── CommandBlockChecker.cs
├── CommandBlock.cs 剧本块
├── CommandBlockManager.cs 剧本管理器
├── CommandBlockTriggerManager.cs 剧本触发器管理器
├── CommandConst.cs 常量
├── Commands 命令
│   ├── BaseCommand.cs 基类，所有命令继承于此
│   ├── LuaCommand.cs lua命令，非CommandConst里定义的命令会尝试以lua命令执行，实际会跑到lua里查找指令
│   ├── NPC npc相关命令
│   │   ├── ChangeEmotionCommand.cs 表情
│   │   ├── ClearSelectCommand.cs 清理对话框选项
│   │   ├── NpcMoveCommand.cs npc移动
│   │   ├── NpcRoatateCommand.cs npc旋转
│   │   ├── SelectCommand.cs 对话框选项数据填充
│   │   ├── ShowBubbleCommand.cs 气泡
│   │   ├── ShowHeadExpressionCommand.cs 头顶表情
│   │   ├── ShowSelectCommand.cs 显示对话框选项，一般和SelectCommand搭配
│   │   ├── StartTalkCommand.cs 开始对话
│   │   ├── StopTalkCommand.cs 停止对话
│   │   ├── TalkCommand.cs 对话
│   │   └── UseNpcCommand.cs 标记使用npc，好像给任务用的
│   ├── Other
│   │   ├── FindElfCommand.cs 查找boli
│   │   └── ShowModelAlarmCommand.cs 显示副本倒计时
│   └── System
│       ├── AddLocalBuffCommand.cs 增加buff（尚未实装，因为不太安全）
│       ├── AddTriggerCommand.cs 增加触发器
│       ├── BreakCommand.cs 中断
│       ├── ConditionCommand.cs 条件指令
│       ├── CustomVarCommand.cs 自定义参数
│       ├── GlobalVarCommand.cs 全局参数
│       ├── GotoTagCommand.cs 跳转
│       ├── IfCommand.cs 二路条件
│       ├── InitBlockCommand.cs 初始化
│       ├── LocalVarCommand.cs 本地变量
│       ├── LogCommand.cs 日志
│       ├── MultipleProcessStartCommand.cs 同时触发多个指令，而不是一个完成才能执行下一个
│       ├── NpcCoupleActionCommand.cs npc双人动作
│       ├── PlotBranchCommand.cs 剧情分支
│       ├── QuitCommand.cs 退出
│       ├── RandomSelectCommand.cs 随机选择
│       ├── RemoveLocalBuffCommand.cs 移除本地buff（尚未实装）
│       ├── RemoveTriggerCommand.cs 移除触发器
│       ├── RunLuaCommand.cs 执行lua
│       ├── ScriptEventCommand.cs 触发事件
│       ├── SendCSEventCommand.cs 向服务器发事件
│       ├── SendLuaEventAndWaitCommand.cs 发lua事件并等待
│       ├── SendLuaEventCommand.cs 发lua事件
│       ├── SwitchCommand.cs 多路条件
│       └── WaitCommand.cs 等待
├── Compile
│   ├── CommandBlockBinaryCompiler.cs 编译器
│   └── CommandBlockParser.cs 解释器
├── Data 参数定义
│   ├── BaseArg.cs  参数基类
│   ├── BlockIndexArg.cs
│   ├── BlockVarArg.cs
│   ├── CommandBlockArg.cs
│   ├── CommandBlockStringArg.cs
│   ├── CommandData.cs
│   ├── CommandLuaArg.cs
│   ├── ExpressionArg.cs
│   ├── FunctionArg.cs
│   └── ValueArg.cs
├── Expression 表达式
│   ├── EOFToken.cs
│   ├── IdentifierToken.cs
│   ├── Lexer.cs
│   ├── LuaConverter.cs
│   ├── NumberToken.cs
│   ├── StringToken.cs
│   └── Token.cs
└── Trigger 触发器
    ├── CommandTrigger.cs
    └── Event
        ├── BaseEvent.cs
        ├── OnCollectSuccEvent.cs
        ├── OnDamagedByPlayerEvent.cs
        ├── OnEnterDungeons.cs
        ├── OnExitDungeons.cs
        ├── OnKilledByPlayerEvent.cs
        ├── OnNpcCreateEvent.cs
        ├── OnNpcDestroyEvent.cs
        └── OnStartCollectEvent.cs
```

```
Command
├── AddCancelTalkCommand.lua
├── AddFlagValCommand.lua
├── AddLocalNpcCommand.lua
├── AddOpenFuncCommand.lua
├── AddTaskBtnCommand.lua
├── BackTrackingCommand.lua
├── BaseCommand.lua
├── CameraSetTriggerSlowSpeedCommand.lua
├── CameraStateCommand.lua
├── CameraTargetCommand.lua
├── CameraToNormalCommand.lua
├── CommandConst.lua
├── CommandMacro.lua
├── FocusToNpcCommand.lua
├── FocusToPlayerCommand.lua
├── FocusToPosCommand.lua
├── GotoNpcCommand.lua
├── GotoPosCommand.lua
├── HideModelAlarmCommand.lua
├── NetAIEventCommand.lua
├── NetEntityAIEventCommand.lua
├── NetEventCommand.lua
├── OpenFuncCommand.lua
├── OpenUICommand.lua
├── PlayBlackCurtainCommand.lua
├── PlayCVCommand.lua
├── PlayCutSceneCommand.lua
├── PlaySoundCommand.lua
├── PlayTheaterCommand.lua
├── PosFxCommand.lua
├── RandomCVCommand.lua
├── RemoveLocalNpcCommand.lua
├── ResetCameraCommand.lua
├── RoleFxCommand.lua
├── ScenarioChatCommand.lua
├── ScenarioEmojiCommand.lua
├── ScenarioSetFontSizeCommand.lua
├── ScenarioSetPosCommand.lua
├── SetInterStateCommand.lua
├── ShowActionCommand.lua
├── ShowTipCommand.lua
├── StopCVCommand.lua
├── StoryboardCommand.lua
└── TryStopTalkCommand.lua

还包括CommandConst.lua、CommandMacro.lua、CommandHandler
```

以下为基类的部分实现

```c#

namespace MoonClient.CommandSystem
{
    public abstract class BaseCommand
    {
				/// <summary>
        /// 初始化参数
        /// </summary>
        public virtual void Init(CommandBlock block, List<BaseArg> args)
        {
            foreach (var arg in args)
            {
                var clone = (arg.Clone() as BaseArg);
                Args.Add(clone);
                if (clone is CommandBlockArg blockArg)
                {
                    blockArg.SetBlock(Block);
                }
            }
        }
				/// <summary>
        /// 处理剧本命令
        /// </summary>
        public virtual void HandleCommand()
        {
        }
				/// <summary>
        /// 处理结束，通知block处理下一行
        /// </summary>
        public virtual void FinishCommand()
        {
            if (!IsInMultipleProcess)
            {
                Block?.TodoNextLine();
            }

            IsInMultipleProcess = false;
        }
}
```

以下为实际的两个简单例子(记得一定要有地方执行FinishCommand，对话框那边选项或者点击才进行下一步即在对应操作的时候执行了FinishCommand)

```c#
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
namespace MoonClient.CommandSystem
{
    [CommandArgs("Log内容", typeof(string), 0)]
    public class LogCommand : BaseCommand
    {
        public override void HandleCommand()
        {
            MDebug.singleton.AddGreenLog($"【剧本Log】{Args[0].Value}");
            FinishCommand();
        }
    }
}
```

### 扩展

扩展也比较简单，继承基类即可，然后在对应的CommandConst中注册一下。

后续扩展一般无本地化需求的直接在lua侧实现即可，目前本地化只支持c#侧。