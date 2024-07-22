# Train the model for agent

We release a tool dataset in **ModelScope** for training and evaluation of finetune LLM([Tool dataset](https://www.modelscope.cn/datasets/modelscope/ms_hackathon_23_agent_train_dev/summary)). A corresponding training script using **ModelScope** library is provided.


## training options

The training script supports various training methods based on your available resources:

- Supervised fine-tuning with full-parameters or Lora.
- Distributed training by integration with ModelScope DeepspeedHook.

## data preprocess

A piece of data may be formulated as follow. Since we use a text-generation task scheme to train LLM, the origin data need to be preprocessed.

```Python
System: system info(agent info, tool info...).
User: user inputs.
Assistants:
# agent call
<|startofthink|>...<|endofthink|>\n\n
# tool execute
<|startofexec|>...<|endofexec|>\n
# summarize
...
# may be multiple rounds
```

- Each data instance consists of three roles: system, user, and assistant. The LLM should only focus on the **assistant** part.
- The **assistant** part is typically composed of three sections. The LLM should only consider the content of the agent call and the final summary.
- The other unnecessary parts are masked using `IGNORE_INDEX` to exclude them from loss calculation.

## training scripts

To start a training, you can use the script `run_train_ddp.sh'

```Shell
CUDA_VISIBLE_DEVICES=0 \
python llm_sft.py \
    --model_type modelscope-agent-7b \
    --sft_type lora \
    --output_dir runs \
    --dataset damo/MSAgent-Bench \
    --dataset_sample 20000 \
    --dataset_test_ratio 0.02 \
    --max_length 2048 \
    --dtype bf16 \
    --lora_rank 8 \
    --lora_alpha 32 \
    --lora_dropout_p 0.1 \
    --batch_size 1 \
    --learning_rate 1e-4 \
    --gradient_accumulation_steps 16 \
    --eval_steps 50 \
    --save_steps 50 \
    --save_total_limit 2 \
    --logging_steps 20 \
    --use_flash_attn true \
```

## evaluate

After training, we also provided an evaluation script to judge the performance of agents in testing datasets. The ground truth of testing datasets is generated by ...

The evaluation metrics include:
- Accuracy of tool names and parameters.
- `Rouge-l` metric for similarity of summarization.