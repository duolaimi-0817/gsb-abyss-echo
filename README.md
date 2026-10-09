# 深渊回响 · Abyss Echo

暗黑克苏鲁题材的 Roguelike 卡牌构建游戏，单文件实现（HTML + CSS + 原生 JS，零依赖、零构建、无外部资源请求）。

直接双击 `index.html` 或拖进浏览器即可运行。

## 玩法闭环

一条局内路线固定为 5 个节点：事件 → 战斗 → 事件 → 战斗 → 守关（`NODES` / `NODE_LABELS`）。每个节点完成后从奖励池抽卡，选 1 张入组，然后推进到下一节点；守关节点胜利即通关，理智或生命归零即失败。

- **双资源**：生命（HP）与理智（SAN）。理智会被敌人的尖啸、缠绕等意图削减，也可以被卡牌恢复；部分卡牌主动消耗理智换收益。
- **战斗**：能量 + 护盾 + 抽牌的回合制，敌人带预览意图（`INTENTS`，如撕咬/猛扑/尖啸/蜷缩/缠绕），可多点围攻。
- **事件**：`EVENTS` 里每个事件给若干选项，选项改变资源或向牌组注入诅咒疯牌（如蚀心低语），同一局内不重复触发（`S.usedEvents`）。

## 数据

- 3 个可选角色，各有独占机制（`CHARACTERS`）：调查员"洞察"（每回合首张攻击 +3）、疯狂诗人"痛苦灵感"（掉理智抽牌，每回合上限 2 次）、掘墓人"亡者之躯"（回合开始白送 4 点护盾），初始 HP/SAN 各不相同。
- 46 张卡（`CARD_DB`），按 `rarity` 分层：起始 30 张（角色专属牌组 `STARTER_DECKS`，各 10 张）、普通 5 / 稀有 6 / 史诗 4（奖励池 `POOL` 按档位抽取）、诅咒 1（事件或敌人注入）。
- 卡牌效果是扁平字段（`effects: {damage, damageAll, block, draw, energy, sanityGain, selfSanity, selfHp}`），由 `resolveCard` 逐个消费。
- 敌人意图与敌人库见 `INTENTS` / `ENEMY_DB`。

## 代码结构（都在 `index.html` 里）

| 区域 | 说明 |
| --- | --- |
| 样式段 | CSS 变量定义配色（`--bg` / `--green` / `--purple` / `--blood` / `--bone` / `--gold` 等），眼睛图形用内联 SVG（`eyeSvg`）现算 |
| 数据段 | `CARD_DB` / `STARTER_DECKS` / `POOL` / `CHARACTERS` / `INTENTS` / `ENEMY_DB` / `EVENTS` / `NODES` |
| 状态段 | 全局单一可变对象 `S`（`phase` / `charKey` / `nodeIndex` / `usedEvents` / `player` / `battle` / `pendingCard` / `pendingEvent` / `rewardCtx` / `rewardCards`），`P()` 取当前玩家；卡牌实例用 `uidCounter` + `newCard()` 生成 |
| 渲染段 | `renderAll` 分派到 `renderTitle` / `renderHUD` / `renderHand` / `renderEnemies` / `renderLog`，一律用 `innerHTML` 字符串拼接重绘 |
| 流程段 | `startBattle` / `startPlayerTurn` / `onCardClick` / `onEndTurn` / `enemyPhase` / `nextNode` / `startEvent` / `pickEventOption` / `rollRewards` / `pickReward` / `showVictory` / `showGameOver` / `restart` |
| 日志段 | `LOGS` 是环形缓冲（超过 60 条丢弃最旧），`log()` 写入并立刻重绘 |

## 实现约定

- 无模块化：所有函数挂在全局作用域，数据与逻辑同文件。
- 随机只有 `shuffle`（Fisher-Yates）与 `randFrom`，直接吃 `Math.random()`，没有种子，因此每局不可复现。
- 状态更新与渲染耦合在流程函数里（改完 `S` 就直接 `renderAll()`）。
- 没有存档、没有设置面板、没有音效；文本与 UI 全为中文。
