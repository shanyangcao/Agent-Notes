# Skills 概念与设计

- **简单比喻**：Skills就像是给Claude准备的"专业教程手册"

  想象你要让一个助手帮你做专业工作，你会给他一份详细的操作指南对吧？Skills就是这样的指南。

  **具体来说**：

  - Skills是预先写好的最佳实践文档
  - 比如创建Word文档的skill，里面详细说明了如何生成高质量的docx文件
  - 创建PPT的skill，包含了如何设计布局、添加内容的最佳方法
  - 这些都是经过大量测试总结出来的经验

  **工作流程**：

  1. 你让Claude做某件事（比如"做个PPT"）
  2. Claude会先查看相关的skill文档
  3. 按照skill里的指导来完成任务
  4. 结果质量更高、更专业

  就像一个工人，MCP给了他工具箱（锤子、扳手、电钻），Skills给了他施工图纸和操作手册。  

  **技术文档**：<https://mcnvohbtbeha.feishu.cn/wiki/SJicwK9EKiGh9Jk1Cb6ctJIKnsc>

  ### 理解skills

  skills其实就是技能

  ![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1772032079392-350eaea2-0ec4-49e1-b395-75c81c6562b3.png)

  ### 定制skills

  #### skills.md

  skills.md内容包含元信息（metadata）和指令（instruction）

  **元信息**：skills名字和skills的描述

  ![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1772513640502-dffc1a26-322d-4ad3-ac83-071763c58ede.png)

  **指令**：具体告诉AI怎么样做提示词

  ![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1772513813749-476a1d64-439f-47c2-8e35-6c5861daa2d8.png)

  #### references

  不同实物放在references包下，进一步按需加载

  ![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1772514349336-21936727-702a-411b-a674-75664dd79213.png)

  #### scripts和assets

  scripts放可执行脚本，assets放图片作为生图参考

  ![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1772514740894-2d4adbfc-378e-401f-a5cd-4199dbae91aa.png)

  ### skills创建

  ### skills推荐

  - [**SkillsMP**](https://skillsmp.com/zh)**：聚合GitHub上超过11万+开源技能。**
  - [**Agent Skills**](https://agent-skills.md/)**：6000+好用的技能。**
  - [**Skills.sh**](https://skills.sh/)**：适合快速发现热门技能，支持一键安装。**
  - [**SkillStore**](https://skillstore.io/zh-hans)**：中文友好的技能商店，上架技能都是经过安全审查的。**
  - [**Skills Directory**](https://www.skillsdirectory.com/)**：Reddit社区推荐的技能合集。**
  - [**Agent Skills (Me)**](https://agentskills.me/)**：编辑精选出来的一些技能合集。**

