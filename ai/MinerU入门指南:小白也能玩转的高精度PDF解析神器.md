# MinerU 入门指南：小白也能玩转的高精度PDF解析神器

## 🚀 项目简介

MinerU 是一款将PDF转化为机器可读格式的开源工具，专门解决传统PDF工具无法处理的复杂文档结构问题。可以轻松将PDF文档转换为Markdown、JSON等格式，这些格式便于后续的数据处理和分析。这个项目诞生于书生-浦语（InternLM）的预训练过程中，项目团队在大模型训练中遇到科技文献解析难题，因此开发了这一专业工具。专门为解决科技文献中的符号转换问题而生，特别是数学公式、复杂表格等传统工具难以处理的内容。在大模型时代为科技发展贡献力量，为AI训练提供高质量的结构化数据源。

![项目演示](docs/images/MinerU-logo.png)

### 🎯 为什么选择MinerU？

相比传统的PDF解析工具如Adobe Acrobat、PDFplumber等，MinerU具有以下显著优势：

- **智能布局识别**：自动检测页眉、页脚、脚注、页码等元素，这些元素通常会干扰正文的连续性。确保语义连贯，避免出现"第1页"、"参考文献"等无关内容插入正文中
- **阅读顺序还原**：输出符合人类阅读习惯的文本，解决多栏布局时传统工具按坐标顺序输出导致的文本错乱问题。适用于单栏、多栏及复杂排版，包括学术论文、技术报告等专业文档
- **结构化保留**：完美保留原文档的结构，传统工具往往会丢失层次信息导致扁平化输出。包括标题、段落、列表等层次关系，保持文档的逻辑结构完整性
- **多媒体提取**：智能提取图像、图片描述、表格、表格标题及脚注，并建立它们之间的关联关系。解决传统工具无法准确匹配图表与其说明文字的问题
- **公式识别**：自动识别并转换文档中的数学公式为LaTeX格式，这是科技文献处理的核心需求。相比OCR工具的纯文本输出，LaTeX格式可以完美还原公式的数学语义
- **表格解析**：将复杂表格转换为HTML格式，保持原有结构包括合并单元格、表头关系等。传统工具通常只能提取表格文本而无法保持结构信息
- **OCR支持**：自动检测扫描版PDF，这类文档传统文本提取方法完全无效。支持84种语言的OCR识别，覆盖全球主要语言体系
- **多平台兼容**：支持Windows、Linux和Mac平台，满足不同开发环境需求。提供CPU和GPU加速选项，在不同硬件配置下都能获得最佳性能

## 🔥 核心特性

### 1. 多种解析后端架构
- **pipeline后端**：更通用，采用传统的模块化流水线设计，支持CPU推理适合资源受限环境。内部包含布局检测、OCR识别、公式解析、表格识别等独立模块，每个模块可以单独优化和替换
- **vlm-transformers后端**：更智能的多模态解析，基于视觉-语言大模型的端到端解决方案。通过统一的神经网络同时处理视觉和文本信息，解析质量更高但需要更多GPU资源
- **vlm-vllm后端**：更快速的推理引擎，专门针对大模型推理优化的加速框架。通过并行计算和内存优化技术，在保持解析质量的同时大幅提升处理速度

### 2. 智能内容识别能力
- 🔤 **文本识别**：支持中英文混合、手写文本、竖排文本，这些都是传统OCR工具的痛点。混合语言文档在国际化场景中很常见，手写文本在笔记和表单中大量存在，竖排文本在中文古籍和特殊排版中出现
- 📊 **表格解析**：支持旋转表格、无线框表格、跨页表格，这些复杂表格形式在实际文档中频繁出现。旋转表格常见于横向打印的宽表，无线框表格在现代文档设计中越来越普遍，跨页表格在长数据表中不可避免
- 🧮 **公式识别**：准确识别复杂数学公式，输出LaTeX格式保持数学语义的完整性。支持分数、积分、求和、矩阵等各种数学符号，这是科技文献处理的关键技术
- 🖼️ **图像处理**：提取图像并匹配对应的标题和描述，建立内容之间的语义关联。解决传统工具图文分离的问题，保持文档的完整性和可读性

### 3. 多样化输出格式
- **Markdown格式**：便于编辑和展示，是当前最流行的文档格式之一。支持GitHub、Typora等主流平台，方便二次编辑和版本控制
- **JSON格式**：结构化数据，便于程序处理和数据分析。包含完整的位置信息、类型标注、置信度等元数据，支持复杂的数据处理流程
- **HTML表格**：保持原有表格结构包括样式和布局，可以直接在网页中展示。比纯文本表格具有更好的视觉效果和交互性
- **图像文件**：提取的图片单独保存，保持原始分辨率和质量。支持PNG、JPEG等主流格式，便于后续的图像处理和分析

## 💡 核心亮点源码解析

### 🚀 智能批处理性能优化

MinerU实现了多层次的批处理优化策略，大幅提升大批量文档处理效率：

