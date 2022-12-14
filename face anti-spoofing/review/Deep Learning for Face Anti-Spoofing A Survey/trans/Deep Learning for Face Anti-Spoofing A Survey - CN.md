# Deep Learning for Face Anti-Spoofing: A Survey

## 3. 使用商用RGB相机的深度FAS

由于商用RGB相机在许多真实世界应用场景(如，权限控制系统和移动设备解锁)中已经广泛使用，在这一章节，我们将回顾现有的基于商用RGB相机的FAS方法。图6介绍了几个具有里程碑意义的深度FAS方法。

### 3.1 混合方法(手工+深度学习)

虽然深度学习和CNN在许多计算机视觉任务(如，图像分类[104]、[105]，语义分割[106]，物体检测[107])中取得了巨大的成功，由于训练数据的数量和种类有限，它们容易在FAS任务中遭受到过拟合的问题。由于手工特征(LBP描述子[108]，HOG描述子[109]，图像质量[110]，光流运动[111]，和rPPG线索[112])已经被证明具有将真实和PA区分开来的判别能力，一些近期的混合works针对FAS把手工特征和深度特征组合起来。(附录)表-A5总结了这些混合方法的典型属性。

一些FAS方法首先从人脸输入中提取手工特征，然后使用CNN做语义特征表征(范例见图7(a))。一方面，基于颜色纹理的静态特征从每一帧中提取出来，随后被送入深度模型。基于丰富的low-level纹理特征，深度模型可以挖掘纹理感知的语义线索。为此，Cai和Chen[113]采用了多尺度颜色LBP特征作为局部纹理描述子，随后串联一个随机森林用于语义表征。类似的，Khammari[22]提取了LBP和Weber局部描述子编码的CNN特征，它们可以联合使用来保留局部强度和边缘方向信息。然而，与原始人脸输入相比，基于局部描述子的特征丢失了像素级别的细节，因此限制了模型的表现。另一方面，跨越时间的动态特征(如，运动，光照变化，生理信号)也是有效的CNN输入。Feng et al.[114]提出从提取的基于稠密光流的运动中，训练一个多层感知器，它可以揭示打印攻击的异常。此外，Yu et al.[115]从人脸视频中构建了空间－时间rPPG图，并使用了一个视觉transformer来捕捉活体样本里周期性的心跳活体特征。然而，头部运动和rPPG信号在重放攻击中容易被模仿，使得这类动态线索不可靠。重放攻击通常存在异常的反射变化，基于这个事实，使用来自反射率图像的强度差异直方图作为输入，Li et al.[116]提出使用一个1D CNN来捕捉这类光照变化。

其它一些混合FAS方法从深度卷积特征中提取手工特征，它们跟随图7(b)所示的混合框架。为了降低与FAS无关的冗余，Li et al.[30]利用分块主成分分析(Block PCA)从预训练的VGG-face模型中过滤掉了与深度无关的特征。除了基于PCA的降维之外，Agarwal et al.[117]从浅卷积特征中显式提取了彩色LBP描述子，它包含了丰富的low-level统计信息。除了静态欺骗样式，一些works也在训练好的深度模型中探索手工动态时间线索。Asim et al.[20]和Shao et al.[118]分别使用LBP-TOP[119]和光流各自从序列卷积特征中提取深度动态纹理和运动特征。这种混合框架的一个限制是，手工特征高度依赖于训练好的卷积特征，并且依然不清楚，对于不同种类的手工特征，浅卷积特征或者深度卷积特征哪个更合适。

由于手工特征和深度卷积特征持有不同的属性，另一种受欢迎的混合框架(见图7(c))将它们融合起来以用于更通用的表征。为了提供更加可靠的预测，Sharifi[120]提出融合来自手工LBP特征和深度VGG16模型的预测分数。然而，为这两种特征确定最优的分数权重是具有挑战性的。除了分数级别的融合，Rehmana et al.[21], [121]提出利用HOG和LBP图来扰动和调节low-level卷积特征。尽管事实是来自手工特征的局部先验知识可以提升判别能力，但总的模型会遭受到语义表征降级的问题。根据时间方法，为了充分利用真实与PA之间的动态差异，Li et al.[122]通过1D CNN提取了强度变化特征，它与来自运动放大人脸视频的运动模糊特征融合，用于重放攻击检测。

总之，受益于显式专家设计的特征提取，混合方法可以表示特殊的非纹理线索(如，时间的rPPG和运动模糊)，通过端到端的基于纹理的FAS模型则很难捕捉到这些。然而，缺点也很明显：1) 手工特征高度依赖于专家知识并且不可学习，一旦有足够的训练数据可用，这种方法会变得效率低下；2) 手工特征和深度特征之间可能存在特征间隙/不兼容，它会导致性能饱和。

### 3.2 传统深度学习方法

