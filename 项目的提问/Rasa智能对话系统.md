1、NLU模块的作用是什么？
在聊天对话中，NLU模块接受用户的输入，输出用户的意图和其中的实体。
例如用户输入”爬泰山“，模型的目标是输出意图 = "爬山"，实体 = ”泰山“。Rasa提供了很多组件用于意图识别和实体抽取，比如DIET。

2、介绍DIET模型
DIET的全称是Dual intend and Entity Transformer,从名字可以知道，它引入了现在前沿的transfomer模型。
DIET的一个特点是它是一个joint模型，它用一个模型同时进行意图识别和实体抽取，联合学习的好处是第一节省了所需的资源，第二是因为意图识别和实体抽取是两个强关联任务，一起训练会有相辅相成的效果。输入的每个token有两个路线，一个路线是取得Sparse features，如词袋模型，character n-gram等，然后输出到全连接层，目的是为了得到和Dense features一样的维度，另一条路线是为了取得Dense features，可以引入预训练好的Glove、Bert等来获得pretrained embedding，然后把Sparse features和Dense features拼接起来，经过全连接层，输入到Transformer block中。
另一个特点是DIET有非常灵活的模型架构，比如说场景一：现在任务要求很低，我们可以不引用Dense features而只使用Sparse features,在模型的损失部分去掉Entity loss和Mask loss，只使用Intend loss，transformer block只取一层。场景二：现在任务很难，并且场景具有很强的专业性，我们可以把Sparse features的embeding size设置为512，同时使用Bert作为预训练模型，因为场景语料很专业，跟预训练语料有差异，我们可以把Mask loss打开。
所以DIET的架构很灵活，参数配置可以根据任务难度和设备资源进行调优。

3、训练中引入Masking损失的目的是什么？BERT这种预训练模型在预训练时，已经带有Mask任务，DIET有继续做Mask的必要吗？
有必要，因为向BERT这类预训练模型，训练语料主要是维基百科，这些训练语料太过官方了，但真实的对话应该是更加口语化的。加入mask任务，是想让模型学习到这些特定的场景，让学习到的模型更加通用化。

