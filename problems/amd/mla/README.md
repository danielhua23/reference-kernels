# Description

You will implement a custom mla decode kernel optimized for MI300, a few things simplified here:

1. Q, K, V data type as bfloat16

2. provide Q, K, V hidden states directly, no Q, K, V up/down projections 
    
3. decode only with pre-allocated non-paged latent kv cache

4. no need to update kv cache

5. no need to implement RoPE in mla kernel, we only show its implementation in ref kernel

The shapes of all outer and inner dimensions of tensors are from DeepSeek-R1, and split number of heads to fit in one GPU. 
To be explicit, you will be given a tuple to tensors:

```yml
q_input [B, q_seqlen, h_q, kv_lora_rank + qk_rope_hd]
k_input [B, kv_seqlen, 1, kv_lora_rank + qk_rope_hd]
v_input [B, kv_seqlen, 1, kv_lora_rank]
attn_output [B, q_seqlen, h_q, kv_lora_rank]
``` 

where 

0. B::128 # batch size
1. kv_seqlen [1024, 6144]
2. q_seqlen:: 1 # as only consider decoding
3. qk_nope_head_dim:: 512
4. qk_rope_hd:: 64
5. kv_lora_rank(v_head_dim):: 512 
6. h_q:: 128  # num of q heads
7. h_kv:: 1 # as it's mla, kv head is 1


The ranking criteria is the geometric mean of the benchmark results.

For the grand price, your kernel will be evaluated against the speed of light analysis
and the solution closest to the speed of light will be awarded the grand price.

aiter performance for different kv_seqlen is below:
| batch | kv_seqlen | q_seqlen | dtype |  aiter time(us) |
|---|---|---|---|---|
| 128 | 1024 | 1 | bf16 | 152.52 |
| 128 | 6144 | 1 | bf16 | 640.57 | 