得益于高级CNN架构的发展[105]、[123]，正则化技术[124]、[125]以及近年来发布的大规模FAS数据集[44]、[59]、[77]，端到端的深度学习方法越来越受到关注，并且主导了FAS领域。混合方法集成了部分手工特征，并且不需要学习参数，与此不同，基于传统深度学习的方法直接学习到从人脸输入到欺骗检测之间的映射函数。传统深度学习框架通常包括：1) 使用二元交叉熵损失函数的直接监督(见图8(a))；2) 使用辅助任务的像素级监督(见图8(b))或使用生成模型的像素级监督(见图8(c))。

#### 3.2.1 使用二元交叉熵损失函数的直接监督

由于FAS直觉上可以看做二分类(真实 vs. PA)任务，许多端到端的深度学习方法使用二元交叉熵(CE)损失函数以及扩展的损失函数(如，三元损失函数[126])来直接进行监督学习。(附录)表-A6总结了这些方法。

一方面，研究人员针对FAS提出了通过二元交叉熵损失函数监督学习的不同网络架构。Yang et al.[29]使用8层浅CNN用于特征表征，提出了第一个端到端的深度FAS方法。然而，由于数据集有限的规模和多样性，基于CNN的方法容易在FAS任务中产生过拟合。为了缓解这个问题，一些works[127]、[128]、[129]针对FAS，对ImageNet预训练模型(如，VGG16, ResNet18和vision transformer)进行了微调。对于移动级别的FAS应用，Heusch et al.[55]采用了轻量级的MobileNetV2[130]来进行更高效FAS。上述提到的通用骨干网络通常关注high-level语义表征，忽略了low-level的特征，而这些特征对于欺骗方式的挖掘也很重要。为了更好的使用多尺度的特征，Deb和Jain[131]提出了使用浅全连接卷积网络(FCN)以自监督方式从人脸图像中学习局部判别线索。在基于单帧的外观特征之外，一些works[25]、[132]、[133]、[134]考虑真实样本和PA在时间上的差异性，并且使用LSTM[135]串联了多帧CNN特征以实现更健壮的动态线索传播。

另一方面，考虑到二元交叉熵的类内和类间限制较弱，一些works通过修改二元交叉熵损失函数来为CNN提供更具有判别性的监督信号。取代二元限制，Xu et al.[100]将FAS重新表达为一个细粒度的分类问题，并且将类型标签(如，真实，打印攻击，重放攻击)用于多类别监督。用这种方法，PA的一些特殊的属性可以被表示(如，材质)。然而，尤其在比较困难的活体/欺骗样本中，使用多类别交叉熵损失函数监督训练的模型，依然存在混乱的活体/欺骗分布。例如，高保真的PAs与对应的真实样本具有相似的外观线索。一方面，为了学习到类内间距小，类间间距大的紧致空间，Hao[136]和Almeida[80]分别各自引入了对比损失函数和三元损失函数。然而，与视觉检索任务不同，FAS任务中的真实与欺骗通常具有不对称的类内分布(更加紧致，更加多样性)。基于此，Wang et al.[101]提出通过非对称angular-margin softmax损失函数来放松类内PA之间的限制，来监督FAS补丁模型。另一方面，为了在困难样本上提供更加自信的预测，Chen et al.[137]采用二元焦点损失函数来指导模型放大活体/欺骗样本之间的边缘，并且在困难样本上实现了很强的判别性能。

总的来说，两类方法便于使用，且模型可以快速收敛。然而，对于活体/欺骗嵌入学习来说，这些监督信号只提供全局性(空间/时间)的限制，可能导致FAS模型对于unfaithful patterns容易过拟合。此外，二元监督的FAS模型通常是黑盒，并且学习到的特性也很难理解。

#### 3.2.2 像素级监督

直接通过二元损失函数监督的深度模型容易学习到unfaithful patterns(如，屏幕边框)。相反，对于更好的内在特征学习，像素级监督可以提供更加细粒度的和任务相关的上下文线索。一方面，根据物理线索和判别设计原理，辅助监督信号，如伪深度标签[13]、[26], 二元面具标签[32]、[38]、[39], 和反射图[24]、[36]，被用来做局部的活体/欺骗线索描述。另一方面，带有显式像素级监督和生成模型(如，原始人脸输入重建[42]、[138])最近被用于通用欺骗样式的估计。我们在(附录)表-A7中总结了代表性的像素级监督方法。

##### 使用辅助任务的像素级监督

