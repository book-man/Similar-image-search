## 相似图像搜索

### 1. 图像指纹

人的指纹可以作为人的标识，来区分你是你，不是他，主要是因为这个指纹独一无二，如果两个指纹相同，那么可以认为对应同一个人。就像如果人的血液 DNA 极其相近，那么也可以认为是一个人，或者是亲人。同样，如果几个图像的指纹相同或相似，那么可以认为它们对应相同或相似的图像。那么什么是图像的指纹呢？这是一个问题。

图像指纹可以理解为对图像进行的某种变换的结果，当然，这种变换要满足一下几个准则：

1. 不同的图像得到的变换应该不同，或者说只有相同或相似图像变换后的结果才相同或相似。
2. 这种变换不能过于复杂，不然建立指纹库或者查询的时间会过于复杂。
3. 输出结果不能过于复杂，方便指纹库的存储

在本文中我们使用三种指纹，指纹的大小都是 8x8=64 ，以 Lena 图为例。


#### 1. 第一种指纹

> 称为 hash 法。

1. 输入图像，如果是 RGB 图像则转换为灰度图像
2. 缩放，将图像缩放为 8x8 大小的图像
3. 求均值，计算此 8x8 图像所有像素的均值
4. 量化，比较 8x8 图像中每一个像素像素与均值大小关系，如果大于均值，则指纹上该位对应的值为1；否则，为0。
5. 组合，将这64bit的0和1组成这幅图像的指纹。

输出如下所示：

>1101000100000000101000001110000011111111111000111001000000000000

当然，为了方便，可以把上述64bit写成16进制数：
> D100A0E0FFE39000


#### 2. 第二种指纹

> 称为 phash 法。

1. 输入图像，如果是 RGB 图像则转换为灰度图像
2. 缩放，将图像缩放为 32x32 大小的图像，注意此时不是 8x8
3. DCT变换，对上述 32x32 的图像进行dct变换
4. 取低频，对图像进行变换后的大小为 32x32，如果以此作为图像的指纹会浪费太多空间，所以此处只取低频部份，也就是变换后的左上 8x8 的块
5. 求均值，计算此 8x8 块的所有像素的均值
6. 量化，比较 8x8 块中每一个像素与均值大小关系，如果大于均值，则指纹上该位对应的值为1；否则，为0。
7. 组合，将这64bit的0和1组成这幅图像的指纹。

输出如下所示：

>1100101111100001011101101111000010100100111100000001011010010111

当然，为了方便，可以把上述64bit写成16进制数：
> F445821C820CAF3

#### 3. 第三种指纹

> 称为 dhash 法。

1. 输入图像，如果是 RGB 图像则转换为灰度图像
2. 缩放，将图像缩放为 8x9 大小的图像，注意，此时不是 8x8
3. 量化，比较 8x9 图像中的前 8 列 像素中每一个像素与它右侧像素的大小，如果大于右侧像素，则指纹上该位对应的值为1；否则，为0。
5. 组合，将这64bit的0和1组成这幅图像的指纹。

输出如下所示：

>1101110010011111100110110110010000101101101111001110000110010111

当然，为了方便，可以把上述64bit写成16进制数：
> DC9F9B642DBCE197

### 2. 建立指纹库

要找相似图像，首先我们需要有一个现成的数据集，作为后面的查找库。

假设已有数据集中有数以千计、万计的图像。分别计算每一个图像的指纹（或者称为哈希），然后建立该指纹与图像的一一对应关系，也就是说：知道指纹，就可以找到该图像。

### 3. 识别

要判断两幅图像是否相似，就是要判断它们的指纹是否相似，如果指纹相似则认为对应的两幅图像像素。那么怎么样的指纹才叫相似呢，这里用到了汉明距离，0与0的汉明距离为0，0与1的汉明距离为1，010与100的汉明距离为2，也就是说，针对二进制数，汉明距离可以理解为不同bit位的个数。如果两个指纹的所有bit位完全相同，可以认为这两个指纹相同，或者放宽松点，如果两个指纹只有0或1或2个bit位不同，则认为这两个指纹相似。


具体的识别过程如下所示：

1. 输入待搜索图像
2. 计算该图像的指纹
3. 将该图像的指纹与数据库中的指纹进行对比
4. 如果在数据库中找到与输入图像的指纹相似的指纹，则认为找到匹配的相似图像
5. 输出找到的相似图像

> 注意：上述方式需要用输入指纹与数据库中的所有指纹进行对比，如果数据库较大，那么时间复杂度会比较高，为 O(n)，n 为指纹库的大小，此时需要进行一些改进。

+ 使用哈希表
    将指纹组成十六进制数，作为查找的键值，这样就可以在 O(1) 的时间复杂度内找到是否存在匹配的指纹。此种方法依旧存在问题，也就是只能查找完全相同的指纹，而不能查找相似的指纹。
+ 使用 kd 树
    此时虽然复杂度降为 O(lg(n))，但是可以查找相似指纹，且不用遍历所有指纹，效果较好。
+ 使用并行查找
 要把输入图像的指纹与所有指纹库中的指纹库进行对比，可以使用并行的思路，每个线程复杂处理一部分的匹配。

### 4. 实验效果

本文采用[CALTECH-101](http://www.vision.caltech.edu/Image_Datasets/Caltech101/)数据集进行测试，此数据集包含101个场景的共9144幅图像。

首先根据 hash 法、phash法、dhash法进行对比实验，试验组随机选择了[CALTECH-101](http://www.vision.caltech.edu/Image_Datasets/Caltech101/)数据中的953个图像（随机产生1000个数，去重之后剩余953）。实验结果如下表所示：

| 方法       |  一个匹配 | 多个匹配  | 精度 |
| :-------- | :--------:| :--:   | :--:   |
| hash      | 87       |  866     | 90.87% |
| phash     |  5       |  948     | 99.48% |
| dhash     |  5       |  948     | 99.48% |

> 说明：
> > 1. 在匹配指纹时选择的阈值为1，也就是说汉明距离小于等于1则认为指纹匹配。
> > 2. 实际上 phash 组的结果里面并不是只有5个一一匹配的结果，还有22个如下图中的第三列所示，观察后发现这22组结果有明显的规律性，间距均为100，于是找到了其对应的分组，也就是[CALTECH-101](http://www.vision.caltech.edu/Image_Datasets/Caltech101/)数据中的Leopards分组，该分组的前100个图像与后100个图像一一对应，呈左右镜像的形式，也就是说，下图中的第1348个图像左右反转之后就是第1448个图像，也就是说，phash法可以正确搜索出镜像的图像，而hash法和dhash法不能。
>>> 这是因为DCT变换具有一定的对称性，而phash法中计算指纹就是基于dct方法。


> 补充：
> > 1. 上述方法都不具有旋转不变性，也就是对于旋转的图像都无法识别。
> > 2. 第三种指纹相当去先将图像缩放，然后求水平差分，然后量化为0和1。

