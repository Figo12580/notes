## DASH 相关

[TOC]

### 文献阅读

#### 背景

- RNA-seq中的高丰度冗余信号（例如rRNA，体液微生物，或者肿瘤组织中的正常基因），限制了检测的成本和灵敏度，这在cf核酸检测中尤其突出

  > 使用其他富集方法的可能？为什么尤其突出？

- CRISPR/Cas System: sgRNA, PAM site, double strand DNA break, off-target

- 现存富集方法：

  - pull-down, ??,castPCR, 限制酶切：只能针对我们已知的库复现，不可以用来探索未知库的基因。
  - 硬计数方法，比如dPCR，高通量平行（multiplex）能力不好，即使加入barcode以后成本也相当高。
  - 序列特异性RNA降解方法：Ribo-Zero，GLOBINclear等基于磁珠和寡聚序列的方法；NEBNext等基于RNase H的方法。这些都是针对RNA的方法，最好在建库前就应用，因此对样本量需求的比较高（10ng-1μg）

#### 实验

> 基本思路：在细胞系、病人样本、癌组织样本模拟建库过程中（转座酶切割后，加接头和扩增前），使用体外CRISPR系统切割DNA，使得靶向DNA清除而非靶向DNA保留。由于文库需要有两端完整adaptor才可以测序等，所以有一个切口即可。
>
> 它将可以用于成熟的测序库、质粒、噬菌体等任何DNA库？

1. 在HeLa细胞和病原体感染的脑脊液两种样本中，分别应用DASH除去线粒体rRNA（数倍于其他RNA）。sgRNA设计自标准图谱(coverage plot)。

   > 使用产脓链球菌Cas 9, 对应的PAM site是NGG，与切割位点的距离是3 nt 。sgRNA识别长度20 nt。

   效率估算：假设样本中90%都是目标rRNA，Cas9是one turnover enzyme，在CSF中使用了100倍的cas9和1000倍的sgRNA（摩尔浓度）；在HeLa Cell中由于rRNA占比略低，用了150和1500倍。重复3次

   > 细胞系RNA 1ng/10μL，CSF 5倍于此；到底为什么cf的rRNA含量更高？

   结果：在HeLa细胞样本中，rRNA含量（去重复后；什么是含量？）从61%降低到3%左右，fpkm80-120倍；有Cas 9 和sgRNA的剂量依赖效应。