根据人类FAS先验知识，很多PA(如普通的打印纸和电子屏幕)仅仅没有真实的深度信息，这可以被用作判别监督信号。因此，一些近期的works[23]、[26]、[96]、[139]采用像素级伪深度标签来指导深度模型，强制模型对活体样本预测真实的深度，而将欺骗样本映射为0。Atoum et al.[26]第一个使用伪深度标签来指导多尺度FCN(即简称DepthNet)。因此，训练好的DepthNet可以预测整体深度图作为决策根据。为了进一步提高细粒度内在特征表征能力，Yu et al.[23]使用中心差分卷积(CDC)替换了DepthNet中的普通卷积，从而形成了CDCN架构(详细结构见图9)。在静态架构方面，由于紧致性和优秀表现，DepthNet和CDCN是最受欢迎的，并且广泛应用在深度FAS社区。许多近期的变种[37]、[98]、[140]基于DepthNet/CDCN构建。至于时间架构，经典的FAS-SGTD[96]因它对短期和长期微运动优秀的估计而出名，它可以用于精准的脸部深度预测。FAS-SGTD的详细架构见图10，它是在一个transformer对应的架构[141]上后边经过修改和扩展而来。

为每一个训练样本合成3D形状标签成本高且不够精确，并且对于具有深度信息的PA类型(如，3D面具和人体模型)来说也缺乏合理性。相反，二元面具标签[32]、[38]、[99]、[142]、[143]更容易生成，且更容易泛化到所有PA。特别的，对于每个空间位置上，二元监督可以为深度内嵌特征提供。换句话说，通过二元面具标签，可以在对应的补丁里发现是否有PA出现，即它是攻击类型不可知，并且空间可解释的。George和Marcel[32]是第一个引入深度像素级二元监督，用来为串联的最终二元分类来预测中间的置信图。由于充足的像素级监督，骨干网络DenseNet121收敛良好，并且可以提供补丁级别的活体/欺骗预测。由于细微的欺骗线索(如，莫尔条纹)通常存在于不同的空间区域，具有不同的强度，普通的像素级二元监督把所有的补丁看成具有相等的贡献，这可能会导致有偏向的特征表征。为了解决这个问题，Hossaind et al.[142]提出在计算深度像素级二元损失之前，为特征改进增加一个可学习的注意力模块，它对显著的信息传播有用。虽然灵活且易于使用，但是当前的二元面具标签通常假定脸部区域的所有像素具有相同的活体/欺骗分布，因此分别为真实或者PA生成全部1或者全部0的映射。然而，当遇到部分攻击时(如，funny eyes)，这些标签不精确且有噪声。

除了主流的深度图和二元面具标签，还有几个信息量大的辅助监督(如，伪反射图[24]、[36]、[44]，3D点云图[40]，三元图[39]，和傅里叶光谱[144])。根据活体皮肤和欺骗媒介反射率在脸部材质相关的反射率不同，Kim et al.[36]提出同时使用深度和反射标签，来监督深度模型。此外，为了进一步提升类型不可知的泛化能力，有人引入了二元面具图[24]，同时使用这三个像素级监督来训练双边卷积网络。不像二元面具标签会考虑所有空间位置，包括与活体/欺骗无关的背景，Sun et al.[39]移除了脸部无关的部分，留下整个脸部区域作为一个改进的二元面具，叫做“三元图”，它消除了脸部之外的噪音，并且对脸部欺骗线索挖掘有用。基于真实与PA之间丰富的纹理和几何差异，使用来自傅里叶图[33]、[144]，LBP纹理图[97]，和稀疏3D点云图[40]的其它辅助监督的深度模型也显示出它们优秀的表征能力。

总的来说，像素级辅助监督有益于物理意义，可解释性的活体/欺骗特征学习(如，反射和深度监督分别对应于材质与几何表征)。此外，以一个多任务学习的方式[24]，通过综合使用多个补充的辅助像素级监督(如，深度，反射，反射率)，可以训练出可靠的且泛化性好的FAS模型。然而，需要提到辅助监督的两个限制：1) 依赖于高质量(如，高分辨率)的训练数据，当训练数据太noisy并且质量较低时，难以提供有效的监督信号；2) 伪辅助标签是人工设计或者是其它现成算法生成的，并不总是可靠。

##### 使用生成模型的像素级监督

虽然有辅助任务的细粒度的监督信号，深度黑盒模型是否真的表示内在的FAS特征，依然很难理解。最近，一个热门的趋势是，挖掘存在于欺骗样本中的视觉欺骗样式，旨在为样本的spoofness提供一个更加直观的解释。我们在(附录)表-A7的底部总结了这种使用生成模型的像素级监督方法。考虑到辅助的像素级监督强烈的受物理影响的限制，一些works将这种显式的监督信号放松，并且提供一个更宽的空间用来挖掘隐式欺骗线索。Jourabloo et al.[33]将FAS重新表达成一个欺骗噪声建模问题，并且设计了一个编码器-解码器架构，使用放松的像素级监督(如，活体人脸是零噪音图)来估计底层欺骗样式。使用这种针对真实样本单方面的限制，模型可以灵活挖掘PA的欺骗线索。类似的，Feng et al.[41]设计了一个欺骗线索生成器来最小化活体样本的欺骗线索，同时对于欺骗样本不加任何显式限制。与上面提到的对活体样本强行加上严格的限制的works不同，Mohammadi et al.[138]使用了一个重建错误图来进行欺骗检测，它通过一个活体人脸预训练的自动编码器计算得来。由于这种错误图从重建活体人脸的残留噪声中生成，没有人工定义的元素，它们在知识线索改变的域变换下是健壮的。然而，由自动编码器而来的低质量的重建人脸，可能会导致有噪音的残差错误图。