```python
class BatchAnalyze:
    def __init__(self, model_manager, batch_ratio: int, formula_enable, table_enable):
        self.batch_ratio = batch_ratio  # 根据GPU显存动态调整
        
        # 各模块的基础批处理大小
        YOLO_LAYOUT_BASE_BATCH_SIZE = 1
        MFD_BASE_BATCH_SIZE = 1
        MFR_BASE_BATCH_SIZE = 16
        OCR_DET_BASE_BATCH_SIZE = 16
    
    def dynamic_memory_allocation(self, device):
        # 智能显存检测和批次调整
        vram = get_vram(device)
        if vram >= 16:
            batch_ratio = 16  # 16GB+ 使用最大批次
        elif vram >= 12:
            batch_ratio = 8   # 12GB 中等批次
        elif vram >= 8:
            batch_ratio = 4   # 8GB 小批次
        elif vram >= 6:
            batch_ratio = 2   # 6GB 最小可用批次
        else:
            batch_ratio = 1   # 低于6GB 逐个处理
```

**分组批处理策略：**
```python
def resolution_based_batching(self, images_list):
    # 按分辨率分组，相同尺寸图像批处理
    RESOLUTION_GROUP_STRIDE = 128  # 分辨率分组步长
    resolution_groups = defaultdict(list)
    
    for img in images_list:
        h, w = img.shape[:2]
        # 归一化到128的倍数，减少GPU内存碎片
        normalized_h = ((h + RESOLUTION_GROUP_STRIDE) // RESOLUTION_GROUP_STRIDE) * RESOLUTION_GROUP_STRIDE
        normalized_w = ((w + RESOLUTION_GROUP_STRIDE) // RESOLUTION_GROUP_STRIDE) * RESOLUTION_GROUP_STRIDE
        group_key = (normalized_h, normalized_w)
        resolution_groups[group_key].append(img)
    
    # 对每组进行统一尺寸批处理
    for group_key, group_imgs in resolution_groups.items():
        target_h, target_w = group_key
        batch_images = []
        
        for img in group_imgs:
            h, w = img.shape[:2]
            # 白色背景padding到统一尺寸
            padded_img = np.ones((target_h, target_w, 3), dtype=np.uint8) * 255
            padded_img[:h, :w] = img  # 左上角对齐
            batch_images.append(padded_img)
        
        # 批量推理，充分利用GPU并行计算
        batch_results = model.batch_predict(batch_images, batch_size)
```

**性能优化亮点：**
- **自适应批次调整**：根据GPU显存大小自动调整批处理参数，在6GB-32GB不同配置下都能获得最佳性能
- **分辨率分组优化**：相同尺寸的图像分组处理，避免不同尺寸混合导致的内存浪费和计算低效
- **内存友好的Padding策略**：使用白色背景填充到统一尺寸，保持模型输入一致性的同时最小化内存开销
- **多级缓存机制**：模型权重缓存、中间结果缓存、计算图优化等多层次缓存策略
- **异步并发处理**：支持多文档并发处理，充分利用多核CPU和GPU资源

### 💾 内存优化与资源管理

MinerU实现了精细化的内存管理策略，显著降低资源占用：

```python
def memory_optimization_pipeline():
    # 1. 模型权重共享
    model_manager = AtomModelSingleton()  # 单例模式共享模型权重
    
    # 2. 动态模型加载卸载
    with torch.no_grad():  # 禁用梯度计算节省内存
        if not formula_enable:
            del self.mfr_model  # 不需要公式识别时释放模型
        if not table_enable:
            del self.table_models  # 不需要表格解析时释放模型
    
    # 3. 批处理后立即清理
    batch_results = model.batch_predict(batch_images)
    clean_vram(device)  # 清理GPU显存碎片
    clean_memory(device)  # 清理系统内存
    
    # 4. 流式处理大文档
    for page_batch in document_pages:
        process_batch(page_batch)
        torch.cuda.empty_cache()  # 每批次后清理缓存
```

**内存优化技术：**
- **显存需求降低70%**：从原来的24GB降低到10GB（开启所有功能），从16GB降低到8GB（基础功能）
- **模型权重共享**：多个处理实例共享同一份模型权重，避免重复加载
- **动态模型管理**：根据功能需求动态加载/卸载模型，不使用的模块不占用内存
- **流式处理架构**：大文档分批处理，避免一次性加载导致内存溢出
- **智能垃圾回收**：在关键节点主动清理内存和显存，避免内存泄漏

### 🏗️ Pipeline后端架构设计

MinerU采用了模块化的流水线设计，核心代码位于`pipeline_analyze.py`：

```python
def doc_analyze(pdf_bytes_list, lang_list, parse_method='auto', 
                formula_enable=True, table_enable=True):
    # 自动检测文档类型，决定是否启用OCR
    _ocr_enable = False
    if parse_method == 'auto':
        if classify(pdf_bytes) == 'ocr':  # 智能检测扫描文档
            _ocr_enable = True
    
    # 批处理优化，提升大批量文档处理效率
    batch_size = min_batch_inference_size
    batch_images = [images_with_extra_info[i:i + batch_size] 
                   for i in range(0, len(images_with_extra_info), batch_size)]
    
    # 执行批处理分析
    for batch_image in batch_images:
        batch_results = batch_image_analyze(batch_image, formula_enable, table_enable)
```

