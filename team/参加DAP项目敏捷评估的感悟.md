参加DAP项目敏捷评估的感悟
=========================

>文中图片来源网络，版权属于原图作者

9月中旬研究院组织了一系列项目敏捷评估活动，作为研究院代码重构教练的我获得了参加对DAP项目敏捷评估的机会，这是我第一次以外部教练的身份进入别的项目观摩敏捷实践活动，以往的经验只是带领团队进行敏捷实践，对于敏捷活动的评估基本上是一片空白。因此就我个人而言，此次活动是一次难得的机会去学习如何评判敏捷实践开展的情况，去学习如何发现项目中存在的痛点及改进方向。

DAP项目敏捷评估活动从9.14开始，持续4天，研究院选派了2位公司级敏捷教练（张莹、程冲）和3位研究院级敏捷教练（李加周、周晓俊、我）组成调研小组进入DAP项目，同时研究院技术部的大牛（高静、陈刚、贺晋宁、施绍国、王志慧）和外聘的咨询师（刘传湘）作为调研小组的顾问。

#方向

调研方向怎么选，这是从收到参加调研活动通知后萦绕在我心头的问题，通常对某个问题的研究都会借助一套模型或者工具，但对我毫无经验的我而言，模型和工具都无从说起。不过这个问题很快就被巧妙地解决了，而且在我看来解决的非常行之有效，且不会有任何提前引入模型带来的先入为主的影响，调研小组请DAP项目为小组进行了简单的项目介绍，涉及项目的组织结构，日常运行，工作实践以及自身痛点和改进，并不冗长的介绍过程迅速地聚焦了关注点。由于每位教练对项目关注点不同，因此介绍过程中各种问题自然而然也就产生了。外聘咨询师传湘用一种回顾会常用的方式让我们自己完成了调研方向的确定，将项目介绍过程我们发现的项目亮点及对项目的疑问写成纸条，通过小组的头脑风暴，将亮点和疑问分成了若干个关键词的集合，接下来的就是教练根据自身关注点选择相应的关键词，调研方向也就落到了不同的教练身上，这种方式十分自然地就让教练有了带入感，根据听到的去制定调研策略而非凭空地创造，更减少了调研方向错误的几率，更重要的是传湘提醒我们，要从对项目、团队运行观察和访谈中发现问题。

>技能1: 项目介绍对于咨询师而言是一个很好的确定调研方向的素材，无论亮点还是疑问都可以通过分类归纳出若干方向

#实施

关键词中的测试策略和DevOps是我的调研方向，在晓俊和施总的帮助下，我们采用了一种分层多维度的方式，关注到从项目到团队中涉及的开发、测试角色，由开发经理、项目CI管理员、SM、Dev再到QA、测试团队，评估对象包括CI环境、代码走查、编码规范及测试用例。调研过程主要以访谈为主，从项目和团队的每个角色眼中去审视整个项目，与开发经理、CI管理员的访谈，我们看到了以流水线样式呈现的版本构建、测试、发布流程和多语言构建脚本，也看到了开发经理和CI管理员面对的CI困境；与开发团队SM、Dev的访谈，我们看到了相当出色的团队编码纪律，也看到了开发团队对自动化测试的迫切需求；与QA及测试团队的访谈，我们看到了测试与开发协作加速需求理解，也看到缺少测试前移带来的问题。

在访谈的同时，教练之间也加强沟通，参照Scrum团队，整个调研组每日都会有一次晨会及午间讨论，对调研结果进行汇总，特别是提醒其他调研方向小组自己发现的属于该方向的问题或信息，更是方便教练随时能够转换视角，统一规划的调研资源也防止了重复浪费。

>技能2: 定期检视调研任务进行情况，有助于了解任务进度，同时相互交流可以给予双方全新视角，并有助对资源合理统一使用

#输出

![blind_elephant](http://static3.photo.sina.com.cn/middle/4e93ef2ft82dbe4a407b2&690)

在输出最后的调研报告前，李部的一句"其实我们在大多数时候是盲人摸象"，这句话提醒大家在输出报告前开始用统一的视角来整体性地看待DAP项目，进行到这个阶段，终于见识到了几位资深教练对问题认识、把握的深度与准度，经过反复的讨论与修改，一个便于理解和解读的痛点改进措施模型诞生，此模型演进出报告的大纲，同时"按图说话"也成为了报告的一条编写准则，尽量以直观、清晰的图表来描述问题，即使是需要语言描述，李部也要求简短明确，不要有太多的修饰，用一句话讲问题讲述清楚。

报告经过再三修改和内部审议后，在最终发布之前被发送给几位DAP项目的负责人进行预先沟通，预先沟通消除可能出现的误会，当报告中提及的所有问题都被确认后，整个报告就可以安全地发布。

>技能3: 调研分立的观点在最后需要通过统一的视界来完善，否则就会出现盲人摸象，不知所云

>技能4: 图比文字更直观，文字描述简短更有力，问题描述不要过多修饰，容易喧宾夺主

>技能5: 结论在发布前要和利益相关人进行沟通，消除无意义的误会，建立安全氛围

>技能6: 会议正式开始前，进行简短的icebreak可以迅速拉近双方距离，消除隔阂

#感谢

最后再一次感谢研究院技术部挑选我参加此次调研，这样的经历和体验都十分深刻。感谢李加周部长、张莹、程冲、周晓俊在此次调研过程的给予我的帮助，你们的对敏捷的理解和知识让我学习了很多，很多方法、技能都为我提供了新的思考和视野，希望有机会能够再合作。
