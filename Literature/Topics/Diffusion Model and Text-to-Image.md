# Diffusion Model and Text-to-Image

## 文献列表

| 论文 | 状态 | 总结级别 | 优先级 | 备注 |
|---|---|---|---|---|
| [[Papers/DDPM\|DDPM]] | `fully-summarized` | `full` | `medium` |  |
| [[Papers/DDIM\|DDIM]] | `fully-summarized` | `full` | `medium` |  |
| [[Papers/LDM\|LDM]] | `fully-summarized` | `full` | `medium` | Batch 002 已完成全文总结。 |
| [[Papers/Null-text Inversion for Editing Real Images using Guided Diffusion Models\|Null-text Inversion for Editing Real Images using Guided Diffusion Models]] | `fully-summarized` | `full` | `medium` | Batch 002 已完成全文总结。 |
| [[Papers/DreamBooth\|DreamBooth]] | `fully-summarized` | `full` | `medium` |  |
| [[Papers/Deep Unsupervised Learning using Nonequilibrium Thermodynamics\|Deep Unsupervised Learning using Nonequilibrium Thermodynamics]] | `unread` | `none` | `medium` |  |
| [[Papers/Improved DDPM\|Improved DDPM]] | `unread` | `none` | `medium` |  |
| [[Papers/CFG\|CFG]] | `fully-summarized` | `full` | `high` | 第一遍读的时候这部分过的比较快，再读这个系列的时候需要深究一下里面的原理；CFG 似乎也是文生图、文生 3D 中的基础之一。 |
| [[Papers/SDE\|SDE]] | `unread` | `none` | `medium` |  |
| [[Papers/VAE\|VAE]] | `unread` | `none` | `medium` | 也就是 VAE |
| [[Papers/VQ-VAE\|VQ-VAE]] | `unread` | `none` | `medium` | VQ-VAE |
| [[Papers/CLIP\|CLIP]] | `unread` | `none` | `medium` | CLIP |
| [[Papers/Generative Adversarial Networks\|Generative Adversarial Networks]] | `unread` | `none` | `medium` |  |
| [[Papers/Conditional Generative Adversarial Nets\|Conditional Generative Adversarial Nets]] | `unread` | `none` | `medium` |  |
| [[Papers/Wasserstein GAN\|Wasserstein GAN]] | `unread` | `none` | `medium` |  |
| [[Papers/CycleGAN\|CycleGAN]] | `unread` | `none` | `medium` | CycleGAN |
| [[Papers/Pix2Pix\|Pix2Pix]] | `unread` | `none` | `medium` | Pix2Pix |

## 阅读顺序

1. [[Papers/DDPM|DDPM]]
1. [[Papers/DDIM|DDIM]]
1. [[Papers/LDM|LDM]]
1. [[Papers/Null-text Inversion for Editing Real Images using Guided Diffusion Models|Null-text Inversion for Editing Real Images using Guided Diffusion Models]]
1. [[Papers/DreamBooth|DreamBooth]]
1. [[Papers/Deep Unsupervised Learning using Nonequilibrium Thermodynamics|Deep Unsupervised Learning using Nonequilibrium Thermodynamics]]
1. [[Papers/Improved DDPM|Improved DDPM]]
1. [[Papers/CFG|CFG]]
1. [[Papers/SDE|SDE]]
1. [[Papers/VAE|VAE]]
1. [[Papers/VQ-VAE|VQ-VAE]]
1. [[Papers/CLIP|CLIP]]
1. [[Papers/Generative Adversarial Networks|Generative Adversarial Networks]]
1. [[Papers/Conditional Generative Adversarial Nets|Conditional Generative Adversarial Nets]]
1. [[Papers/Wasserstein GAN|Wasserstein GAN]]
1. [[Papers/CycleGAN|CycleGAN]]
1. [[Papers/Pix2Pix|Pix2Pix]]

## 用户原始备注

> 这个系列是扩散模型的原理和应用。[Deep Unsupervised Learning using Nonequilibrium Thermodynamics](https://arxiv.org/pdf/1503.03585) 补充了 DDPM 中一些略过的细节。[Improved Denoising Diffusion Probabilistic Models](https://arxiv.org/pdf/2102.09672) 在 DDPM 的基础上对训练过程进行了一些改进。在 Null-text Inversion 中，用到了 [Classifier-Free Diffusion Guidance](https://arxiv.org/pdf/2207.12598)（在第一遍读的时候这部分过的比较快，再读这个系列的时候需要深究一下里面的原理），CFG 似乎也是文生图、文生 3D 中的基础之一。[Score-Based Generative Modeling through Stochastic Differential Equations](https://arxiv.org/pdf/2011.13456) 也是 DDPM 系列的基础。
>
> T2I 系列，比较重要（听过了解过但没细致读过的）：
>
> 生成系列：