**设计亮点解析：**
- **智能文档分类**：`classify(pdf_bytes)`函数自动识别文档是原生PDF还是扫描件，这一步骤至关重要因为两种文档需要完全不同的处理策略
- **批处理优化**：通过动态调整批次大小，充分利用GPU并行计算能力，相比逐页处理可提升5-10倍效率
- **资源自适应**：根据GPU显存大小自动调整批处理参数，在6GB-32GB不同配置下都能获得最佳性能

### 🧠 VLM后端智能推理

VLM后端采用视觉-语言大模型，实现端到端的文档理解：

```python
def doc_analyze(pdf_bytes, image_writer, backend="transformers"):
    # 加载PDF页面为图像
    images_list, pdf_doc = load_images_from_pdf(pdf_bytes, ImageType.PIL)
    images_pil_list = [image_dict["img_pil"] for image_dict in images_list]
    
    # 多模态模型推理
    results = predictor.batch_two_step_extract(images=images_pil_list)
    
    # 转换为标准化中间格式
    middle_json = result_to_middle_json(results, images_list, pdf_doc, image_writer)
```

**技术创新点：**
- **两阶段推理**：先进行布局分析，再进行内容识别，这种解耦设计提高了准确性和可解释性
- **原生高分辨率处理**：直接处理高分辨率图像，保持细节信息不丢失，这是传统方法的重大改进
- **端到端优化**：单一模型完成多项任务，避免了传统pipeline中模块间的误差累积

### 🎯 智能布局检测核心

布局检测是文档解析的基础，MinerU使用了专门优化的YOLO模型：

```python
class DocLayoutYOLOModel:
    def __init__(self, weight, device="cuda", imgsz=1280, conf=0.1, iou=0.45):
        self.model = YOLOv10(weight).to(device)
        # 针对文档特点优化的参数
        self.imgsz = imgsz      # 高分辨率输入
        self.conf = conf        # 低置信度阈值，避免漏检
        self.iou = iou         # IoU阈值平衡精度和召回
    
    def batch_predict(self, images, batch_size=4):
        # 动态置信度调整
        for idx in range(0, len(images), batch_size):
            if batch_size == 1:
                conf = 0.9 * self.conf  # 单张图片时提高置信度
            else:
                conf = self.conf
```

**算法优化要点：**
- **高分辨率输入**：1280像素输入尺寸保证小字体和细节的检测精度
- **动态置信度**：根据批处理大小调整置信度阈值，平衡速度和准确性
- **多类别检测**：同时检测文本、图像、表格、公式等15+种文档元素

### 📊 表格解析技术突破

MinerU的表格解析结合了有线表和无线表两种算法：

```python
class MineruPipelineModel:
    def __init__(self, **kwargs):
        if self.apply_table:
            # 有线表格识别模型
            self.wired_table_model = atom_model_manager.get_atom_model(
                atom_model_name=AtomicModel.WiredTable, lang=self.lang)
            # 无线表格识别模型  
            self.wireless_table_model = atom_model_manager.get_atom_model(
                atom_model_name=AtomicModel.WirelessTable, lang=self.lang)
            # 表格分类模型
            self.table_cls_model = atom_model_manager.get_atom_model(
                atom_model_name=AtomicModel.TableCls)
```

**技术创新：**
- **双模型策略**：先分类再选择对应的专用模型，有线表和无线表采用不同算法
- **跨页合并**：支持跨页表格的自动识别和合并，解决长表格被截断的问题
- **结构还原**：完整还原合并单元格、表头层次等复杂结构

### 🧮 UniMERNet公式识别引擎

MinerU采用了先进的UniMERNet模型进行数学公式识别，这是一个基于视觉-语言大模型的端到端公式解析系统：

```python
class UnimernetModel:
    def predict(self, mfd_res, image):
        formula_list = []
        mf_image_list = []
        
        # 提取公式区域图像
        for xyxy, conf, cla in zip(mfd_res.boxes.xyxy.cpu(), 
                                   mfd_res.boxes.conf.cpu(), 
                                   mfd_res.boxes.cls.cpu()):
            xmin, ymin, xmax, ymax = [int(p.item()) for p in xyxy]
            new_item = {
                "category_id": 13 + int(cla.item()),  # 公式类别ID
                "poly": [xmin, ymin, xmax, ymin, xmax, ymax, xmin, ymax],
                "score": round(float(conf.item()), 2),
                "latex": "",
            }
            formula_list.append(new_item)
            bbox_img = image[ymin:ymax, xmin:xmax]  # 裁剪公式区域
            mf_image_list.append(bbox_img)

        # 批量处理公式图像
        dataset = MathDataset(mf_image_list, transform=self.model.transform)
        dataloader = DataLoader(dataset, batch_size=32, num_workers=0)
        
        mfr_res = []
        for mf_img in dataloader:
            mf_img = mf_img.to(dtype=self.model.dtype)
            mf_img = mf_img.to(self.device)
            with torch.no_grad():
                output = self.model.generate({"image": mf_img})
            mfr_res.extend(output["fixed_str"])
        
        # 填充LaTeX结果
        for res, latex in zip(formula_list, mfr_res):
            res["latex"] = latex
        
        return formula_list
```