除了直接的欺骗样式生成之外，Qin et al.[43]提出通过一个meta-teacher框架，自动生成像素级标签，它可以为学生FAS模型提供更合适的监督，以学习足够的和内在的欺骗线索。然而，在其工作当中，只有可学习的欺骗监督[43]生成出来。因此，如何为活体样本和欺骗样本自动生成最优的像素级信号依然值得探索。

总的来说，使用生成模型的像素级监督通常会放宽专家设计的硬限制(如，辅助任务)，使得解码器重建更加自然的欺骗相关的线索。因此，预测的欺骗样式具有更强的数据驱动性和良好的可解释性。生成的欺骗样式是可见地有洞察力的，并且用人类先验知识来人工描述是很有挑战性的。然而，这种软的像素级监督在未知干扰(如，传感器噪音)容易陷入局部最优且过拟合，在真实世界场景下会导致泛化糟糕。将显式辅助监督和基于监督的生成模型联合训练，可能会缓解这个问题。

### 3.3 泛化深度学习方法

传统的基于深度学习的端到端FAS方法在未知主导的情况（如，光照，相机质量，脸部可见性）中和未知攻击类型（如，使用新材质制造的高保真面具）中的泛化能力较差。因此，在有高安全需求的实际应用中，这些方法并不可靠。有鉴于此，越来越多的研究者聚焦在提升深度FAS模型的泛化能力上。一方面，在无限的域变化下，域适应和域泛化技术被用于健壮的活体/欺骗分类。另一方面，零样本/少样本学习和异常检测被用于未知人脸PA类型检测。在这篇文章中，未见过的域指与欺骗无关的外部变化(如，光照和传感器噪声)，但是实际上会影响外观质量。相反，未知欺骗攻击通常指具有内在物理属性的新的攻击类型（如，材质和几何），它们并不会在训练阶段出现。(附录)表-A8和表-A9分别总结了在未见过的域和未知攻击类型上具有代表性的泛化深度FAS方法。

#### 3.3.1 泛化到未见过的域

如图11所示，在源域和目标域之间，存在着严重的域变换，当直接在源数据集上(如，OULU-NPU，CASIA-MFSD，和Replay-Attack)训练深度模型时，在偏向目标数据集(如，MSU-MFSD)上容易导致性能低下。域适应技术利用目标域的知识缩小源域与目标域之间的差异。相反，域泛化帮助FAS模型直接从多个源域中学习泛化特征表征，而不需要访问任何目标数据，更适用于在现实世界部署。

##### 域适应

域适应技术缓解了源域和目标域之间差异。在一个学习到特征空间中，源特征和目标特征通常是一致的。如果特征具有相似的分布，在源样本的特征上训练的分类器可以用于目标样本的分类。

为了使源域数据和目标域数据的特征空间一致，Li et al.[78]提出用无监督域适应方法来学习映射函数，通过最小化最大均值差异(MMD) [145]，对齐源域-目标域内嵌子空间。为进一步增强在两个域上的泛化性能，带自监督对抗域适应的UDA-Net[146]、[147]被提出，用来联合优化源域和目标域的编码器。当特征不能区分来自哪个域时，就学习到了域感知的共同特征空间。因为无法访问目标域的标签信息，通过MMD和对抗域适应约束的域不变特征，其判别能力依然较弱。为了缓解这个问题，两个works[103]、[148]将半监督学习引入到了域适应，即可以使用目标域少量有标签数据和大量无标签数据。Jia et al.[103]提出一种统一的无监督和半监督域适应网络，用来表示域不变特征空间，并且发现使用少量带标签的目标域数据(三到五个)可以显著提升其在目标域上的性能。类似的，Quan et al.[148]提出一种半监督FAS学习方法，它仅仅使用少量带标签的训练数据用于预训练，在训练过程中，逐步使用可靠的无标签数据，以减小域间隙。尽管这些半监督的方法具有优秀的适应能力，但是严重依赖于类别均衡的少样本带标签数据(如，同时具有真实和欺骗样本)。当带标签的欺骗样本不可用时，性能会显著下降。

