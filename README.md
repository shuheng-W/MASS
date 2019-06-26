[Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct)

[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/mass-masked-sequence-to-sequence-pre-training/unsupervised-machine-translation-on-wmt2014-2)](https://paperswithcode.com/sota/unsupervised-machine-translation-on-wmt2014-2?p=mass-masked-sequence-to-sequence-pre-training)
[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/mass-masked-sequence-to-sequence-pre-training/unsupervised-machine-translation-on-wmt2014-1)](https://paperswithcode.com/sota/unsupervised-machine-translation-on-wmt2014-1?p=mass-masked-sequence-to-sequence-pre-training)
[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/mass-masked-sequence-to-sequence-pre-training/unsupervised-machine-translation-on-wmt2016)](https://paperswithcode.com/sota/unsupervised-machine-translation-on-wmt2016?p=mass-masked-sequence-to-sequence-pre-training)
[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/mass-masked-sequence-to-sequence-pre-training/unsupervised-machine-translation-on-wmt2016-1)](https://paperswithcode.com/sota/unsupervised-machine-translation-on-wmt2016-1?p=mass-masked-sequence-to-sequence-pre-training)
[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/mass-masked-sequence-to-sequence-pre-training/unsupervised-machine-translation-on-wmt2016-3)](https://paperswithcode.com/sota/unsupervised-machine-translation-on-wmt2016-3?p=mass-masked-sequence-to-sequence-pre-training)
[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/mass-masked-sequence-to-sequence-pre-training/unsupervised-machine-translation-on-wmt2016-5)](https://paperswithcode.com/sota/unsupervised-machine-translation-on-wmt2016-5?p=mass-masked-sequence-to-sequence-pre-training)







# MASS

[MASS](https://arxiv.org/pdf/1905.02450.pdf) is a novel pre-training method for sequence to sequence based language generation tasks. It randomly masks a sentence fragment in the encoder, and then predicts it in the decoder.

![img](figs/mass.png)

MASS can be applied on cross-lingual tasks such as neural machine translation, and monolingual tasks such text summarization. The current codebase supports unsupervised, low-resource and rich-resource NMT, text summarization and conversational response generation. We will We will release our implementation for other sequence to sequence generation tasks in the future.




## Unsupervised NMT

Unsupervised Neural Machine Translation just uses monolingual data to train the models. For this task, we implement MASS on [XLM](https://github.com/facebookresearch/XLM).

We also provide pre-trained and fine-tuned models:

| Languages | Pre-trained Model | Fine-tuned Model | BPE codes | Vocabulary |
|-----------|:-----------------:|:----------------:| ---------:| ----------:|
| EN - FR   | [MODEL](https://modelrelease.blob.core.windows.net/mass/mass_enfr_1024.pth)    |   [MODEL](https://modelrelease.blob.core.windows.net/mass/mass_ft_enfr_1024.pth)   | [BPE codes](https://dl.fbaipublicfiles.com/XLM/codes_enfr) | [Vocabulary](https://dl.fbaipublicfiles.com/XLM/vocab_enfr) |
| EN - DE   | [MODEL](https://modelrelease.blob.core.windows.net/mass/mass_ende_1024.pth) | [MODEL](https://modelrelease.blob.core.windows.net/mass/mass_ft_ende_1024.pth) | [BPE codes](https://dl.fbaipublicfiles.com/XLM/codes_ende) | [Vocabulary](https://dl.fbaipublicfiles.com/XLM/vocab_ende) |
| En - RO   | [MODEL](https://modelrelease.blob.core.windows.net/mass/mass_enro_1024.pth) | [MODEL](https://modelrelease.blob.core.windows.net/mass/mass_ft_enro_1024.pth) | [BPE_codes](https://dl.fbaipublicfiles.com/XLM/codes_enro) | [Vocabulary](https://dl.fbaipublicfiles.com/XLM/vocab_enro) |

We are also preparing larger models on more language pairs, and will release them in the future.

### Data Ready

We use the same BPE codes and vocabulary with XLM. Here we take English-French as an example.

```
cd MASS

wget https://dl.fbaipublicfiles.com/XLM/codes_enfr
wget https://dl.fbaipublicfiles.com/XLM/vocab_enfr

./get-data-nmt.sh --src en --tgt fr --reload_codes codes_enfr --reload_vocab vocab_enfr
```

### Pre-training:
```
python train.py                                      \
--exp_name unsupMT_enfr                              \
--data_path ./data/processed/en-fr/                  \
--lgs 'en-fr'                                        \
--mass_steps 'en,fr'                                 \
--encoder_only false                                 \
--emb_dim 1024                                       \
--n_layers 6                                         \
--n_heads 8                                          \
--dropout 0.1                                        \
--attention_dropout 0.1                              \
--gelu_activation true                               \
--tokens_per_batch 3000                              \
--optimizer adam_inverse_sqrt,beta1=0.9,beta2=0.98,lr=0.0001 \
--epoch_size 200000                                  \
--max_epoch 100                                      \
--eval_bleu true                                     \
--word_mass 0.5                                      \
--min_len 5                                          \
```


During the pre-training prcess, even without any back-translation, you can observe the model can achieve some intial BLEU scores:
```
epoch -> 4
valid_fr-en_mt_bleu -> 10.55
valid_en-fr_mt_bleu ->  7.81
test_fr-en_mt_bleu  -> 11.72
test_en-fr_mt_bleu  ->  8.80
```

### Fine-tuning 
After pre-training, we use back-translation to fine-tune the pre-trained model on unsupervised machine translation:
```
MODEL=mass_enfr_1024.pth

python train.py \
  --exp_name unsupMT_enfr                              \
  --data_path ./data/processed/en-fr/                  \
  --lgs 'en-fr'                                        \
  --bt_steps 'en-fr-en,fr-en-fr'                       \
  --encoder_only false                                 \
  --emb_dim 1024                                       \
  --n_layers 6                                         \
  --n_heads 8                                          \
  --dropout 0.1                                        \
  --attention_dropout 0.1                              \
  --gelu_activation true                               \
  --tokens_per_batch 2000                              \
  --batch_size 32	                                     \
  --bptt 256                                           \
  --optimizer adam_inverse_sqrt,beta1=0.9,beta2=0.98,lr=0.0001 \
  --epoch_size 200000                                  \
  --max_epoch 30                                       \
  --eval_bleu true                                     \
  --reload_model "$MODEL,$MODEL"                       \
```

## Text Summarization
To apply MASS on text summarization, we provide an example of how to run MASS pre-training and fine-tuning on the [Gigaword](https://github.com/harvardnlp/sent-summary) dataset.


| Pre-trained Model | BPE codes | Vocabulary |
|:-----------------:| ---------:| ----------:|
| Coming soon       | [BPE codes](https://modelrelease.blob.core.windows.net/mass/codes_en) | [Vocabulary](https://modelrelease.blob.core.windows.net/mass/vocab_en) |

### Pre-training:
For pre-training, we use the following command:

```
python train.py                                      \
--exp_name mass_english                              \
--data_path ./data/processed/en/                     \
--lgs 'en'                                           \
--mass_steps 'en'                                    \
--encoder_only false                                 \
--emb_dim 1024                                       \
--n_layers 6                                         \
--n_heads 8                                          \
--dropout 0.1                                        \
--attention_dropout 0.1                              \
--gelu_activation true                               \
--tokens_per_batch 3000                              \
--optimizer adam_inverse_sqrt,beta1=0.9,beta2=0.98,lr=0.0001 \
--epoch_size 200000                                  \
--max_epoch 100                                      \
--eval_bleu true                                     \
--word_mass 0.5                                      \
--min_len 5                                          \
--english_only true
```

### Fine-tuning:
Different from unsupervised NMT tasks, we directly use paired data (article-title) to fine-tune the pre-trained model. The fine-tuning command is:

```
MODEL=mass_en_1024.pth 

python train.py                                      \
--exp_name mass_summarization                        \
--data_path ./data/processed/summarization/          \
--lgs 'ar-ti'                                        \
--mt_steps 'ar-ti'                                   \
--encoder_only false                                 \
--emb_dim 1024                                       \
--n_layers 6                                         \
--n_heads 8                                          \
--dropout 0.2                                        \
--attention_dropout 0.2                              \
--gelu_activation true                               \
--tokens_per_batch 3000                              \
--optimizer adam_inverse_sqrt,beta1=0.9,beta2=0.98,lr=0.0001 \
--epoch_size 200000                                  \
--max_epoch 20                                       \
--eval_bleu true                                     \
--english_only true                                  \
--reload_model "$MODEL,$MODEL"
```

## Reference

If you find MASS useful in your work, you can cite the paper as below:

    @inproceedings{song2019mass,
        title={MASS: Masked Sequence to Sequence Pre-training for Language Generation},
        author={Song, Kaitao and Tan, Xu and Qin, Tao and Lu, Jianfeng and Liu, Tie-Yan},
        booktitle={International Conference on Machine Learning},
        pages={5926--5936},
        year={2019}
    }