**技术亮点：**
- **两阶段识别**：先用MFD模型检测公式位置，再用UniMERNet识别公式内容，这种解耦设计提高了准确性和鲁棒性
- **批量优化处理**：通过DataLoader批量处理公式图像，充分利用GPU并行计算能力，相比逐个识别可提升10倍以上效率
- **混合精度推理**：支持FP16推理加速，在保持精度的同时减少显存占用和计算时间
- **自适应输入处理**：自动调整图像尺寸和格式，适应不同分辨率和质量的公式图像

### 🏗️ UniMERNet架构深度解析

UniMERNet采用了基于MBart的编码器-解码器架构，专门针对数学公式识别进行了优化：

```python
class UnimerMBartForConditionalGeneration(UnimerMBartPreTrainedModel):
    def __init__(self, config: UnimerMBartConfig):
        super().__init__(config)
        self.model = UnimerMBartModel(config)
        
        # 专门的数学符号词汇表
        self.register_buffer("final_logits_bias", 
                           torch.zeros((1, self.model.shared.num_embeddings)))
        
        # 线性层用于生成LaTeX序列
        self.lm_head = nn.Linear(config.d_model, 
                               self.model.shared.num_embeddings, bias=False)

class UnimerMBartAttention(nn.Module):
    def __init__(self, embed_dim, num_heads, dropout=0.0, 
                 config: UnimerMBartConfig):
        super().__init__()
        self.embed_dim = embed_dim
        self.num_heads = num_heads
        
        # 关键创新：压缩注意力机制
        self.squeeze_dim = embed_dim // config.qk_squeeze  # 查询和键的压缩维度
        self.squeeze_head_dim = self.squeeze_dim // num_heads
        self.scaling = self.squeeze_head_dim**-0.5
        
        # 压缩的查询和键投影
        self.q_proj = nn.Linear(embed_dim, self.squeeze_dim, bias=bias)
        self.k_proj = nn.Linear(embed_dim, self.squeeze_dim, bias=bias)
        # 值投影保持原始维度
        self.v_proj = nn.Linear(embed_dim, embed_dim, bias=bias)
        self.out_proj = nn.Linear(embed_dim, embed_dim, bias=bias)
```

**核心创新技术：**
- **压缩注意力机制**：通过`qk_squeeze`参数将查询和键映射到低维空间，在不大幅降低信息量的前提下加速注意力计算。这是UniMERNet相比传统Transformer的重要优化
- **数学符号专用词汇表**：包含丰富的LaTeX数学符号，覆盖分数、积分、求和、矩阵、希腊字母等数学表达式的完整符号集
- **视觉特征编码器**：专门设计的图像编码器，能够准确提取数学公式的细粒度视觉特征，包括上下标关系、分数线、根号等复杂结构
- **序列解码优化**：针对LaTeX序列的特点优化解码策略，能够生成语法正确、语义准确的LaTeX代码

### 🔄 智能阅读顺序排序

使用基于LayoutReader的神经网络模型确定阅读顺序：

```python
def sort_by_layout_reader(page_line_list, page_w, page_h):
    # 坐标归一化到0-1000范围
    boxes = []
    for line in page_line_list:
        left, top, right, bottom = line['bbox']
        left = round(left * x_scale)
        top = round(top * y_scale) 
        right = round(right * x_scale)
        bottom = round(bottom * y_scale)
        boxes.append([left, top, right, bottom])
    
    # 使用LayoutReader模型预测阅读顺序
    model_manager = ModelSingleton()
    model = model_manager.get_model('layoutreader')
    with torch.no_grad():
        orders = do_predict(boxes, model)
    
    # 按预测顺序重排
    sorted_bboxes = [page_line_list[i] for i in orders]
```

**算法特点：**
- **神经网络排序**：基于Transformer架构，学习复杂文档的阅读模式
- **坐标归一化**：将不同尺寸文档映射到统一坐标系，提高模型泛化能力
- **上下文感知**：综合考虑所有元素的位置关系，而非简单的几何规则

## 📦 快速安装

### 环境要求

**硬件要求：**
- 内存：最低16GB，这是运行基本功能的最小要求。推荦32GB+，在处理大型文档或批量处理时能获得更好的性能和稳定性
- 磁盘：20GB+，主要用于存储模型文件和中间结果。推荐使用SSD，可显著提升模型加载和数据读写速度
- GPU：Turing架构及以上（GTX 1660及更新款），6GB+ VRAM对于pipeline后端。这个要求覆盖了大部分主流GPU，包括RTX 20/30/40系列（可选）

**软件要求：**
- Python 3.10-3.13，这是当前主流的Python版本范围，兼容性好。不支持过旧版本（3.9及以下）是因为缺少必要的语言特性
- 支持Linux、Windows、macOS，基本覆盖了所有主流开发平台。macOS用户可以使用Apple Silicon的MPS加速

### 一键安装

