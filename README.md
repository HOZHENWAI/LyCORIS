# LyCORIS - Lora beYond Conventional methods, Other Rank adaptation Implementations for Stable diffusion.

![image](https://user-images.githubusercontent.com/59680068/224026402-7b779d58-5164-4ecd-a807-f98badae589e.png)
(This image is generated by the model trained in Hadamard product representation)

A project for implementing different algorithm to do parameter-efficient finetuning on stable diffusion or more.

This project is started from LoCon(see archive branch).


## What we have now
See [Algo.md](https://github.com/KohakuBlueleaf/LyCORIS/blob/main/Algo.md) or [Demo.md](https://github.com/KohakuBlueleaf/LyCORIS/blob/main/Demo.md) for more example and explanation

### Conventional LoRA
* Include Conv layer implementation from LoCon
* recommended settings
  * dim <= 64
  * alpha = 1 (or lower, like 0.3)


### LoRA with Hadamard Product representation (LoHa)
* Ref: [FedPara Low-Rank Hadamard Product For Communication-Efficient Federated Learning](https://openreview.net/pdf?id=d71n4ftoCBy)
* designed for federated learning, but has some cool property like rank<=dim^2 so should be good for parameter-efficient finetuning.
  * Conventional LoRA is rank<=dim
* recommended settings
  * dim <= 32
  * alpha = 1 (or lower)
  
**WARNING: You are not supposed to use dim>64 in LoHa, which is over sqrt(original_dim) for almost all layer in SD**

**High dim with LoHa may cause unstable loss or just goes to NaN. If you want to use high dim LoHa, please use lower lr**

**WARNING-AGAIN: Use parameter-efficient algorithim in parameter-unefficient way is not a good idea**

---

## usage
### For kohya script
Activate sd-scripts' venv and then install this package
```bash
source PATH_TO_SDSCRIPTS_VENV/Scripts/activate
```
or
```powershell
PATH_TO_SDSCRIPTS_VENV\Scripts\Activate.ps1 # or .bat for cmd
```

And then you can install this package:
* through pip
```bash
pip install lycoris_lora
```

* from source
```bash
git clone https://github.com/KohakuBlueleaf/LyCORIS
cd LyCORIS
pip install .
```

Finally you can use this package's kohya module to run kohya's training script
```bash
python3 sd-scripts/train_network.py \
  --network_module lycoris.kohya \
  --network_dim "DIM_FOR_LINEAR" --network_alpha "ALPHA_FOR_LINEAR"\
  --network_args "conv_dim=DIM_FOR_CONV" "conv_alpha=ALPHA_FOR_CONV" \
  "dropout=DROPOUT_RATE" "algo=lora" \
```
to train lycoris module for SD model

* algo list:
  * lora: Conventional Methods
  * loha: Hadamard product representation introduced by FedPara

* Tips:
  * Use network_dim=0 or conv_dim=0 to disable linear/conv layer
  * LoHa doesn't support dropout yet.


### For a1111's sd-webui
download [Extension](https://github.com/KohakuBlueleaf/a1111-sd-webui-locon) into sd-webui, and then use your model as how you use lora model.
**LoHa Model supported**


### Additional Networks
Once you install the extension. You can also use your model in [addnet](https://github.com/kohya-ss/sd-webui-additional-networks/releases)<br>
just use it as LoRA model.
**LoHa Model not supported yet**


### Extract LoCon
You can extract LoCon from a dreambooth model with its base model.
```bash
python3 extract_locon.py <settings> <base_model> <db_model> <output>
```
Use --help to get more info
```
$ python3 extract_locon.py --help
usage: extract_locon.py [-h] [--is_v2] [--device DEVICE] [--mode MODE] [--safetensors] [--linear_dim LINEAR_DIM] [--conv_dim CONV_DIM]
                        [--linear_threshold LINEAR_THRESHOLD] [--conv_threshold CONV_THRESHOLD] [--linear_ratio LINEAR_RATIO] [--conv_ratio CONV_RATIO]
                        [--linear_percentile LINEAR_PERCENTILE] [--conv_percentile CONV_PERCENTILE]
                        base_model db_model output_name
```


## Example and Comparing for different algo
see [Demo.md](https://github.com/KohakuBlueleaf/LyCORIS/blob/lycoris/Demo.md) and [Algo.md](https://github.com/KohakuBlueleaf/LyCORIS/blob/lycoris/Algo.md)


## Change Log
For full log, please see [Change.md](https://github.com/KohakuBlueleaf/LyCORIS/blob/lycoris/Change.md)

### 2023/03/12 Update to 0.1.0
* Add cp-decomposition implementation for convolution layer
  * Both LoRA(LoCon) and LoHa can use this more parameter-efficient decomposition
* Add sparse bias for extracted LoRA
  * Will add to training in the future (Maybe)
* Change weight initialization method in LoHa
  * Use lower std to avoid loss to go high or NaN when using normal lr (like 0.5 in Dadap)


## Todo list
- [ ] Module and Document for using LyCORIS in any other model, Not only SD.
- [ ] Proposition3 in [FedPara](https://arxiv.org/abs/2108.06098)
  * also need custom backward to save the vram
- [ ] Low rank + sparse representation
- [ ] Support more operation, not only linear and conv2d.
- [ ] Configure varying ranks or dimensions for specific modules as needed.
- [ ] Automatically selecting an algorithm based on the specific rank requirement.
- [ ] Explore other low-rank representations or parameter-efficient methods to fine-tune either the entire model or specific parts of it.
- [ ] More experiments for different task, not only diffusion models.


## Citation
```bibtex
@misc{LyCORIS,
  author       = "Shih-Ying Yeh (Kohaku-BlueLeaf), Yu-Guan Hsieh, Zhidong Gao",
  title        = "LyCORIS - Lora beYond Conventional methods, Other Rank adaptation Implementations for Stable diffusion",
  howpublished = "\url{https://github.com/KohakuBlueleaf/LyCORIS}",
  month        = "March",
  year         = "2023"
}
```
