# 每日锻炼推送 — 计划与查询规则

数据源: `~/Downloads/zhanmusierfit-workouts/exercises.db`(表 `exercises`,字段 bvid/video_title/exercise_name/movement_pattern/targets/equipment/position_hint/extraction_confidence)

## 器材限制
只有徒手 + 两个5kg哑铃,没有单杠。查询时必须排除 `equipment LIKE '%单杠%'` 的动作(引体向上、悬垂肩胛骨激活、离心式引体向上)。

## 每周安排(5 训练日 + 2 唤醒日)
- 周一:深蹲 + 推 + 核心
- 周二:深蹲 + 肩推 + 拉(= 周五,复用)
- 周三:硬拉 + 拉 + 核心
- 周四:硬拉 + 拉 + 核心(= 周三,复用)
- 周五:深蹲 + 肩推 + 拉
- 周六/日:身体唤醒(不查数据库,固定动作,见下)

学动作阶段:周二/周四不引入新动作,直接复用已有训练日(周五/周三)的整套动作与教程视频。
均衡版:深蹲 3 天(一/二/五)、硬拉 2 天(三/四),五大基本模式练得更匀,髋铰链也能练熟。

## 选动作规则(保持简单,不做复杂轮换)
每个模式,按出现频次从高到低取**前2个**。用窗口函数确保 video_title 和 bvid 来自同一行(不能分别用 MIN() 各取,会配对错):

```sql
WITH ranked AS (
    SELECT exercise_name, targets, video_title, bvid,
           count(*) OVER (PARTITION BY exercise_name) AS freq,
           ROW_NUMBER() OVER (PARTITION BY exercise_name ORDER BY id) AS rn
    FROM exercises
    WHERE movement_pattern = '<今天的模式>' AND equipment NOT LIKE '%单杠%'
)
SELECT exercise_name, targets, video_title, bvid, freq
FROM ranked WHERE rn = 1
ORDER BY freq DESC LIMIT 2;
```

视频链接 = `https://www.bilibili.com/video/` + bvid

## 身体唤醒日固定内容(周六/日,不查库)
- 原地踏步 1分钟
- 肩绕环 10次
- 髋关节绕环 10次
- 徒手深蹲 10次
- 猫牛式 8次 — 参考《猫牛式 | 伸展脊柱 缓解腰背痛 新人必看》(伽一的瑜伽生活) https://www.bilibili.com/video/BV1dd4y1N7RM (非詹木丝儿fit视频,专门找的瑜伽教学)

## 处方规则(每个动作都要写,不要省略,不要合并成一句通用footnote)
- 前2周:1组×8-10次
- 之后:1组×10-15次
- 完成后休息30-45秒
- 留有余力,不追求力竭

## 推送消息格式
训练日:按锻炼顺序**连续编号**(不按部位分组标题),每条都完整写出动作名+处方+链接,**哪怕和其他条目重复也不要省略或简写成"同上"**:

```
今天周X,练:[部位1] + [部位2] + [部位3],按顺序完成:

1. [动作名]([targets]) — 1组×8-10次(2周后10-15次),完成后休息30-45秒 — 《[video_title]》 https://www.bilibili.com/video/[bvid]
2. [动作名]([targets]) — 1组×8-10次(2周后10-15次),完成后休息30-45秒 — 《[video_title]》 https://www.bilibili.com/video/[bvid]
3. ...(依次编号到当天全部动作数,顺序=[部位1]的动作→[部位2]的动作→[部位3]的动作)
```

唤醒日:
```
今天周X,身体唤醒日,按顺序完成:
1. 原地踏步 1分钟
2. 肩绕环 10次
3. 髋关节绕环 10次
4. 徒手深蹲 10次
5. 猫牛式 8次 — 参考《猫牛式 | 伸展脊柱 缓解腰背痛 新人必看》 https://www.bilibili.com/video/BV1dd4y1N7RM
```

唤醒日:
```
今天周X,身体唤醒日:
原地踏步1分钟、肩绕环10次、髋关节绕环10次、徒手深蹲10次、猫牛式8次
```