```bash
# 方法1：使用pip安装（推荐）
pip install --upgrade pip  # 升级pip到最新版本避免兼容性问题
pip install uv             # uv是更快的Python包管理器，可显著提升安装速度
uv pip install -U "mineru[core]"  # core包含所有核心功能但不包含vLLM加速

# 方法2：从源码安装（开发者）
git clone https://github.com/opendatalab/MinerU.git
cd MinerU
uv pip install -e .[core]  # -e参数允许在修改源码后直接生效，方便开发调试
```

### Docker部署
```bash
# 拉取镜像，包含了完整的运行环境和所有依赖
docker pull opendatalab/mineru:latest

# 运行容器，--gpus all参数支持GPU加速
docker run -it --gpus all opendatalab/mineru:latest
```

Docker部署的优势在于环境隔离，避免了Python依赖冲突问题，特别适合服务器部署和生产环境

## 🎮 使用方法

### 命令行使用

最简单的使用方式，只需要两个参数就能开始解析：
```bash
mineru -p input.pdf -o output_dir
```

带参数的完整命令，可以精确控制觧析行为：
```bash
mineru -p input.pdf -o output_dir \
    --lang ch \                    # 指定文档语言为中文，提高OCR准确性
    --backend pipeline \           # 使用pipeline后端，兼容性更好
    --method auto \                # 自动检测文档类型（原生或扫描）
    --formula \                    # 启用公式识别功能
    --table                        # 启用表格解析功能
```
这些参数都有合理的默认值，新手可以直接使用最简命令，高级用户可以根据需要精确调整

### Python API使用

对于需要集成到自己程序中的开发者，Python API提供了更灵活的控制：

```python
from pathlib import Path
from demo.demo import parse_doc

# 准备文档路径，支持PDF和图像文件
doc_paths = [Path("your_document.pdf")]
output_dir = "output"

# 使用pipeline后端解析，适合大部分场景
parse_doc(
    path_list=doc_paths,          # 要处理的文件列表
    output_dir=output_dir,        # 输出目录
    lang="ch",                   # 中文优化，支持84种语言
    backend="pipeline",          # 通用后端，支持CPU和GPU
    method="auto"                # 自动判断文档类型
)

# 使用VLM后端解析（更智能），适合对精度要求高的场景
parse_doc(
    path_list=doc_paths,
    output_dir=output_dir,
    backend="vlm-transformers"    # 多模态大模型，理解能力更强
)
```

### 批量处理示例

针对大量文档处理需求，MinerU提供了高效的批量处理方案：

```python
import os
from pathlib import Path
from demo.demo import parse_doc

# 批量处理PDF文件，适合文档库数字化项目
pdf_dir = Path("pdfs")
output_dir = "batch_output"

# 获取所有PDF文件，支持递归扫描子目录
pdf_files = list(pdf_dir.glob("*.pdf"))
print(f"找到 {len(pdf_files)} 个文件待处理")

# 批量解析，内部会自动进行批处理优化
parse_doc(
    path_list=pdf_files,
    output_dir=output_dir,
    lang="ch",                   # 根据主要文档语言调整
    backend="pipeline"           # 批量处理推荐使用pipeline后端
)
```

批处理的优势在于能充分利用GPU资源，相比逐个处理可提升5-10倍效率。系统会根据可用内存自动调整批大小，避免内存溢出

## 🛠️ 参数详解

### 后端选择策略
- `pipeline`：通用后端，采用模块化设计，每个组件都经过专门优化。支持CPU，适合大多数场景包括资源受限的环境。内部包含布局检测、OCR、公式识别、表格解析等独立模块
- `vlm-transformers`：多模态后端，基于先进的视觉-语言大模型技术。解析质量更高，能处理更复杂的文档结构，但需要更多的GPU资源和计算时间
- `vlm-vllm-engine`：高速推理引擎，专门针对大模型推理优化的加速框架。在保持VLM高质量的同时大幅提升处理速度，适合大量文档处理
- `vlm-http-client`：客户端模式，将推理任务委托给远程服务器处理。需要配置服务器URL，适合分布式部署和资源共享场景

### 语言支持范围
- `ch`：中文（默认），优化了中文字体识别和排版理解。支持15000+常用汉字，包括繁体字和日文汉字
- `en`：英文，针对英文文档的特殊排版和字体进行了优化。在纯英文环境下准确性更高
- `korean`：韩文，支持韩文字符和排版特点。包括韩文汉字和韩文字母的综合识别
- `japan`：日文，支持平假名、片假名和汉字的混合识别。针对日文的竖排版进行了专门优化
- `chinese_cht`：繁体中文，针对台湾、香港地区的繁体字使用习惯进行了特别优化
- 以及其他80+种语言，包括法语、德语、西班牙语、俄语等主流欧洲语言，以及阿拉伯语、泰语等亚非语言

### 解析方法选择
- `auto`：自动检测（推荐），系统会智能分析文档特征判断是原生PDF还是扫描件。对于混合文档（部分页面是原生、部分是扫描）也能智能处理
- `txt`：文本提取模式，适用于原生PDF文档。直接提取文本内容，速度最快但不能处理扫描文档和图像内的文字
- `ocr`：光学字符识别模式，适用于图像型PDF和扫描文档。通过图像识别技术提取文字信息，能处理各种复杂版面