与上边提到的只适应最终分类层的方法不同，有一些works设计和适应整个FAS网络。由于不同的deep layers共享不同粒度的域信息，[149]的作者考虑使用MMD损失在表现层和分类层进行多层分布适应。除了由多层线索带来的有效适应能力，该架构可能是冗余的，且本身的泛化能力有限。为了获得更加泛化的架构，Mohammadi et al.[150]提出裁剪具有高度特征差异的过滤器，这些过滤器在不同数据集之间泛化较差，因此网络在目标域的性能会得到提升。与对特定的过滤器/层网络裁剪不同，Li et al.[151]提出从训练良好的teacher网络中，为特定的应用域提取整个FAS模型。它通过特征MMD和来自两个域的双向相似度内嵌进行正则化。通过这种方式，可以发现轻量的且泛化的FAS模型，但是与teacher FAS网络相比，判别能力较弱。

虽然域适应通过使用无标签的目标数据帮助减小源域和目标域的分布差异，但是在真实应用场景中，收集很多无标签的目标数据用于训练，比较困难且成本高(尤其是欺骗攻击)。此外，考虑到隐私问题，当在目标域上部署FAS模型时，源人脸数据通常无法访问。

##### 域泛化

域泛化假定多个可见的源域与不可见但相关的目标域之下，存在一个泛化的特征空间。从可见源域学习到的模型，可以在不可见域上泛化良好。

一方面，一些works采用域感知的生成限制来学习域无关的判别特征。他们假定该类特征包含跨越所有可见域的本质线索，因此可以在不可见域上泛化良好。Shao et al.[48]第一个提出，通过多对抗判别域泛化框架学习多个源域共享的泛化特征空间。同时，也建立了跨4个数据集的域泛化基准[48]。但是，有两个限制：1) 这类域独立的特征可能依然包含欺骗无关的线索(如，subject ID和传感器噪音)；2) 域泛化空间的判别能力依然不能令人满意。为了改善第一个限制，Wang et al.[50]提出从主判别和域相关的特征中分解泛化的FAS特征。对于第二个限制，考虑到不同域中欺骗人脸之间较大的分布差异，Jia et al.[51]提出学习一个判别泛化特征空间，在该空间中，真实人脸的特征分布是紧凑的，而PA的特征分布在域之间是分散的，但在每个域内又是紧凑的。

另一方面，一些代表性的works使用域感知的元学习来学习域泛化特征空间。特别的，来自部分人脸源域的数据作为支持集，剩下的未遮挡的域作为支持集。基于上述设置，Shao et al.[49]提出在细粒度的域感知元学习过程中通过查找泛化学习方向对FAS模型进行正则化。交替地强制元学习器在支持集上表现良好，学习到的模型具有健壮的泛化能力。然而，这种域感知的元学习严格要求源数据的域标签构建查询集和支持集，但在真实世界中，域标签并不总是可用。不使用域标签，Chen et al.[152]提出使用域动态调整元学习来训练一个泛化的FAS模型，它可以迭代的将混合域划分为具有伪域标签的组。然而，通过一个简单的通道注意力模块将欺骗判别特征与域感知特征分解开，使得域特征在伪域标签分配上不可靠。从特征归一化的角度看，基于实例归一化可以有效移除域差异这一证据，Liu et al.[153]提出了通过元学习对泛化表示适应性的聚合批处理和实例归一化。注意，批处理和实例归一化适应的折衷可能会减弱活体/欺骗的判别能力。

总的来说，最近三年来，FAS域泛化是一个新的热点。一些潜在的和令人激动的趋势，如将域泛化和域适应[154]组合起来，和无域标签[152]学习正在研究。然而，依然缺少works，可以揭开关于判别和泛化能力的面纱。换句话说，域泛化对FAS模型在不可见域表现良好有帮助。但是，在可见场景下的欺骗检测是否被恶化，依然是未知的。

#### 3.3.1 泛化到未知攻击类型

除了域变换问题，在真实世界的实际应用中，FAS模型对于新出现的PA是脆弱的。大多数先前的深度学习方法将FAS描述为一个探测多种预定义PA的闭集问题，它需要大规模的训练数据来覆盖尽量多的攻击类型。然而，训练的模型容易过拟合几种常见的攻击(如，打印和重放)，并且对于未知的攻击类型依然脆弱(如，面具和化妆)。最近，许多研究致力于开发泛化FAS模型，用于未知欺骗攻击检测。一边，零样本/少样本学习被用来提升新欺骗类型检测，而它仅需要少量或者甚至不需要目标攻击类型的样本。另一边，FAS也可以被看做是一类分类任务，在这种情况下，真实样本紧密地聚集在一起，而异常检测用于检测分布外PA样本。

##### 零样本/少样本学习

