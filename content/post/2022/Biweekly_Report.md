---
title: "BiWeekly Report 03-02-2023"
description: 
date: 2023-01-31T16:58:12+08:00
hidden: false
comments: true
draft: true

---

## Brief Summary

### Tasks

Literature review:

- Cross-Domain Adaptive Clustering for Semi-Supervised Domain Adaptation [CVPR 2021, https://github.com/lijichang/CVPR2021-SSDA]
- AdaMatch: A Unified Approach to Semi-supervised Learning and Domain Adaptation [ICLR 2022, https://github.com/yizhe-ang/AdaMatch-PyTorch]

### Results

#### CDAC

- Introduce an adversarial adaptive clustering loss to perform cross-domain **cluster-wise** feature alignment so as to solve the SSDA problem. 
- Integrate an adapted version of **pseudo labeling** to enhance the robustness and power of cluster cores in the target domain to facilitate adversarial learning.

![img](https://cdn.nlark.com/yuque/0/2023/png/21656683/1675154995393-c827f6c1-a12e-43be-ab9e-593cb308867a.png)

#### AdaMatch

- Problem to be solved：Extend **semi-supervised** learning to the problem of **domain adaptation**. AdaMatch, a unified solution for unsupervised domain adaptation (UDA), semi-supervised learning (SSL), and semi-supervised domain adaptation (SSDA).

- The effect achieved：Compare its behavior with respective state-of-the-art techniques from SSL, SSDA, and UDA and find that AdaMatch either **matches or significantly exceeds** the state-of-the-art in each case using the same hyper-parameters **regardless of the dataset or task**.



![img](https://cdn.nlark.com/yuque/0/2023/png/21656683/1675154970599-59ab5bb6-ca6d-4371-b857-36fc52642bcb.png)

## Action Items For The Next Two Weeks

Literature review:

- Learning Invariant Representations and Risks for Semi-supervised Domain Adaptation [CVPR 2021, https://github.com/Luodian/Learning-Invariant-Representations-and-Risks] 
- Surprisingly Simple Semi-Supervised Domain Adaptation with Pretraining and Consistency [BMVC 2021, https://github.com/venkatesh-saligrama/PAC] 