### 功能开关配置
- `--formula` / `formula_enable=True`：启用公式识别，对科技文献和数学教材特别重要。会增加一定的处理时间但大幅提升数学内容的准确性
- `--table` / `table_enable=True`：启用表格解析，对报告和数据文档必不可少。能处理复杂表格结构包括合并单元格和跨页表格
- `--start-page` / `start_page_id`：指定起始页面，适合大文档的部分处理。可以先处理关键章节验证效果
- `--end-page` / `end_page_id`：指定结束页面，和起始页面组合使用可以精确控制处理范围

## 🎨 输出结果展示

### Markdown输出示例
MinerU的Markdown输出保持了原文档的完整结构和格式，下面是一个典型的输出示例：
```markdown
# 文档标题

## 章节标题

这是一段正文内容，包含了**粗体**和*斜体*文本。相比传统工具的纯文本输出，MinerU能保持这些格式信息，使得输出的文档更加美观和可读。

### 公式示例
$$E = mc^2$$

这个著名的爱因斯坦质能方程展示了MinerU对数学公式的准确识别能力。传统工具往往会将公式识别为普通文本或图像，而无法保持数学语义。

### 表格示例
| 列1 | 列2 | 列3 |
|-----|-----|-----|
| 数据1 | 数据2 | 数据3 |
| 数据4 | 数据5 | 数据6 |

这个表格展示了MinerU对结构化数据的处理能力。即使是复杂的多列表格，也能准确识别并转换为标准的Markdown表格格式。

![图片描述](images/figure_1.png)

图像处理展示了MinerU的智能图文关联能力。系统不仅能提取图像，还能自动匹配相应的标题和说明文字，这在传统工具中是非常困难的。
```

### JSON结构输出
JSON格式提供了更丰富的结构化信息，适合程序化处理：
```json
{
  "type": "paragraph",           // 元素类型：段落、标题、表格等
  "content": "这是一段文本内容",  // 实际文本内容
  "bbox": [100, 200, 300, 250],  // 元素在页面中的位置坐标
  "page_number": 1,              // 所在页面号
  "confidence": 0.95,            // 识别置信度
  "font_size": 12,               // 字体大小信息
  "reading_order": 3             // 在页面中的阅读顺序
}
```

这些丰富的元数据使得JSON格式非常适合二次开发，比如文档分析、版面重构、内容检索等应用。bbox坐标可以用于精确定位，阅读顺序保证了文本的逻辑连贯性。

### 可视化结果展示
MinerU还提供了丰富的可视化功能帮助用户验证解析结果：

**布局可视化**：生成`{filename}_layout.pdf`文件，在原文档上标注出所有检测到的布局元素。不同颜色的边框代表不同类型的内容（文本、图像、表格等），右上角的数字表示阅读顺序。

**文本块可视化**：生成`{filename}_span.pdf`文件，显示每个文本块的详细边界和分组情况。这对于调试文本提取的准确性非常有用，可以检查是否有文本丢失或分割错误。

## 🚀 进阶功能探索

### 1. 配置文件自定义
MinerU支持通过配置文件进行高级自定义，这为专业用户提供了更精细的控制能力。创建配置文件 `mineru.json`：
```json
{
  "formula_enable": true,          // 启用公式识别，对科技文献至关重要
  "table_enable": true,            // 启用表格解析，处理数据报告必备
  "layout_model": "doclayout_yolo", // 布局检测模型选择，影响解析速度和精度
  "ocr_model": "PP-OCRv5",         // OCR模型版本，v5对手写文本支持更好
  "batch_size": 4,                // 批处理大小，影响内存使用和处理速度
  "output_formats": ["markdown", "json"], // 同时输出多种格式
  "custom_formula_delimiters": {   // 自定义公式分隔符，适应不同文档风格
    "inline": ["$", "$"],
    "display": ["$$", "$$"]
  }
}
```
这些配置可以根据具体使用场景进行调整，比如处理纯文本文档时可以关闭公式和表格识别提高速度。

### 2. WebUI界面体验
MinerU内置了基于Gradio的Web界面，无需编程即可体验全部功能：
```bash
mineru gradio  # 启动Web界面，默认在http://localhost:7860访问
```

Web界面支持拖拽上传文件、实时参数调整、在线预览结果等功能。特别适合非技术用户和快速原型验证，降低了工具的使用门槛。

### 3. API服务部署
MinerU提供了专业的API服务，支持RESTful接口调用：
```bash
mineru fastapi  # 启动API服务，默认端口8000
```

API服务支持批量处理、异步任务、进度查询等企业级功能。内置了详细的API文档和交互式测试界面，方便开发者集成到自己的系统中。

### 4. 模型管理系统
MinerU内置了智能的模型管理系统，自动处理模型下载和更新：
```bash
# 下载所有必要模型，首次使用时自动执行
mineru download

# 指定模型源（国内用户推荐），解决网络访问问题
export MINERU_MODEL_SOURCE=modelscope
mineru download

# 检查模型版本和更新状态
mineru check-models
```

模型管理系统会自动检查本地模型版本，在有更新时提示用户升级。这确保了用户始终使用最新、性能最优的模型版本。

