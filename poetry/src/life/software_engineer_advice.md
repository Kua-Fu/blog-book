# software engineer advice

20 Things I’ve Learned in my 20 Years as a Software Engineer

## 一、参考

> [20 Things I’ve Learned in my 20 Years as a Software Engineer](https://www.simplethread.com/20-things-ive-learned-in-my-20-years-as-a-software-engineer/)

## 二、Important, Read This First

You’re about to read a blog post with a lot of advice. Learning from those who came before us is instrumental to success, but we often forget an important caveat. Almost all advice is contextual, yet it is rarely delivered with any context.

“You just need to charge more!” says the company who has been in business for 20 years and spent years charging “too little” to gain customers and become successful.


“You need to build everything as microservices!” says the company who built a quick monolith, gained thousands of customers, and then pivoted into microservices as they started running into scaling issues.


你即将阅读一篇包含许多建议的博客文章。从前人的经验中学习对于成功至关重要，但我们常常忽视一个重要的前提条件。几乎所有的建议都是有情境背景的，然而很少有人在传达建议时提供任何背景信息。

“你只需要提高价格！” 这是一家经营了20年的公司说的，他们花了很多年时间以“太低”的价格来吸引客户并取得成功。

“你需要将所有东西都构建成微服务！” 这是一家迅速构建了一个单体架构、获得了数千客户，然后在遇到扩展问题时转向微服务的公司所说。

Without understanding the context, the advice is meaningless, or even worse, harmful. If those folks had followed their own advice early on, they themselves would likely have suffered from it. It is hard to escape this trap. We may be the culmination of our experiences, but we view them through the lens of the present.

So to give you a little context on where my advice comes from, I spent the first half of my career as a software engineer working for various small businesses and startups, then I went into consulting and worked in a number of really large businesses. Then I started Simple Thread and we grew from a team of 2 to a team of 25. 10 years ago we worked with mostly small/medium businesses, and now we work with a mix of big and small businesses.

如果不理解背景，这些建议就是毫无意义，甚至可能更糟，会带来伤害。如果这些人早期就采纳了他们自己的建议，他们自己很可能会因此受害。摆脱这个陷阱是很困难的。这些可能是之前经验的结晶，但我们需要通过现在的视角来看待它们。

为了让你对我提出的建议有一些背景了解，我前半段职业生涯是作为一名软件工程师在各种小型企业和创业公司工作，然后我进入了咨询行业，在一些非常大的企业工作过。然后我创办了Simple Thread，我们的团队从2人发展到了25人。10年前，我们主要与中小型企业合作，现在我们与大大小小的企业都有合作。

My advice is from someone who…

1. has almost always been on small, lean teams where we have to do a lot with very little.

1. values working software over specific tools.

1. is starting new projects all the time, but also has to maintain a number of systems.

1. values engineer productivity over most other considerations

My experiences over the last 20 years have shaped how I view software, and have led me to some beliefs which I’ve tried to whittle down to a manageable list that I hope you find valuable.

我的建议来自一个...

1. 几乎总是在小而精干的团队中工作，在这些团队中我们必须用极少的资源做很多事情。

1. 更看重可工作的软件，而非特定的工具。

1. 经常启动新项目，但同时也必须维护多个系统。

1. 更重视工程师的生产力，而非其他大部分考虑因素。

过去20年的经验塑造了我对软件的看法，并使我形成了一些信念，我试图将它们精简为一个可以管理的列表，希望你会觉得有价值。

## 三、On with the list

接下来就是这个列表

### 1. I still don’t know very much

我仍然了解得不多

“How can you not know what BGP is?” “You’ve never heard of Rust?” Most of us have heard these kinds of statements, probably too often. The reason many of us love software is because we are lifelong learners, and in software no matter which direction you look, there are wide vistas of knowledge going off in every direction and expanding by the day. This means that you can spend decades in your career, and still have a huge knowledge gap compared to someone who has also spent decades in a seemingly similar role. The sooner you realize this, the sooner you can start to shed your imposter syndrome and instead delight in learning from and teaching others.


“你怎么可能不知道BGP是什么？” “你从来没听说过Rust吗？” 我们中的大多数人都听过这些说法，可能太多次了。之所以我们中的许多人热爱软件，是因为我们是终身学习者，在软件领域，无论你往哪个方向看，都有广阔的知识领域，每个方向都在不断扩展。这意味着你可以在职业生涯中度过几十年，但与那些在看似类似的角色中度过几十年的人相比，你仍然存在巨大的知识差距。你越早意识到这一点，你就能越早地摆脱冒名顶替的心理，而是愉快地从他人那里学习并教导他人。

【注】BGP是Border Gateway Protocol的缩写，一种用于在互联网中交换路由信息的协议。Rust是一种系统编程语言。


### 2. The hardest part of software is building the right thing

软件开发中最困难的部分是构建正确的东西

I know this is cliche at this point, but the reason most software engineers don’t believe it is because they think it devalues their work. Personally I think that is nonsense. Instead it highlights the complexity and irrationality of the environments in which we have to work, which compounds our challenges. You can design the most technically impressive thing in the world, and then have nobody want to use it. Happens all the time. Designing software is mostly a listening activity, and we often have to be part software engineer, part psychic, and part anthropologist. Investing in this design process, whether through dedicated UX team members or by simply educating yourself, will deliver enormous dividends. Because how do you really calculate the cost of building the wrong software? It amounts to a lot more than just lost engineering time.

我知道这已经是陈词滥调了，但大多数软件工程师不相信这一点的原因是因为他们认为这会贬低他们的工作。我个人认为这是无稽之谈。相反，这凸显了我们必须工作的环境的复杂性和非理性，这加剧了我们的挑战。你可以设计出世界上技术上最令人印象深刻的东西，然后却没有人想使用它。这种情况经常发生。设计软件在很大程度上是一种倾听的活动，我们通常既必须是软件工程师，又必须是心灵感应者，还要是人类学家。投资于这个设计过程，无论是通过专门的用户体验团队成员，还是简单地自我教育，都将带来巨大的回报。因为你真的如何计算构建错误软件的成本呢？这远不止是失去的工程时间那么简单。

【注】UX是用户体验（User Experience）的缩写，指的是用户在使用产品或系统时的整体体验。

### 3. The best software engineers think like designers

Great software engineers think deeply about the user experience of their code. They might not think about it in those terms, but whether it is an external API, programmatic API, user interface, protocol, or any other interface; great engineers consider who will be using it, why it will be used, how it will be used, and what is important to those users. Keeping the user’s needs in mind is really the heart of good user experience


最优秀的软件工程师像设计师一样思考

优秀的软件工程师深入思考他们代码的用户体验。他们可能并不会用这种术语来思考，但无论是外部API、编程API、用户界面、协议还是任何其他界面；优秀的工程师会考虑谁将使用它，为什么会被使用，如何使用以及对这些用户来说什么是重要的。牢记用户的需求实际上是良好用户体验的核心。

【注】API是应用程序编程接口（Application Programming Interface）的缩写，是一种不同软件系统之间交互的方式。

### 4. The best code is no code, or code you don’t have to maintain

All I have to say is “coders gonna code.” You ask someone in any profession how to solve a problem, and they are going to err on the side of what they are good at. It is just human nature. Most software engineers are always going to err on the side of writing code, especially when a non-technical solution isn’t obvious. The same goes for code you don’t have to maintain. Engineering teams are apt to want to reinvent the wheel, when lots of wheels already exist. This is a balancing act, there are lots of reasons to grow your own, but beware of toxic “Not Invented Here” syndrome.

最好的代码是没有代码，或者是不需要维护的代码

我只想说：“编码者总是会编码。” 你向任何一个行业的人询问如何解决一个问题，他们都会偏向于他们擅长的领域。这只是人类的本性。大多数软件工程师总是倾向于编写代码，尤其是在非技术解决方案不明显的情况下。对于不需要维护的代码也是一样。工程团队倾向于想要重新发明轮子，即使已经有很多轮子存在。这是一个平衡的过程，有很多理由可以自己开发，但要注意有毒的“非本土”综合症。

【注】“Not Invented Here” syndrome，中文翻译为“非本土”综合症，指的是一个团队或组织倾向于拒绝使用外部创意、产品或解决方案，而更愿意自行开发、创造。

### 5. Software is a means to an end

The primary job of any software engineer is delivering value. Very few software developers understand this, even fewer internalize it. Truly internalizing this leads to a different way of solving problems, and a different way of viewing your tools. If you really believe that software is subservient to the outcome, you’ll be ready to really find “the right tool for the job” which might not be software at all.

软件是达成目标的手段

任何一名软件工程师的主要职责是提供价值。很少有软件开发人员理解这一点，更少的人能够内化这个观念。真正内化这一点会导致以不同的方式解决问题，以及以不同的视角看待你的工具。如果你真的相信软件是为了达成结果而存在的，你将会准备好真正找到“最适合的工具”，而这可能并不一定是软件。

### 6. Sometimes you have to stop sharpening the saw, and just start cutting shit

Some people tend to jump into problems and just start writing code. Other people tend to want to research and research and get caught in analysis paralysis. In those cases, set a deadline for yourself and just start exploring solutions. You’ll quickly learn more as you start solving the problem, and that will lead you to iterate into a better solution.

有时候你必须停止磨练技能，直接开始动手

有些人倾向于迅速投入解决问题并开始编写代码。而另一些人则倾向于不断进行研究，深陷于分析瘫痪。在这些情况下，为自己设定一个截止日期，然后开始探索解决方案。当你开始解决问题时，你会迅速学到更多，这将引导你不断迭代，找到更好的解决方案。

### 7. If you don’t have a good grasp of the universe of what’s possible, you can’t design a good system

This is something I struggle with a lot as my responsibilities take me further and further from the day to day of software engineering. Keeping up with the developer ecosystem is a huge amount of work, but it is critical to understand what is possible. If you don’t understand what is possible and what is available in a given ecosystem then you’ll find it impossible to design a reasonable solution to all but the most simple of problems. To summarize, be wary of people designing systems who haven’t written any code in a long time.

如果你不对可能性的范围有很好的把握，就无法设计出一个好的系统。

这是我在工作中经常遇到的问题，随着我的责任越来越远离日常的软件工程。跟上开发者生态系统的发展是一项巨大的工作，但了解什么是可能的是至关重要的。如果你不了解在给定的生态系统中可能性和可用性，那么除了最简单的问题，你将发现几乎不可能设计出合理的解决方案。总结一下，要谨慎对待那些已经很长时间没有编写过代码的人设计系统的观点。


### 8. Every system eventually sucks, get over it

Bjarne Stroustrup has a quote that goes “There are only two kinds of languages: the ones people complain about and the ones nobody uses”. This can be extended to large systems as well. There is no “right” architecture, you’ll never pay down all of your technical debt, you’ll never design the perfect interface, your tests will always be too slow. This isn’t an excuse to never make things better, but instead a way to give you perspective. Worry less about elegance and perfection; instead strive for continuous improvement and creating a livable system that your team enjoys working in and sustainably delivers value.

每个系统最终都会变得糟糕，接受这个事实吧

Bjarne Stroustrup有一句名言：“只有两种类型的语言：人们抱怨的语言和没人使用的语言”。这个观点同样适用于大型系统。没有所谓的“正确”架构，你永远无法还清所有的技术债务，你永远无法设计出完美的接口，你的测试始终会太慢。这并不是一个不去改进的借口，而是一种让你保持透视的方式。少一些对优雅和完美的担忧，而是追求持续改进，创造一个团队喜欢并能够持续提供价值的可维护系统。

【注】Bjarne Stroustrup是C++语言的创始人之一，他提出了C++的设计理念和开发。

### 9. Nobody asks “why” enough

Take any opportunity to question assumptions and approaches that are “the way things have always been done”. Have a new team member coming on board? Pay attention to where they get confused and what questions they ask. Have a new feature request that doesn’t make sense? Make sure you understand the goal and what is driving the desire for this functionality. If you don’t get a clear answer, keep asking why until you understand.

很少人问了足够多的问什么？

抓住任何机会来质疑那些“一直以来都是这样做的”假设和方法。有新的团队成员加入吗？注意他们在哪些地方感到困惑，以及他们提出了什么问题。有一个不合理的新功能请求吗？确保你理解了目标以及是什么驱使着对这个功能的渴望。如果你没有得到一个清晰的答案，就一直问“为什么”，直到你理解为止。

### 10. We should be far more focused on avoiding 0.1x programmers than finding 10x programmers

The 10x programmer is a silly myth. The idea that someone can produce in 1 day what another competent, hard working, similarly experienced programmer can produce in 2 weeks is silly. I’ve seen programmers that sling 10x the amount of code, and then you have to fix it 10x the amount of times. The only way someone can be a 10x programmer is if you compare them to 0.1x programmers. Someone who wastes time, doesn’t ask for feedback, doesn’t test their code, doesn’t consider edge cases, etc… We should be far more concerned with keeping 0.1x programmers off our teams than finding the mythical 10x programmer.

我们应该更专注于避免0.1倍程序员，而不是寻找10倍程序员

关于10倍程序员的观点是荒谬的神话。认为某人可以在1天内完成另一位有能力、勤奋工作、经验相似的程序员在2周内才能完成的工作是荒谬的。我见过一些编程者编写了10倍数量的代码，然后你需要花10倍的时间来修复。某人能成为10倍程序员的唯一方式是将他们与0.1倍程序员进行比较。那些浪费时间、不寻求反馈、不测试他们的代码、不考虑边缘情况等的人… 我们应该更关心如何防止0.1倍程序员加入我们的团队，而不是寻找那些神秘的10倍程序员。

### 11. One of the biggest differences between a senior engineer and a junior engineer is that they’ve formed opinions about the way things should be

高级工程师和初级工程师之间最大的区别之一是他们对事物应该如何的看法已经形成。

Nothing worries me more than a senior engineer that has no opinion of their tools or how to approach building software. I’d rather someone give me opinions that I violently disagree with than for them to have no opinions at all. If you are using your tools, and you don’t love or hate them in a myriad of ways, you need to experience more. You need to explore other languages, libraries, and paradigms. There are few ways of leveling up your skills faster than actively seeking out how others accomplish tasks with different tools and techniques than you do.

没有什么比一个高级工程师，对工具或构建软件方法，没有看法，更让我担忧了。我宁愿有人给我提出激烈反对意见，也不愿意看到他们完全没有任何意见。如果你正在使用工具，而且你无法在各种各样的方式中喜欢或讨厌它们，那么你需要更多地积累经验。你需要探索其他编程语言、库和范例。比起被动学习，主动寻找不同工具和技术的方法，来了解其他人如何完成任务，是提升你技能水平的几种方法之一。

### 12. People don’t really want innovation

People talk about innovation a whole lot, but what they are usually looking for is cheap wins and novelty. If you truly innovate, and change the way that people have to do things, expect mostly negative feedback. If you believe in what you’re doing, and know it will really improve things, then brace yourself for a long battle.

人们实际上并不真正想要创新

人们谈论创新的频率很高，但他们通常寻求的是廉价的成功和新奇。如果你真正创新，改变了人们的做事方式，预计会遭到大多数负面反馈。如果你相信自己正在做的事情，并知道这将真正改善情况，那就为漫长的斗争做好准备。

### 13. Your data is the most important part of your system

I’ve seen a lot of systems where hope was the primary mechanism of data integrity. In systems like this, anything that happens off the golden path creates partial or dirty data. Dealing with this data in the future can become a nightmare. Just remember, your data will likely long outlive your codebase. Spend energy keeping it orderly and clean, it’ll pay off well in the long run.

你的数据是系统中最重要的部分

我见过很多系统，其中希望是数据完整性的主要机制。在这样的系统中，任何偏离正常路径的操作都会导致部分或脏数据。未来处理这些数据可能会变成一场噩梦。只要记住，你的数据可能会在很长时间内超越你的代码库存在。花时间保持数据的有序和清洁，从长远来看会有很大的回报。

### 14. Look for technological sharks


Old technologies that have stuck around are sharks, not dinosaurs. They solve problems so well that they have survived the rapid changes that occur constantly in the technology world. Don’t bet against these technologies, and replace them only if you have a very good reason. These tools won’t be flashy, and they won’t be exciting, but they will get the job done without a lot of sleepless nights.

寻找技术上的“鲨鱼”

那些长期存在的老旧技术不是恐龙，而是鲨鱼。它们解决问题的效果非常好，以至于它们在技术领域不断发生的快速变化中存活下来。不要押注对抗这些技术，只有在你有很好的理由时才替换它们。这些工具可能不会引人注目，也不会激动人心，但它们将会高效地完成任务，而不会让你夜不能寐。


### 15. Don’t mistake humility for ignorance

There are a lot of software engineers out there who won’t express opinions unless asked. Never assume that just because someone isn’t throwing their opinions in your face that they don’t have anything to add. Sometimes the noisiest people are the ones we want to listen to the least. Talk to the people around you, seek their feedback and advice. You’ll be glad you did.

不要将谦虚误认为无知

有很多软件工程师不会主动表达自己的意见，除非被问及。永远不要假设，仅因为某人没有把自己的意见摆在你面前，就认为他们没有什么可以补充的。有时候最喧嚣的人恰恰是我们最不想倾听的人。与你周围的人交流，寻求他们的反馈和建议。你会庆幸自己这样做了。

### 16. Software engineers should write regularly

Software engineers should regularly blog, journal, write documentation and in general do anything that requires them to keep their written communication skills sharp. Writing helps you think about your problems, and helps you communicate those more effectively with your team and your future self. Good written communication is one of the most important skills for any software engineer to master.

软件工程师应该定期进行写作

软件工程师应该定期撰写博客、日志、编写文档，总之要进行任何需要保持他们书面沟通技巧敏锐的活动。写作有助于你思考问题，并帮助你更有效地与团队以及未来的自己进行沟通。良好的书面沟通是任何软件工程师都需要掌握的最重要的技能之一。

### 17. Keep your processes as lean as possible

Everyone wants to be agile these days, but being “agile” is about building things in small chunks, learning, and then iterating. If someone is trying to shoehorn much more into it than that, then they’re probably selling something. It isn’t to say that people don’t need accountability or help to work this way, but how many times have you heard someone from your favorite tech company or large open source project brag about how great their Scrum process is? Stay lean on process until you know you need more. Trust your team and they will deliver.

尽可能保持你的流程简洁

如今每个人都想要变得“敏捷”，但“敏捷”是关于分阶段构建事物、学习，然后不断迭代。如果有人试图把更多东西强行塞进去，那么他们可能在推销某些东西。这并不是说人们不需要负责或帮助以这种方式工作，但你有多少次听到你最喜欢的科技公司或大型开源项目中的人吹嘘他们的Scrum流程有多好？在你确实需要更多流程之前，保持流程简洁。相信你的团队，他们会交付结果。

### 18. Software engineers, like all humans, need to feel ownership

If you divorce someone from the output of their work, they will care less about their work. I see this almost as a tautology. This is the primary reason why cross-functional teams work so well, and why DevOps has become so popular. It isn’t all about handoffs and inefficiencies, it is about owning the whole process from start to finish, and being directly responsible for delivering value. Give a group of passionate people complete ownership over designing, building, and delivering a piece of software (or anything really) and amazing things will happen.

像所有人类一样，软件工程师需要有归属感

如果你让某人与他们的工作成果分离，他们会对工作产生较少的关心。我几乎将这视为一个自明的道理。这是为什么跨职能团队如此有效，以及为什么DevOps变得如此受欢迎的主要原因。这并不仅仅关于交接和低效，而是关于从头到尾拥有整个过程，并直接负责交付价值。让一组充满热情的人完全负责设计、构建和交付一款软件（或其他任何东西），将会发生令人惊讶的事情。

### 19. Interviews are almost worthless for telling how good of a team member someone will be

Interviews are far better spent trying to understand who someone is, and how interested they are in a given field of expertise. Trying to suss out how good of a team member they will be is a fruitless endeavor. And believe me, how smart or knowledgable someone is is also not a good indicator that they will be a great team member. No one is going to tell you in an interview that they are going to be unreliable, abusive, pompous, or never show up to meetings on time. People might claim they have “signals” for these things… “if they ask about time off in the first interview then they are never going to be there!” But these are all bullshit. If you’re using signals like these you’re just guessing and turning away good candidates.

面试几乎无法判断某人是否是一个优秀的团队成员

面试更好地花在努力了解一个人是谁，以及他们对特定领域的专业知识有多感兴趣。试图了解某人是否是一个出色的团队成员是徒劳的。而且请相信我，某人有多聪明或知识丰富也不是他们是否会成为一个出色团队成员的好指标。在面试中，没有人会告诉你他们会不可靠、虐待他人、自大或者不准时参加会议。人们可能声称他们对这些事情有“信号”……“如果他们在第一次面试中问起休假时间，那么他们就不会来上班！”但这些都是胡说八道。如果你使用这样的信号，你只是在猜测，并且可能会错过很多优秀的候选人。

### 20. Always strive to build a smaller system

There are a lot of forces that will push you to build the bigger system up-front. Budget allocation, the inability to decide which features should be cut, the desire to deliver the “best version” of a system. All of these things push us very forcefully towards building too much. You should fight this. You learn so much as you’re building a system that you will end up iterating into a much better system than you ever could have designed in the first place. This is surprisingly a hard sell to most people.


始终努力构建更小的系统

有许多因素会推动你在一开始就构建更大的系统。预算分配、无法决定应该削减哪些功能、希望交付系统的“最佳版本”。所有这些因素都强烈地推动我们朝着构建过多的方向发展。你应该抵制这种趋势。在构建一个系统的过程中，你会学到很多东西，最终会将其迭代为一个比你一开始设计的更好的系统。令人惊讶的是，这在大多数人那里很难得到认同。