对于新的攻击检测的一种直接的方式是使用新攻击的足够的样本来对FAS模型进行微调。然而，由于欺骗持续演变，为每一种新的攻击类型收集带标签样本昂贵且耗时。为了克服这个挑战，一些works[38]、[53]、[155]提出将FAS看做一个开集的零样本/少样本学习问题。零样本旨在从预定义的PA中，学习泛化特征和判别特征用于未知的新的PA检测。少样本学习，通过从预定义的PA和收集到的新的攻击的少量样本学习，旨在快速将FAS模型适应于新的攻击类型。

不使用未知欺骗攻击的任何先验知识，Liu et al.[38]设计了一个深度树网络(DTN)以无监督的方式来学习预定义攻击的语义属性并且把欺骗样本分成语义小组。基于内嵌特征的相似性，DTN适应性地将已知或未知PA路由到对应的欺骗组里。活体/欺骗的树形拓扑通过DTN自动构建，与人工定义的类别关系相比，这种方式更加语义化和泛化。然而，不使用任何未知攻击的先验知识，零样本DTN在检测新型高保真攻击时容易失败。为了缓解这一问题，两个works采用了一个开集少样本框架引入部分但有效的未知攻击知识，用于表征学习。Qin et al.[53]通过融合将零样本和少样本FAS任务统一起来，使用一种自适应的内更新学习率策略训练一个元学习器。在零样本和少样本任务上同时训练元学习器，增强了FAS模型从预定义PA和少量新PA实例的判别能力和泛化能力。然而，在新攻击类型上直接使用少样本元学习容易遭受对预定义PA灾难性的遗忘。为了解决这一问题，Perez-Cabo et al.[155]提出了一个持续的少样本学习范式，它可以从持续的数据流当中增量式的扩展获得的知识，并且可以通过一种元学习方案使用少量训练样本来检测出新的PA。

虽然少样本学习可以帮助FAS模型检测未知攻击，但是当目标攻击类型的数据不能用于适配时(如，零样本情况)，性能下降明显。我们发现，失败的检测通常发生在有挑战性的攻击类型上(如透明面具，funny eye，和化妆)，它们和真实样本共同拥有相似的外观分布。

##### 异常检测

基于异常检测的FAS假设活体样本在一个正常的类别，因为它们共同具有更多相似相似的和紧致的特征表示，而欺骗样本由于攻击类型和材质的高度不同，它的特征在异常样本空间中具有较大的分布差异。基于上述假设，异常加测通常首先训练一个单类别分类器将活体样本精确地聚集在一起。然后，任何在活体样本簇边缘外的样本(如，未知攻击)将被检测为攻击。

Arashloo et al.[52]第一个提出在交叉类型测试协议上评估单类别异常检测和传统的二分类FAS系统。他们发现使用单类别SVM的基于异常的方法并不比使用两类别SVM的二分类方法差。为了更好的表示真实样本的概率分布，Nikisins et al.[156]提出用高斯混合模型(GMM)作为异常检测器替换传统的单类别SVM。除了单类别SVM和GMM，Xiong and AbdAlmageed[157]也考虑带有LBP特征提取器的基于离群检测器的自动编码器来处理开集未知PAD。上述提到的works使用单类别分类器将特征提取分离，它使得真实表征学习具有挑战性并且次优。相反，Baweja et al.[158]提出一种端到端的异常检测方法用来同时训练单类别分类器和特征表征。此外，为了学习健壮的真实表征以应对分布外的小变化，他们生成伪负特征以模仿PA类别，且强制单类别分类器对PAD具有判别性。然而，生成的伪PA特征不能表示多种多样的真实世界的特征，使得单类别异常检测系统在真实世界部署时并不可靠。

虽然合理，只使用活体人脸来训练分类器通常会限制异常检测模型在新PA类型上的泛化能力。取代只使用活体样本，一些works通过度量学习，同时使用活体和欺骗样本来训练泛化的异常检测系统。Pe ́rez-Cabo et al.[159]提出通过一个三元焦点损失函数正则化FAS模型来学习判别特征表征，然后引入一个少样本后概率估计作为异常检测器来检测未知PA。类似的，George和Marcel[160]设计了一个成对单类别对比损失函数(OCCL)来强制网络为真实类别学习一个紧致的内嵌空间，而远离攻击的表征。然后串联一个单类别GMM以检测未知PA。虽然可以通过三元或者损失函数学习到判别内嵌，works[159]、[160]在内嵌特征之后，依然需要串联额外的异常检测器(如，单类别GMM)，它影响端到端的表征学习。相反，为了保持类内活体紧致，同时类间活体-欺骗分离，Li et al.[161]提出使用一个新的超球面损失函数来监督深度FAS模型。无需额外的异常检测分类器，在学习到的特征空间中可以直接检测未知攻击。一个限制就是，预测的活体/欺骗分数是从内嵌特征的L2范数的平方计算得来的，很难选择一个合适的预定义阈值，用来检测不同种类的攻击类型。

