# 三种模式工具调用测试

三种模式指 `native` `xml` `blackbox_json`

`native` 是原版的 Function Call。我起初认为它不靠谱，因为在真实的 Agent 环境中会遇到大量需要转义的符号，比如这个 `C:\Users\Kirsmin` 不可以直接写入 JSON 字符串，需要转义。

像这种路径问题不大，但需要转义的符号多了模型调用工具可能会出错。

`xml` 是我设计了一个 demo，像 `<tool_call:show> Hello World </tool_call:show>`。这种方式我认为很理想，因为几乎不需要处理转义问题。但是逆 RL 可能会浪费 CoT、结果不理想。经过测试确实如此。

`blackbox_json` 是我设计的特殊 JSON 格式。其实不算 JSON，因为改变了定义。比如在字符串中使用 `"<INDEX>C:\Users\Kirsmin</INDEX>"` 来实现免转义。但是结果很不理想。模型会浪费大量 CoT 来思考工具调用，或者给出错误的调用。

设计测试实例时我一开始使用的方案是单轮简单消息，发现 XML 的表现更好；第二次试验我使用模拟真实 Agent 环境，发现原生调用方式才是最好的。因此问题终结。

另外我需要谴责一下 Kimi。49元档我当时跑没几下测试就 100% 5h Limited了。所以后面的测试都是走 API 按量计费的。
<img width="1366" height="642" alt="图片" src="https://github.com/user-attachments/assets/d07e655a-304e-422e-bb2b-80303cf12eac" />

总结开销

Kimi: 会员不计，API 4.02511 元，K2.6 2.22元，K2.7Code 1.81元。

| 模型 | 请求数 | 输入 Tokens | 输出 Tokens | Cached Tokens | 合计 |
|---|---:|---:|---:|---:|---:|
| kimi-k2.6 | 18 | 22,107 | 77,696 | 4,352 | 99,803 |
| kimi-k2.7-code | 25 | 28,589 | 62,146 | 10,882 | 90,735 |
| **合计** | **43** | **50,696** | **139,842** | **15,234** | **190,538** |

Deepseek：API 0.24 元，Deepseek V4.1 Flash

| 模型 ID | 输入（命中缓存） | 输入（未命中缓存） | 输出 | 总计 |
|---|---:|---:|---:|---:|
| deepseek-flash | 27,008 | 31,885 | 54,247 | 113,140 |

总结：白测试了，浪费接近￥5
