# SoftVC VITS Singing Voice Conversion
## Updates
> According to incomplete statistics, a multi-speaker dataset might cause **the timbre to be less stable**, currently **single speaker** datasets are recommended for training for the model to sound closer to the target voice.\
> In light of the issue where inferencing sovits3.0 48kHz models take up high amounts of vram (video memory), you can switch to the [32kHz branch](https://github.com/innnky/so-vits-svc/tree/32k) and train a 32kHz model.\
> Currently there's a larger issue where inferencing models trained on the 3.0 version code takes up a great deal of vram, with 6GB of vram it's only possible to inference audio that's around 30 seconds or so.\
> Fixed the issue with unwanted staccato, improving audio quality by a decent amount.\
> The 2.0 version has been moved to the 2.0 branch.\
> Version 3.0 uses the code structure of FreeVC, which isn't compatible with older versions.\
> Currently the audio quality is still worse than [DiffSVC](https://github.com/prophesier/diff-svc) 
## Model Overview
A singing voice coversion (SVC) model, using the SoftVC encoder to extract features from the input audio, sent into VITS along with the F0 to replace the original input to acheive a voice conversion effect. Additionally, changing the vocoder to [NSF HiFiGAN](https://github.com/openvpi/DiffSinger/tree/refactor/modules/nsf_hifigan) to fix the issue with unwanted staccato.
## Notice
The current branch is the 48kHz version, which requires more vram during inferencing, often causing OOM (Out Of Memory) errors, if this happens, it's required to manually splice the samples to convert them one by one, and so it's recommended to use the [32kHz branch](https://github.com/innnky/so-vits-svc/tree/32k) and train 32kHz variant models for use.
## Colab notebook script for dataset creation and training.
[colab training notebook](https://colab.research.google.com/drive/1rCUOOVG7-XQlVZuWRAj5IpGrMM8t07pE?usp=sharing)

## Required models
+ soft vc hubert：[hubert-soft-0d54a1f4.pt](https://github.com/bshall/hubert/releases/download/v0.1/hubert-soft-0d54a1f4.pt)
  + Place under `hubert`.
+ Pretrained models [G_0.pth](https://huggingface.co/innnky/sovits_pretrained/resolve/main/G_0.pth) and [D_0.pth](https://huggingface.co/innnky/sovits_pretrained/resolve/main/D_0.pth)
  + Place under `logs/48k`.
  + Pretrained models are required, because from experiments, training from scratch can be rather unpredictable to say the least, and training with a pretrained model can greatly improve training speeds.
  + The pretrained model includes云灏, 即霜, 辉宇·星AI, 派蒙, and 绫地宁宁, covering the common ranges of both male and female voices, and so it can be seen as a rather universal pretrained model.
  + The pretrained model exludes the `optimizer speaker_embedding` section, rendering it only usable for pretraining and incapable of inferencing with.
```shell
# For simple downloading.
# hubert
wget -P hubert/ https://github.com/bshall/hubert/releases/download/v0.1/hubert-soft-0d54a1f4.pt
# G&D pretrained models
wget -P logs/48k/ https://huggingface.co/innnky/sovits_pretrained/resolve/main/G_0.pth
wget -P logs/48k/ https://huggingface.co/innnky/sovits_pretrained/resolve/main/D_0.pth

```


## Dataset preparation
All that is required is that the data be put under the `dataset_raw` folder in the structure format provided below.
```shell
dataset_raw
├───speaker0
│   ├───xxx1-xxx1.wav
│   ├───...
│   └───Lxx-0xx8.wav
└───speaker1
    ├───xx2-0xxx2.wav
    ├───...
    └───xxx7-xxx007.wav
```

## Data pre-processing.
1. Resample to 48khz

```shell
python resample.py
 ```
2. Automatically sort out training set, validation set, test set, and automatically generate configuration files.
```shell
python preprocess_flist_config.py
# Notice.
# The n_speakers value in the config will be set automatically according to the amount of speakers in the dataset.
# To reserve space for additionally added speakers in the dataset, the n_speakers value will be be set to twice the actual amount.
# If you want even more space for adding more data, you can edit the n_speakers value in the config after runing this step.
# This can not be changed after training starts.
```
3. Generate hubert and F0 features/
```shell
python preprocess_hubert_f0.py
```
After running the step above, the `dataset` folder will contain all the pre-processed data, you can delete the `dataset_raw` folder after that.

## Training.
```shell
python train.py -c configs/config.json -m 48k
```

## Adding more speaker data while training.
Basically similar to pre-processing.
1. Add the appended data under `dataset_raw` in the same format as before, and resample to 48kHz.
```shell
python resample.py
 ```
2. Use `add_speaker.py` to regenerate the training set, validation set, and configuration file.
```shell
python add_speaker.py
```
3. Regenerate hubert and F0 features.
```shell
python preprocess_hubert_f0.py
```
You can delete the `dataset_raw` folder after this.

## Inferencing.

Use [inference_main.py](inference_main.py)
+ Edit `model_path` to your newest checkpoint.
+ Place the input audio under the `raw` folder.
+ Change `clean_names` to the output file name.
+ Use `trans` to edit the pitch shifting amount (semitones). 
+ Change `spk_list` to the speaker name.
