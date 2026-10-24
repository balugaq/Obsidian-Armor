# AGENTS.md — Obsidian-Armor

面向 AI 代理（与人类协作者）的项目指南。改动代码/文档/配置前先读完本文件。

## 这是什么

`Obsidian-Armor` 是一个 **Slimefun4 的 Minecraft 服务器插件扩展（addon）**，运行在 Spigot/Paper 1.18 服务端上。
它新增：

- 材料 **黑曜石合金（OBSIDIAN_ALLOY）**：以黑曜石为中心、八颗钻石环绕，在「强化合成台（Enhanced Crafting Table）」中合成。
- 全套 **黑曜石合金装甲**：头盔、胸甲、护腿、靴子，由合金合成，附魔火焰/爆炸/环境防护（等级 6）与耐久（等级 3），防御力远超钻石甲。

作者：Jordan Goren（GitHub: JordanG8）。许可证：MIT。

## 技术栈与构建

- 语言：Java 16（`maven.compiler.source/target = 16`）
- 构建：Maven
- 依赖（均为 `provided`，运行时由服务端提供）：
  - `org.spigotmc:spigot-api:1.18.1-R0.1-SNAPSHOT`
  - `com.github.SlimefunGuguProject:Slimefun4:2024.3`（已排除 cscorelib2）
- 插件依赖（运行时）：`Slimefun`（见 `plugin.yml` 的 `depend`）

构建命令（在项目根目录执行）：

```bash
mvn clean package
```

产物：`target/Obsidian-Armor v1.0.0.jar`。将其放入服务端的 `plugins/` 目录（需同时安装 Slimefun4）即可加载。

## 目录结构

```
Obsidian-Armor/
├── plugin.yml                       # 根目录副本（api-version: 1.14）—— 注意：不被打包
├── pom.xml                          # Maven 构建配置
├── src/main/java/me/jordang8/obsidianarmor/
│   └── Main.java                    # 唯一源码：注册 ItemGroup / 合金 / 四件装甲
└── src/main/resources/
    ├── plugin.yml                   # 真正被打包进 jar 的插件描述（api-version: 1.18）
    └── config.yml                   # Slimefun 自动更新开关：options.auto-update: true
```

## ⚠️ 关键坑（务必先看）

1. **两份 `plugin.yml`，只有一份生效。**
   根目录 `plugin.yml`（`api-version: 1.14`）与 `src/main/resources/plugin.yml`（`api-version: 1.18`）内容不同。
   `pom.xml` 的 `<resources>` 只把 `src/main/resources/*` 复制进 jar，因此**真正生效的是 resources 里的 1.18 版本**。
   改插件元数据（名称、依赖、api-version 等）务必改 `src/main/resources/plugin.yml`，否则改动会被忽略。两份长期不同步本身就是隐患，建议删掉根目录那份或改为仅作参考。

2. **附魔分两种调用方式，不要随意统一。**
   `Main.java` 中头盔全部用 `addUnsafeEnchantment`，其余部件用 `addEnchantment(DURABILITY, 3)` + `addUnsafeEnchantment(...)`。
   因为 `PROTECTION_FIRE` / `PROTECTION_EXPLOSIONS` 设为 6，超出原版附魔上限，只能用 unsafe；`DURABILITY 3` 在合法范围内故可用安全接口。改动时别把超出上限的附魔改成 `addEnchantment`，否则会抛异常。

## 代码约定

- 主类 `Main` 实现 `JavaPlugin` 与 `SlimefunAddon`，所有注册都在 `onEnable()` 中完成。`onDisable()` 为空。
- 注册模式固定：`new SlimefunItemStack(id, Material, 显示名, lore)` → 构造 `ItemStack[]` 配方 → `new SlimefunItem(group, stack, RecipeType.ENHANCED_CRAFTING_TABLE, recipe)` → `.register(this)`。
- 显示名/lore 使用中文（带 `&` 颜色码），分类名为「黑曜石合金装甲」。
- `getBugTrackerURL()` 返回 `null`，`getJavaPlugin()` 返回 `this`。新增物品时照此实现 `SlimefunAddon` 接口方法即可。

## 新增物品的标准步骤

1. 在 `src/main/resources/plugin.yml` 确认 `depend` 含 `Slimefun`，无需为单个物品额外声明。
2. 在 `Main.onEnable()` 内按上面的注册模式添加 `SlimefunItemStack` + 配方 + `register(this)`。
3. 如需新增分类，用 `ItemGroup`（含 `NamespacedKey` + 图标 `ItemStack`）。
4. 重新 `mvn clean package` 验证编译通过。

## 代理工作边界（重要）

- **不要**执行 `git commit` / `git push` / `git reset` / `rebase`。提交权归仓库所有者，你只需把工作区改到可通过验收的状态，并给出建议的提交信息。
- 改动后优先用 `mvn clean package` 验证编译；无法在本地起服务端时，至少保证编译无误并说明未做运行时验证。
- 改动 `plugin.yml`、配方、附魔等级前，先回想上方「关键坑」，避免踩重复坑。