尽管对于未知攻击检测具有令人满意的泛化能力，与真实世界里的开集场景下的传统活体/欺骗分类(如，既有已知又有未知攻击)相比，基于异常检测的FAS方法将会遭受到判别能力降级。

## 4. 使用高级传感器的深度FAS

在日常人脸识别应用中，从安全性和硬件成本的角度来看，基于商用RGB相机的FAS是一个绝佳的权衡方案。然而，一些高安全性的场景(人脸支付和保险库门禁)要求非常低的误接受错误。最近，具有多种模态的高级传感器被开发出来用于推动超级安全的FAS。表2从环境情况(光照和距离)和攻击类型(打印，重放，和3D面具)角度列出了针对FAS的不同传感器和硬件模块的优缺点。

与单目可见RGB相机(VIS)相比，立体相机(VIS-Stereo) [162]有利于为2D欺骗检测进行3D几何信息重建。当在呈现人脸上与动态闪光灯组装在一起时，VIS-Flash [163]能够捕捉内在的基于反射的材质线索以检测所有这三种攻击类型。

除了可见的RGB模态，深度和NIR模态也以可接受的成本被广泛用于实际的FAS部署。两种深度传感器包括时间飞行(TOF) [164]和3D结构光(SL) [165]已经被内嵌在主流的手机平台(如，iPhone, 三星，OPPO，华为)。它们为2D欺骗检测提供所捕捉人脸精准的3D深度分布。与结构光相比，TOF对于环境情况更加健壮，如光照和距离。相反，NIR模态[166]是除VIS之外一个辅助的光谱(900-1800nm)，它可以有效利用活体与欺骗人脸之间的反射区别，但是在长距离时图像质量较差。此外，对于许多访问控制系统来说，VIS-NIR集成硬件模块具有较高的性价比。

同时，几个小众但是有效的传感器也被引入到FAS当中。在人脸图像上，通过测量吸水率，波长为940-1450nm的短波红外(SWIR) [55]从非皮肤像素中区分活体皮肤材质。通过人脸温度预估，一个热感相机是另外一种有效的针对FAS的传感器。然而，当受试者戴上透明面具之后，它表现地很差。根据它们各自对于脸部深度与反射/折射光绝佳的表征，昂贵的光场相机[87]和四向偏振传感器[47]也被探索用于FAS。

### 4.1 基于专用传感器的单模态深度学习

基于专用的传感器/硬件用于不同的成像，研究人员已经开发出传感器感知的深度学习方法用于有效的FAS，(附录)表-A10中总结了这些方法。Seo和Chung[167]提出一个轻量级的热感人脸-CNN用来从热感图像中估计人脸温度，并且用异常温度来探测欺骗(如，36-37度之外的范围)。它们发现热感图像比RGB图像更适合于重放攻击检测。然而，这种基于热感的方法对于透明面具攻击很脆弱。根据基于立体的FAS，几个works[162]、[168]、[169]证明了通过CNN利用来自立体输入(从立体和双像素(DP)传感器)的预估差异或深度/法线图，在2D打印和重放攻击检测中可以取得引人注目的表现。然而，它通常在具有相似活体人脸几何分布的的3D面具攻击上表现较差。为了进一步捕捉到详细的3D局部样式，Liu et al.[87]提出从单点光场相机中提取光线微分和微透镜图像，然后一个浅CNN用于人脸PAD。由于光场成像中丰富的3D信息，该方法可以潜在的用于对细粒度的欺骗类型进行分类。对于实时和移动级别的部署，Tian et al.[47]提出使用轻量级的MobileNetV2从一个芯片内的集成偏振成像传感器提取有效的DOLP特征。上面提到的方法致力于解决特定的PA类型(如，重放和打印)，它们不能在所有的PA类型上泛化良好。相反，Heusch et al.[55]提出使用一个多通道CNN用于在选定的SWIR-difference输入当中进行深度材质相关的特征提取，它可以几乎完美的检测所有伪装攻击，同时确保较低的真实分类错误。

除了使用专用的硬件，如红外点投影仪和专用相机，基于可见相机和额外的环境闪光灯的一些深度FAS方法也被开发出来。在[163]和[170]中，来自智能手机屏幕的动态闪光灯被用于从多个方向照亮用户的脸部，它通过光度立体技术使得可以恢复人脸表面法线。这种动态法线提示随后被喂给CNN，以预测脸部深度和光验证码用于PA检测。类似的，Ebihara et al.[171]设计了一个新型描述子使用带有和不带有闪光灯的不同提示，表征镜面反射和漫反射，它超过了端到端的连接闪光灯输入的ResNet。这些方法易于部署且不需要额外的硬件集成，并且已经被用于移动验证和支付系统，如支付宝和微信支付。然而，动态闪光灯在户外环境比较敏感，并且由于较长的连续激活时间对用户并不友好。