## 🎯 应用场景深度分析

### 学术研究领域
- **论文文献批量处理**：科研人员常需要处理大量的学术论文，传统PDF阅读效率低且无法批量处理。MinerU可以批量提取论文的核心内容包括摘要、结论、实验数据等，为研究综述和元分析提供原料
- **公式和表格数据提取**：数学、物理、化学等科技文献包含大量公式和实验数据。MinerU的LaTeX公式输出可以直接用于重新排版，表格数据可以导入统计软件进行分析
- **参考文献整理**：自动提取和整理文献中的参考文献，为文献管理软件（如Zotero、EndNote）提供结构化数据，提高文献检索和引用的效率

### 办公自动化应用
- **合同文档数字化**：企业的纸质合同和历史档案数字化是一个巨大工程。传统扫描后的PDF文件无法编辑和搜索，MinerU可以将其转换为可编辑的结构化文档，支持全文搜索和智能分析
- **报告内容提取**：从复杂的商业报告、财务报表中提取关键数据和指标。自动识别表格结构并转换为数据库可读格式，支持后续的数据分析和可视化
- **文档格式转换**：将老旧的PDF文档批量转换为现代办公格式（Word、Markdown等），便于协作编辑和版本控制。保持原有的格式和结构，减少手动调整的工作量

### 数据挖掘与分析
- **大规模文档预处理**：在大数据和机器学习项目中，需要处理海量PDF文档作为训练数据。MinerU的批处理能力和结构化输出能够大幅提高数据预处理的效率和质量
- **结构化数据提取**：从非结构化的PDF文档中提取结构化信息，如产品规格表、价格列表、组织架构等。输出的JSON格式可以直接入库进行数据分析
- **内容分析准备**：为自然语言处理、情感分析、主题建模等任务提供高质量的文本数据。保持文档的逻辑结构和语义关系，提高后续分析的准确性

### 知识管理与检索
- **企业文档库建设**：将企业内部的PDF文档资源（政策文件、技术文档、培训资料等）转换为可搜索的结构化数据。支持全文搜索、分类标签、关联推荐等高级功能
- **知识图谱构建**：从文档中提取实体、关系和属性信息，为知识图谱构建提供原始数据。结构化的输出格式使得实体识别和关系抽取更加准确有效
- **内容搜索优化**：通过结构化解析，建立更精确的内容索引。支持按章节、按类型（文本、表格、图像）、按主题等多维度搜索，提高信息检索的效率和准确性

## ⚠️ 注意事项与优化建议

### 已知限制与解决方案
- **极端复杂排版可能出现阅读顺序错乱**：对于非常复杂的多栏布局或不规则排版，尤其是杂志、报纸等设计复杂的出版物。解决方案：可以先用`--start-page`和`--end-page`参数测试关键页面，或尝试VLM后端获得更好的结果
- **竖排文字支持有限**：主要针对中文古籍、日文文档等传统排版格式。原因是竖排文本的训练数据相对较少。解决方案：可以尝试手动旋转文档后再处理，或使用专门的日文OCR参数
- **部分不常见列表格式无法识别**：目录和列表的识别主要基于规则，对于非常特殊的格式可能失效。解决方案：可以在后处理中手动调整，或提交issue帮助改进识别规则
- **复杂表格可能出现行列识别错误**：特别是包含大量合并单元格、嵌套表格或不规则边框的表格。解决方案：尝试使用VLM后端，或将表格区域单独截取处理
- **小语种OCR可能存在字符不准确问题**：如阿拉伯文易混淆字符、拉丁文重音符号等。解决方案：可以尝试更新到最新的OCR模型版本，或使用专门的语言模型

### 性能优化建议
- **使用GPU加速可显著提升处理速度**：相比CPU模式，GPU加速可以提升5-20倍的处理速度。对于大批量文档处理，建议优先考虑GPU配置
- **批量处理时建议分批进行**：对于数千个文档的大规模处理，建议每次处理100-500个文件为一批。这样可以避免内存溢出，同时方便断点续传和错误恢复
- **大文档建议指定页面范围处理**：对于超过50页的大文档，可以先处理关键章节验证效果，再逐段处理全文。使用`--start-page`和`--end-page`参数可以精确控制处理范围
- **定期清理临时文件释放磁盘空间**：解析过程中会生成大量中间文件和缓存数据。定期清理`output`目录下的临时文件可以节约磁盘空间

### 质量保证措施
- **使用可视化结果验证解析质量**：强烈建议使用`--draw-layout-bbox`参数生成布局可视化结果。通过检查布局检测结果可以及早发现问题并调整参数
- **重要文档建议使用多种后端对比**：对于关键文档，可以同时使用pipeline和vlm后端处理，对比结果选择最佳方案。不同后端在不同类型文档上有各自优势
- **建立质量检查流程**：对于生产环境，建议建立随机抽样检查机制。通过统计各种元素的识别准确率，可以及时发现模型性能退化或参数不当的问题

## 🤝 社区支持

## 🤝 社区支持

