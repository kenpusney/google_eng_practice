# 代码审核过程中要看些什么？

## 目录
*   [设计](#design)
*   [功能](#functionality)
*   [复杂性](#complexity)
*   [测试](#tests)
*   [命名](#naming)
*   [注释](#comments)
*   [代码风格](#style)
*   [文档](#documentation)
*   [每行代码](#every_line)
*   [上下文](#contexts)
*   [好的方面](#good_things)
*   [总结](#summary)


注意：在考虑下面这些要素时，时刻谨记要遵循[代码审核标准](standard.md)。

## 设计 {#design}

审核一个 CL 最重要的事情就是考虑它的整体设计。CL 中的代码交互是否有意义？这段代码应该放到代码库（codebase）里，还是库（library）里？它能很好地与系统其他部分集成吗？现在加入这个功能是时机正好吗？

## 功能 {#functionality}

这个 CL 所实现的功能与开发者期望开发的功能是一致的吗？开发者的意图是否对代码的“用户”有好处？此处提到的“用户”通常包含最终用户（使用这些开发出来的功能的用户）和开发者（以后可能会“使用”这些代码的开发者）。

绝大多数情况，我们期望开发者在提交 CL 进行审核之前，已经做过充分的测试。但作为审核者，在审核代码时仍要考虑边界情况、并发问题等等。确保消灭那些通过阅读代码就能发现的缺陷。

作为审核者，你 *可以* 根据需要亲自验证 CL 的功能，尤其是当这个 CL 的行为影响用户交互时，如**UI 变更**。仅通过阅读代码，你很难理解有哪些变更，对用户有哪些影响。对于这种变更，可以让开发者演示这个功能。当然，如果方便把 CL 的代码集成到你的开发环境，你也可以自己亲自尝试。

在代码审核过程中，对功能的考虑还包含一种重要场景：CL 中包含一些“并行计算”，可能会带来死锁或竞争条件。运行代码一般很难发现这类问题，通常需要（开发者和审核者）仔细考虑，以确保不会引入新的问题。（这也是不要引入并发模型的一个好理由，因为它可能引入死锁或竞争条件，同时也增加了代码审核和代码理解的难度。）


## 复杂性 {#complexity}

是不是 CL 可以不必这么复杂？在 CL 的每个层次上检查——哪一行或哪几行是不是太复杂了？功能是否太复杂了？类（class）是否太复杂了？“太复杂”的定义是**代码阅读者不易快速理解。** 同时意味着**以后其他开发者调用或修改它时，很容易引入新的缺陷。**

另一种类型的复杂是**过度工程化**（也称为过度设计）。开发者在设计代码时太过于在意它的通用性，或在系统中加入了目前不需要的功能。审核者应该特别警惕过度工程化。鼓励开发者解决 *当前* 应该解决的问题，而不是开发者推测将来 *可能* 需要解决的问题。将来的问题，等碰到的时候，你才能看到它的实际需求和具体情况，到那时再解决也不迟。

## 测试 {#tests}

同时要求开发者提供 CL 对应的单元测试、集成测试或端到端的测试。测试代码与开发代码应放到同一个 CL 中，除非碰到[紧急情况](../emergencies.md)。

确保 CL 中的测试是正确的、明智的、有用的。测试代码并不是用来测试其自身，我们很少为测试代码写测试代码——这就要求我们确保测试代码是正确的。

当代码出问题时，是否测试会运行失败？如果代码改变了，是否会产生误报？是否每个测试都使用了简单有用的断言？不同的测试方式是否做了合适的拆分？

谨记：测试代码也是需要维护的代码。不要因为不会编译打包到最终的产品中，就接受复杂的测试代码。

## 命名 {#naming}

开发者是否为所有的元素（如类、变量、方法等）选取了一个好名称。一个好名称应该足够长，足以明确地描述它是什么，他能做什么，但是也不要长到难以阅读。

## 注释 {#comments}

开发者是否使用英文写了清晰的注释？是否所有的注释都是必须的？通常当注释**解释为什么**这些代码应该存在时，它才是必须的，而不是解释这些代码做**什么**。如果代码逻辑不清晰，让人看不懂，那么应该重写，让它变得更简单。当然，也有例外（例如，正则表达式和复杂的算法通常需要注释来说明），但大部分注释应该提供代码本身没有提供的信息，如这么做背后的原因是什么。

有时候也应该看一下这个 CL 相关的历史注释。例如，以前写的TODO，现在可以删掉了；某段代码修改了，其注释也应随之修改。

注意，注释与类、模块、功能的 *文档* 是不同的，这类文档应该描述代码的功能，怎样被调用，以及被调用时它的行为是什么。

## 代码风格 {#style}

在Google，我们所有的主要编程语言都要遵循[代码风格指南](http://google.github.io/styleguide/)，确保 CL 遵守代码风格指南中的建议。

如果发现某些风格在代码风格指南中并未提及，在注释中加上“Nit”，让开发者知道，这是一个小瑕疵，他可以按照你的建议去做，但这不是必须的。不要因为个人的风格偏好而导致 CL 延迟提交。

作者在提交 CL 时，代码中不应包含混合着功能变更的较大的代码风格变更。因为这样很难比较出 CL 修改了哪些代码，其后的代码合并、回滚会变得更困难，容易产生问题。如果作者想重新格式化文件，应该把代码格式化作为单独的 CL 先提交，之后再提交包含功能的 CL。

## 文档 {#documentation}

如果 CL 修改了编译、测试、交互、发布的方式，那么应检查下相关的文档是否也更新了，如 README 文件、g3doc 页面，或其他所有生成的参考文档。如果 CL 删除或弃用（deprecate）了一些代码，考虑是否也应删除相应的文档。如果没有这些文档，让开发者（ CL 提交者）提供。

## 每行代码 {#every_line}

在审核代码时，仔细检查 *每行* 代码。某些文件，如数据文件、生成的代码或较大的数据结构，可以一扫而过。但是人写的代码，如类、功能或代码块不能一目十行，我们不应假设它是正确的。有些代码得尤其小心——这需要你自己权衡——至少你应该确认你 *理解* 这些代码在做什么。

如果代码很难读懂，那就放慢审核速度，告诉开发者你没读懂代码，让他解释与澄清，之后继续审核。在 Google，我们雇佣都是伟大的工程师，你是其中一员。如果你读不懂代码，很有可能其他工程师也不懂。实际上，这么做也是在帮助以后的工程师，当他读到这段代码时更容易理解代码。所以，让开发者解释清楚。

如果你理解这些代码，但是感觉自己不够资格审核它，确保找到一个够资格的人来审核，尤其是比较复杂的问题，如安全、并发、可访问性、国际化等等。

## 上下文 {#contexts}

把 CL 放到一个更广的上下文中来看，通常很有用。在审核工具中，我们往往只能看到开发者修改的那部分代码。更多时候从整个文件的角度来读代码才有意义。例如，有时候你只看到添加了几行代码，但从整个文件来看，你发现这4行代码添加到了一个50行的方法中。在增加之后，需要把它拆分成更小的方法。

把 CL 放到系统的上下文中来考虑也很有用。CL 能提升系统的代码健康状况，还是让系统变得更复杂、更难测试？**不要接受恶化系统健康状况的代码。**大多数系统变得很复杂都是由每个细小的复杂累积而成的，在提交每个 CL 时都应避免让代码变得复杂。

## 好的方面 {#good_things}

如果在 CL 中看到一些比较好的方面，告诉开发者，尤其是当你在审核代码时添加了评论，他在回复你的评论，尝试向你解释的时候。审核者往往只关注代码中的错误，他们也应该对开发者的优秀实践表示鼓励和感谢。有时候，告诉开发者他们在哪些方面做得很好，比告诉他们在哪些方面做得不足更有价值。

## 总结 {#summary}

在进行代码审核时，应确保如下几点：
-   代码是否设计良好。
-   功能是否对代码的用户有用。
-   所有的UI变更都是明智的，看起来很不错。
-   所有的并行计算都很安全。
-   代码尽量简单。
-   开发这没有开发现在不需要，但是他们认为将来 *可能* 会用到的功能。
-   代码有合适的单元测试。
-   测试设计良好。
-   开发者是否使用了清晰的命名。
-   注释是否清晰、有用，大多数应该解释 *为什么* ，而不是 *什么* 。
-   代码是否包含合适的文档（通常是 g3doc ）。
-   代码是否遵循代码风格指南。

确保审核了**每行代码**，要查看上下文，确保你**正在提升代码质量**，当开发者的 CL 中包含 **好的方面** 时，称赞他们。

下一章：[代码审核的步骤](navigate.md)