### 4.2 多模态深度学习

同时，基于多模态学习的方法在FAS研究社区变得热门和活跃。(附录)表-A11收集了具有代表性的多模态融合和跨模态翻译方法。

#### 多模态融合

随着多模态的输入，主流FAS使用特征层级融合策略来提取辅助的多模态特征。由于跨多模态特征中存在冗余，直接的特征串联容易导致高维特征和过拟合。为了缓解这个问题，Zhang et al.[28]提出SD-Net使用特征再权重机制在RGB, 深度, 红外模态中选择信息量大的特征，丢弃冗余的通道特征。然而，再权重融合在SD-Net中只能对high-level特征进行，忽视了多模态low-level线索。为了进一步推动多模态特征在不同层级间的互动，[172]、[173]的作者引入了多模态多层级融合分支，用于加强不同模态间的上下文线索。除了高级的融合策略，多模态融合容易被部分模态所支配(如，深度)，因此，当这些模态有噪音或缺失时，表现很差。为了解决这个问题，Shen et al.[174]设计了一个模态特征擦除操作用来随机dropout部分模态特征，以避免模态感知的过拟合。此外，George and Marcel[175]提出跨模态焦点损失函数用来调节每个模态的损失贡献，它帮助模型在不同模态之间学习辅助信息。总的来说，特征层级融合对于多模态线索聚合灵活并且有效。然而，从不同的分支提取模态特征通常计算成本很高。

除特征层级融合之外，一些works考虑输入层级融合和决策层级融合。输入层级融合假定多模态输入在空间上已经对齐，并且可以在通道维度上直接融合。在[176]中，将复合图像喂给深度PA检测器，而该复合图像通过堆叠归一化的图像将灰度, 深度, 红外模态融合起来。类似的，Liu et al.[177]通过不同的操作符(如，堆叠，求和，差值)复合了VIS-NIR输入，并且所有的人脸融合图像被一个多模态FAS网络转发用于活体/欺骗预测。这些输入层级融合方法效率高且只有少量额外计算成本(大部分在融合操作符和第一个网络层)。然而，太早的融合容易导致多模态线索在随后的mid-level和high-level空间消失。相反，为了在单独的模态偏向与做出可靠的二元决定之间权衡，一些works采取了决策层级融合，它基于从每个模态分支的预测分数。一方面，Yu et al.[27]直接将来自RGB, 深度, 红外模态的单独的模型的预测二元分数平均，在CeFA[91]数据集上的表现超过了输入层级和特征层级融合。另一方面，Zhang et al.[178]设计了一个决策层级融合策略，首先，它使用深度模态，将多个模型的分数聚合，然后与来自红外模态的分数聚合，用于最终的活体/欺骗分类。尽管预测可靠，决策层级融合效率低下，因为对于特定的模态，它需要单独训练好的模型。

#### 跨模态翻译

多模态FAS系统需要额外的传感器来成像具有不同模态数据的人脸输入。然而，在一些传统场景下，只有部分模态(如，RGB)可用。为了解决这个在推理阶段模态缺失的问题，少量works采用了跨模态翻译技术，为多模态FAS生成缺失的模态数据。为了从RGB人脸图像中生成对应的红外图像，Jiang et al.[179]首先提出了一个新的多类别(活体/欺骗，真实/合成)图像翻译循环生成对抗网络。基于生成的红外和原始的RGB输入，与只使用RGB图像相比，该方法可以提取出更健壮的融合特征。然而，从原始循环生成对抗网络生成的红外图像质量较低，它限制了融合特征的表现。为了生成高保真的目标红外模态数据，Liu et al.[180]在跨模态翻译的框架下，设计了一种新的基于子空间的模态正则化。除了生成红外图像，Mallat and Dugelay[181]提出了使用可见-热图转换方案，它使用一个级联的refinement网络从RGB人脸图像合成热攻击。虽然在数据集内测试有效，对这些方法的一个主要顾虑就是，域变换和未知的攻击可能会显著地影响生成模态的质量，并且使用成对的有噪音的模态数据会使得融合特征不可靠。

虽然，自2019年以来有一个上升的趋势，与基于RGB的单模态方法相比，基于传感器的多模态FAS进展依然缓慢。值得注意的是，多模态方法也存在于使用商用RGB相机的深度FAS中。例如，有人已经探索了基于两个RGB视频模态(如，远程生理信号和脸部可见图像)的决策层级融合。因此，有效的融合这类来自商用相机的自然模态和来自高级传感器的模态将会是一个有趣且有价值的方向。同时，一些高级的传感器(如，短波红外，光场和极化)价格昂贵并且在真实世界中部署不可移植。应该探索更多有效的FAS专用传感器和多模态方法。
