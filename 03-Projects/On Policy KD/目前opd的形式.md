OPD
= reverse-KL distillation on student rollouts

sequence-level OPD
= 对整条学生回答做 reverse-KL

token-level OPD
= 对每个位置做局部 reverse-KL 近似

sampled-token OPD
= token-level OPD 的单样本近似，只看学生采样出的 token