### 在线体验平台
MinerU提供了多个在线体验平台，让用户能够在不安装任何软件的情况下快速体验工具的强大功能：
- [官方网站](https://mineru.net) - 免登录在线体验，这是最快速的体验方式，无需注册即可上传PDF文件进行解析测试
- [HuggingFace](https://huggingface.co/spaces/opendatalab/MinerU) - 演示版本，基于HuggingFace Spaces部署，提供稳定的在线服务和详细的使用说明
- [ModelScope](https://www.modelscope.cn/studios/OpenDataLab/MinerU) - 国内访问优化版本，专为中国用户提供的镜像服务，访问速度更快

这些在线平台都提供了完整的功能演示，包括文档上传、参数配置、实时解析、结果下载等完整流程，让用户能够充分了解MinerU的能力和特点。

### 技术支持与帮助
当您在使用过程中遇到问题时，可以通过以下渠道获得专业的技术支持：
- [官方文档](https://opendatalab.github.io/MinerU/zh/) - 最权威的使用指南，包含详细的安装说明、API文档、配置参数、常见问题等完整信息
- [GitHub Issues](https://github.com/opendatalab/MinerU/issues) - 技术问题反馈平台，开发团队会及时响应用户反馈的bug和功能需求
- [Discord社区](https://discord.gg/Tdedn9GTXq) - 国际用户交流平台，可以与全球开发者和用户实时交流技术问题和使用经验
- [微信群](https://mineru.net/community-portal/) - 中文用户交流群，更适合中国用户的沟通习惯，可以获得中文技术支持

### 常见问题与解决方案
基于大量用户反馈，我们整理了最常见的问题和对应的解决方案：

1. **安装失败问题**：这通常是由于Python版本不兼容或网络连接问题导致的。请确保使用Python 3.10-3.13版本，并检查网络连接是否正常，必要时可以使用国内镜像源
2. **显存不足错误**：当出现CUDA out of memory错误时，可以通过降低batch_size参数或切换到CPU模式来解决。对于低配置设备，建议使用pipeline后端而非VLM后端
3. **解析效果不佳**：如果解析结果不理想，可以尝试不同的后端（pipeline vs vlm）或调整语言参数。对于特殊类型的文档，可能需要手动调整OCR和布局检测的参数
4. **中文乱码问题**：确保使用正确的语言参数（--lang ch），并检查系统的中文字体支持。在某些Linux环境下可能需要安装中文字体包

这些解决方案已经帮助数千名用户成功解决了使用中的问题，如果遇到其他问题，建议查阅官方文档或在社区中寻求帮助。

## 🎉 总结

## 🎉 总结

MinerU作为一款强大的PDF解析工具，不仅提供了高精度的内容提取能力，还具备了良好的易用性和扩展性。这个项目代表了当前文档智能处理领域的最高水平，特别是在科技文献处理方面表现出色。无论是学术研究、办公自动化还是数据挖掘，MinerU都能为您提供可靠的文档处理解决方案。

### 🎆 技术创新亮点

MinerU在多个技术维度实现了突破性创新：
- **架构创新**：同时支持Pipeline和VLM两种后端，在性能和精度之间实现完美平衡。Pipeline后端通过模块化设计提供了极佳的稳定性和兼容性，VLM后端则通过端到端大模型实现了更高的智能理解能力
- **算法优化**：在公式识别领域采用了UniMERNet的压缩注意力机制，在保持精度的同时大幅提升了计算效率。这种创新设计使得复杂数学公式的识别变得实用化
- **性能突破**：通过智能批处理、分辨率分组、内存优化等多层次优化，在大批量文档处理中相比传统方法提升10-20倍性能，显存需求降低70%以上

### 🌍 应用前景广阔

随着数字化转型的加速，MinerU的应用场景正在不断拓展：
- **科研领域**：随着开放科学运动的发展，越来越多的科研机构需要对历史文献进行数字化处理。MinerU的高精度公式和表格识别能力使其成为科研数字化的理想工具
- **企业市场**：在企业数字化转型中，大量的纸质文档和历史档案需要转换为结构化数据。MinerU的批量处理能力和高质量输出能够显著降低人工成本和提高处理效率
- **AI训练数据**：在大模型时代，高质量的训练数据是AI模型成功的关键。MinerU能够将PDF文档转换为结构化、语义完整的数据，为AI模型训练提供优质数据源

### 🚀 技术路线展望

随着项目的不断发展和社区的积极贡献，MinerU正在成为PDF文档智能处理领域的标杆工具。未来的发展方向包括：
- **多模态扩展**：支持更多文档格式（Word、PPT、Excel等），打造统一的文档处理平台
- **智能化增强**：集成更先进的大模型技术，实现更高级的文档理解和内容生成能力
- **生态丰富**：构建更完善的开发者生态，包括更多的集成组件、插件系统和第三方工具支持

如果您在工作中经常需要处理PDF文档，不妨试试这款“小白也能玩转的高精度PDF解析神器”！通过它，您将能够体验到前所未有的文档处理效率和质量，让繁琐的文档处理工作变得轻松高效。

---

**开始您的MinerU之旅：**
```bash
pip install uv
uv pip install -U "mineru[core]"
mineru -p your_document.pdf -o output
```

让我们一起探索文档智能处理的新世界！🌟