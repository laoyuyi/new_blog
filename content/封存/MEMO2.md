- Llama2 run
```shell
python run_clm_pt_with_peft.py --model_name_or_path .\pretrained_model --tokenizer_name_or_path .\tokenizer --dataset_dir .\dataset --data_cache_dir .\data_cache --validation_split_percentage 0.001 --per_device_train_batch_size 1 --per_device_eval_batch_size 1 --do_train --seed 0 --max_steps 200 --lr_scheduler_type cosine --learning_rate 1e-7 --warmup_ratio 0.05 --weight_decay 0.01 --logging_strategy steps --logging_steps 10 --save_strategy steps --save_total_limit 3 --save_steps 500 --gradient_accumulation_steps 1 --preprocessing_num_workers 8 --block_size 1 --output_dir .\output --overwrite_output_dir --ddp_timeout 30000 --logging_first_step True --lora_rank 8 --trainable "q_proj,v_proj,k_proj,o_proj,gate_proj,down_proj,up_proj" --lora_dropout 0.1 --torch_dtype float16
```

- **HfArgumentParser的使用：**

```PYTHON
# script.py
from transformers import HfArgumentParser
# 定义数据类
@dataclass
class MyArguments:
    model_name: str
    num_epochs: int
# 创建HfArgumentParser实例
parser = HfArgumentParser((MyArguments,))
# 解析命令行参数
args = parser.parse_args()
# 使用解析结果
print("Model name:", args.model_name)
print("Number of epochs:", args.num_epochs)
```
```CSS
python script.py --model_name my_model --num_epochs 10
```

dhw1sdf233
llllllll

ios america: dhurwegeronav@outlook.com----Pzx85460----qaz1----wsx2----edc3----1984-04-14

hoebeskflayj@outlook.com----ka7nTp61该账户用微软授权登录----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
984230315113040183
![[../assets/memo1.png]]
[OpenAI ChatGPT账号批发 - ChatGPT店铺 (scwxq.cn)](https://shop.scwxq.cn/)

华农用户名：105044101903494

API-KEY：sk-6f70cc06ed874eeeb4074555f8b5345b

parse_args -> main -> load_configuration -> search_articles

pubmed: cfdcb71a38844a028977513f273d42443007