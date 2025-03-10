# LLM

## From AI
+ 大语言模型为什么要推理引擎  

    大语言模型（LLM）需要推理引擎的原因在于，推理引擎是将模型的能力与实际应用场景连接起来的关键组件。推理引擎不仅负责执行模型的预测和生成任务，还为模型提供了一个高效、灵活、可控的运行环境。以下是详细的原因和背景：1. 推理引擎的核心作用(1) 模型执行的桥梁
    •大语言模型本质上是一个复杂的数学模型，训练完成后需要通过推理引擎来加载和运行。
    •推理引擎负责将用户的输入数据（如文本）传递给模型，并接收模型的输出结果。
    (2) 性能优化
    •推理引擎会对模型进行优化，以提高推理速度和降低资源消耗。 
    •量化：将模型从高精度（FP32）转换为低精度（INT8/FP16），减少内存占用。
    •并行计算：利用多核 CPU 或 GPU 加速推理过程。
    •批处理：支持批量处理用户请求，提升吞吐量。
    (3) 灵活性与扩展性
    •推理引擎通常支持多种硬件平台（如 CPU、GPU、TPU），可以根据实际需求选择最佳运行环境。
    •支持动态切换不同版本或类型的模型，满足多样化的业务需求。
    2. 推理引擎在实际应用中的必要性(1) 高效处理复杂任务
    •LLM 的推理过程可能涉及多个步骤，例如： 
    •输入预处理（分词、编码等）。
    •模型推理（生成中间结果）。
    •输出后处理（解码、格式化等）。
    •推理引擎能够自动化这些流程，确保每个步骤无缝衔接。
    (2) 支持实时性和高并发
    •在生产环境中，LLM 应用通常需要处理大量并发请求。
    •推理引擎通过负载均衡、缓存机制和异步处理等技术，确保系统在高并发场景下的稳定性和响应速度。
    (3) 安全性保障
    •推理引擎可以集成安全模块，对输入和输出进行过滤和检查，防止恶意攻击（如 Prompt Injection）。
    •提供日志记录和审计功能，便于追踪和排查问题。
    3. 推理引擎的具体实现方式以下是一些常见的推理引擎及其特点：(1) TensorRT
    •特点: 
    •NVIDIA 提供的高性能推理引擎，专为 GPU 优化。
    •支持模型量化和动态张量形状。
    •适用场景: 
    •需要极致性能的 GPU 加速环境。
    (2) ONNX Runtime
    •特点: 
    •开源推理引擎，支持跨平台部署。
    •兼容多种框架（如 PyTorch、TensorFlow）导出的模型。
    •适用场景: 
    •需要在不同硬件平台上运行模型的应用。
    (3) Triton Inference Server
    •特点: 
    •NVIDIA 提供的企业级推理服务框架。
    •支持多模型并发、动态批处理和自动扩缩容。
    •适用场景: 
    •大规模生产环境，尤其是云原生架构。
    (4) vLLM
    •特点: 
    •针对 LLM 优化的开源推理引擎。
    •提供高效的 KV 缓存管理和连续批处理能力。
    •适用场景: 
    •高吞吐量的 LLM 应用。
    4. 推理引擎的工作流程以下是推理引擎在 LLM 应用中的典型工作流程：[用户输入] --> [输入预处理] --> [推理引擎] --> [模型推理] --> [输出后处理] --> [返回结果]
    (1) 输入预处理
    •将用户输入（如自然语言文本）转换为模型可接受的格式（如 Token ID 序列）。
    •示例工具：HuggingFace Tokenizer。
    (2) 模型推理
    •推理引擎加载模型权重，执行前向传播计算，生成中间结果。
    •示例工具：PyTorch、TensorFlow。
    (3) 输出后处理
    •将模型输出（如 Token ID 序列）解码为人类可读的文本。
    •示例工具：Beam Search、Top-K Sampling。
    5. 推理引擎的未来趋势(1) 更高效的硬件适配
    •随着专用 AI 芯片（如 TPU、NPU）的发展，推理引擎将更加注重硬件加速能力。
    (2) 动态自适应优化
    •推理引擎将根据输入数据的特点动态调整模型参数（如精度、批大小），以实现最优性能。
    (3) 零信任安全架构
    •推理引擎将集成更强的安全模块，防止模型被滥用或攻击。
    总结来说，推理引擎是大语言模型应用中不可或缺的一部分，它不仅提升了模型的运行效率，还为模型的实际落地提供了灵活性和安全性。如果您需要针对特定场景选择合适的推理引擎，或者需要更详细的实施方案，请进一步说明需求！


