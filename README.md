# 深渊回响 · Abyss Echo

暗黑克苏鲁题材的 Roguelike 卡牌构建游戏，单文件实现（HTML + CSS + 原生 JS，零依赖、零构建、无外部资源请求）。

直接双击 `index.html` 或拖进浏览器即可运行。

## 玩法闭环

一条局内路线固定为 5 个节点（`PATH`）：事件 → 战斗 → 事件 → 战斗 → Boss，节点顺序在地图界面上按 `showSelect` 渲染。走完节点即通关，HP 或理智归零即失败（`defeat` / `showEnd`）。

- **双资源**：生命（HP）与理智（SANITY）。敌人既造成hp伤害也造成理智伤害，理智偏低会改变可用牌与风险收益；部分卡牌主动消耗理智换输出。
- **战斗**：能量 + 护盾 + 抽牌的回合制，敌人意图先预览再执行（`pickIntent` / `intentText` / `enemyAct`），支持多个敌人围攻（`BATTLE_POOL` 里 5 种敌人随机组合）。
- **事件**：`EVENT_DEFS` 每个事件带若干选项，效果包括改变资源、给牌组加牌、塞入诅咒牌。
- **奖励**：战斗胜利后按稀有度权重抽 3 张候选（`rollWeighted` 内 `普通:5 / 罕见:4 / 史诗:1`，已入组的卡会被排除），选 1 张入组（`showReward` / `chooseReward` / `grantCard`）。候选池 `REWARD_IDS` 共 12 个 key。

## 数据

- 3 个可选角色（`CHAR_DEFS`），各有独占机制与固定 10 张起始牌组：
  - 艾德蒙·格雷「执灯调查员」——理性推演：每回合首张攻击牌每次打击 +3；HP 60 / 理智 40。
  - 塞拉菲娜「低语术士」——禁忌祭献：每回合限一次，弃 1 张手牌换 1 点能量；HP 50 / 理智 55。
  - 卡尔·血斧「锈盾佣兵」——钢铁之躯：每回合开始获得 4 点护盾；HP 70 / 理智 30。
- 45 张卡（`CARD_DEFS`），`rarity` 分布为 普通 21 / 罕见 18 / 史诗 3 / 诅咒 3；字段含费用、类型与效果，由 `playCard` 逐个消费。
- 敌人意图与数值见 `ENEMY_DEFS` / `BATTLE_POOL`。

## 代码结构（都在 `index.html` 里）

| 区域 | 说明 |
| --- | --- |
| 样式段 | 配色与尺寸用 `:root` 里的 CSS 变量定义，眼睛图形为内联 SVG |
| 数据段 | `CARD_DEFS` / `CHAR_DEFS` / `ENEMY_DEFS` / `EVENT_DEFS` / `PATH` / `REWARD_IDS` / `BATTLE_POOL` |
| 状态段 | 全局单一可变对象 `G`（`let G=null`，开局时按所选角色构建：生命/理智上限、能量、当前节点、三堆牌、敌人数组、日志），`rnd` 为唯一的随机入口 |
| 渲染段 | `showScreen` / `showSelect` / `renderBattle` / `showEvent` / `showReward` / `showEnd`，一律 `innerHTML` 字符串拼接整块重绘 |
| 流程段 | `beginTurn` / `drawCards` / `playCard` / `endTurn` / `enemyAct` / `nextNode` / `pickEvent` / `dealEnemyDamage` / `hitPlayer` / `loseSanity` / `gainSanity` / `healHp` / `doSacrifice` / `atkBonus` |
| 日志段 | `addLog` 追加战斗日志并立刻重绘列表 |

## 实现约定

- 无模块化：函数与数据都挂在全局作用域，逻辑、样式、标记同文件。
- 随机只有 `rnd` 与依赖它的 `rollWeighted` / `grantRandomCard`，直接吃 `Math.random()`，没有种子，每局不可复现。
- 数值与渲染耦合在流程函数里（改完状态直接调对应的 `show*` / `render*`）。
- 没有存档、设置面板、音效；文本与 UI 全为中文。
