$$时间轴：x_i = arrival\_time - first\_arrival\_time$$

$$延迟差：delta\_ms = recv\_delta - send\_delta$$

$$累积延迟差： accumulated\_delay += delta\_ms$$

$$指数移动平均累积延迟差：y_i = 0.9 * y_{i-1} + 0.1 * accumulate\_delay$$

$$数学模型：简单线性回归 \quad y_i = trend * x_i + b$$

$$
趋势：trend = \sum (x_i-\overline x)(y_i-\overline y) / \sum (x_i-\overline x)^2
$$

$$修改后的趋势：modified\_trend = min(num\_of\_deltas,60) * trend * 4.0$$

$$
时间间隔：time\_delta = arrival\_time - last\_update
$$

$$
阈值计算：

threshold_i = threshold_{i-1} + time\_delta * k_i * (|modified\_trend| - threshold)
$$

$$
status = \begin{cases}
kBwOverusing,\quad modified\_trend > threshold_i \\\\
kBwUnderusing, \quad modified\_trend < -threhold_i \\\\
kBwNormal, \quad others
\end{cases}
$$