+ LLM有哪几种架构  
    大语言模型（LLM, Large Language Models）主要基于不同的架构设计，这些架构在处理自然语言任务时各有优势。以下是几种常见的架构类型：
    1. Transformer 架构：
    •这是目前最流行的语言模型架构之一，由 Vaswani 等人在 2017 年的论文《Attention is All You Need》中提出。
    •它摒弃了传统的循环神经网络（RNN）和卷积神经网络（CNN），完全依赖于自注意力机制（self-attention mechanism），允许模型并行化训练，并且能更好地捕捉长距离依赖关系。
    •变体：如 BERT (Bidirectional Encoder Representations from Transformers)，GPT (Generative Pre-trained Transformer)，T5 (Text-to-Text Transfer Transformer) 等。
    2. 循环神经网络（RNN）及其变体：
    •包括 LSTM (Long Short-Term Memory) 和 GRU (Gated Recurrent Unit)，它们通过引入门控机制解决了传统 RNN 在处理长时间序列数据时存在的梯度消失问题。
    •虽然在某些特定任务上仍有一定的应用价值，但在大多数情况下已经被更高效的 Transformer 架构所取代。
    3. 卷积神经网络（CNN）：
    •使用卷积操作来提取局部特征，并结合池化层减少参数数量。
    •对于短文本或需要局部上下文感知的任务可能表现较好，但在处理长序列时效率不如 Transformer。
    4. 稀疏变换器（Sparse Transformer）：
    •是一种改进版的 Transformer 模型，它利用稀疏连接模式降低计算复杂度，使得模型可以在更大规模的数据集上进行训练。
    •特别适合处理超长序列的任务。
    5. 图神经网络（GNN, Graph Neural Network）：
    •将文本表示为图结构，节点代表词汇项，边表示它们之间的关联。
    •在涉及语义关系推理、知识图谱等任务时具有独特的优势。
    6. 多模态架构：
    •结合视觉、音频等多种感知信息与文本信息共同建模，如 CLIP（Contrastive Language–Image Pre-training）。
    •此类模型能够实现跨模态理解，在图像字幕生成、视觉问答等领域展现出强大的能力。
    7. 自回归模型 vs 自编码模型：
    •自回归模型（例如 GPT）逐个预测下一个 token，通常用于生成式任务。
    •自编码模型（例如 BERT）尝试恢复被掩盖的部分，更多地应用于理解型任务。
    8. 轻量化模型：
    •随着对部署成本和效率的关注增加，出现了许多致力于压缩模型体积而不显著损失性能的研究成果，比如 DistilBERT、TinyBERT 等。
    •这些模型适用于资源受限环境下的快速推理。
    9. 混合专家模型（MoE, Mixture of Experts）：
    •MoE 模型将多个小型专家子模型组合起来，每个专家负责处理不同类型的输入。
    •它们可以在保持较高准确性的前提下显著减少计算量，是近年来研究的一个热点方向。
    总结上述提到的各种架构并非完全独立存在，很多现代 LLM 都会融合多种技术的优点以提升性能。例如，最新的预训练模型可能会同时包含 Transformer 的核心组件以及一些专门针对特定任务优化的设计元素。如果你对某个具体的架构感兴趣或者想要了解更多细节，请随时提问！内容由AI生成


## LLM Inference Engine and Service
+ llama.cpp  
    https://github.com/ggml-org/llama.cpp  

+ llama-cpp-python  
    https://github.com/abetlen/llama-cpp-python  


### Prompt Engineering  
+ Prompt Engineering Guide  
    [Prompt Engineering Guide](https://www.promptingguide.ai/)  

+ 一文带你速通RAG、知识库和LLM  
    [Retrieval Augmented Generation，RAG](https://zhuanlan.zhihu.com/p/700166877)  

## Reference
+ Kaggle  
    [Kaggle](https://www.kaggle.com/) is the place to learn data science and build a portfolio.  

+ GPUStack  
    [GPUStack](https://docs.gpustack.ai/overview/). GPUStack is an open-source GPU cluster manager for running large language models(LLMs).  

+ Understanding Deep Learning  
    [Understanding Deep Learning](https://udlbook.github.io/udlbook/)  