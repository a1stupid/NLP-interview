1、NLU模块的作用是什么？

在任务型对话中，NLU模块接受用户的输入，输出用户的意图和其中的实体。
例如用户输入”爬泰山“，模型的目标是输出意图 = "爬山"，实体 = ”泰山“。Rasa提供了很多组件用于意图识别和实体抽取，比如DIET。

2、介绍DIET模型

DIET的全称是Dual intend and Entity Transformer,从名字可以知道，它引入了现在前沿的transfomer模型。
DIET的一个特点是它是一个joint模型，它用一个模型同时进行意图识别和实体抽取，联合学习的好处是第一节省了所需的资源，第二是因为意图识别和实体抽取是两个强关联任务，一起训练会有相辅相成的效果。输入的每个token有两个路线，一个路线是取得Sparse features，如词袋模型，character n-gram等，然后输出到全连接层，目的是为了得到和Dense features一样的维度，另一条路线是为了取得Dense features，可以引入预训练好的Glove、Bert等来获得pretrained embedding，然后把Sparse features和Dense features拼接起来，经过全连接层，输入到Transformer block中。
另一个特点是DIET有非常灵活的模型架构，比如说场景一：现在任务要求很低，我们可以不引用Dense features而只使用Sparse features,在模型的损失部分去掉Entity loss和Mask loss，只使用Intend loss，transformer block只取一层。场景二：现在任务很难，并且场景具有很强的专业性，我们可以把Sparse features的embeding size设置为512，同时使用Bert作为预训练模型，因为场景语料很专业，跟预训练语料有差异，我们可以把Mask loss打开。
所以DIET的架构很灵活，参数配置可以根据任务难度和设备资源进行调优。

3、训练中引入Masking损失的目的是什么？BERT这种预训练模型在预训练时，已经带有Mask任务，DIET有继续做Mask的必要吗？

有必要，因为向BERT这类预训练模型，训练语料主要是维基百科，这些训练语料太过官方了，但真实的对话应该是更加口语化的。加入mask任务，是想让模型学习到这些特定的场景，让学习到的模型更加通用化。

4、介绍Rasa core模块

在任务型对话中，rasa core模块接受当前用户的意图和抽取到的实体，并且结合历史状态，来判断当前步机器人要采取的action。比如说，第一轮用户的输入是“我想点一份披萨”，它对应的意图是点餐，实体是披萨，输入到rasa core模块中，预测得到当前步要执行的第一个action是information 披萨，第二action是listen，也就是等待用户输入，这个时候机器人回答“你想要什么种类的？”，接着用户继续输入来和机器人聊天。值得关注的是，rasa采取action可以有多种策略，例如基于Rule、Memery和机器学习等。这里面Rule是根据先验知识和业务逻辑来设定一些规则，这些规则可以提前定死，这些规矩解决几轮问题也许可以，但对话轮数一长就覆盖不了所有的情况了。Memory是模型靠历史对话提供的story来进行预测，但真实场景的对话，我们准备的story肯定是应付不过来的。最后一种是基于机器学习来进行预测，它的模型架构是TED，全称是Transformer Embedding Dialogue，TED模块的输入有4个部分，当前步的intend，当前步的entity，到目前为止的slot，上一步的action。然后将四个向量拼接，作为当前轮的输入。最后经过TED模块得到预测的输出。
