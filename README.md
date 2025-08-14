
# Spectronaut-Data-Preprocessing-
A standardized workflow for processing Spectronaut proteomics data

# Spectronaut 蛋白质组学数据分析流程


这是一个标准化的分析流程，用于处理由 Spectronaut 软件导出的 Label-Free 定量蛋白质组学数据。项目从原始报告文件开始，完成数据质控、预处理、统计分析和核心结果的可视化。

## 项目结构

```
.
├── data/               # 存放原始数据文件 (.xlsx)，此目录受 .gitignore 保护，但通过 .gitkeep 保留结构。
├── figures/            # 存放生成的所有图表 (如PCA图、火山图、热图)。
├── processed_data/     # 存放处理后的中间数据 (如干净的定量矩阵)。
├── scripts/            # 存放所有的分析代码 (Jupyter Notebook 或 Python 脚本)。
├── LICENSE             # 项目许可证文件。
└── README.md           # 项目说明文件 (就是你正在看的这个)。
```

## 环境设置与安装

1.  **克隆本仓库**
    ```bash
    git clone https://github.com/Jasper-delong/Spectronaut-Data-Preprocessing.git
    cd Spectronaut-Data-Preprocessing
    ```

2.  **安装所需依赖**
    本项目依赖于多个 Python 库。建议在虚拟环境中安装。
    ```bash
    pip install pandas openpyxl matplotlib seaborn scikit-learn matplotlib-venn
    ```
    *注意：代码中为了实现中文显示，指定了字体路径 `/usr/share/fonts/truetype/wqy/wqy-zenhei.ttc`。请确保您的系统中存在此字体或修改为其他可用中文字体路径。*

## 分析流程大纲 (Analysis Workflow)

本流程主要分为两个核心阶段：数据清洗与质控，以及数据整理与归一化。

## 目录 (Table of Contents)

*   [1. 阶段一：数据清洗与质量控制 (Data Cleaning & Quality Control)](#1-阶段一数据清洗与质量控制-data-cleaning--quality-control)
    *   [1.1. 数据导入 (Data Import)](#11-数据导入-data-import)
    *   [1.2. 数据清洗与过滤 (Data Cleaning & Filtering)](#12-数据清洗与过滤-data-cleaning--filtering)
    *   [1.3. 实验重复性评估 (Assess Reproducibility)](#13-实验重复性评估-assess-reproducibility)
    *   [1.4. 绝对质量评估 (Absolute Quality Assessment)](#14-绝对质量评估-absolute-quality-assessment)

---

## 1. 阶段一：数据清洗与质量控制 (Data Cleaning & Quality Control)

本阶段的核心目标是确保用于分析的数据是干净、可靠的。我们将移除低质量和无关的匹配，并评估实验本身的稳定性与数据质量。

### 1.1. 数据导入 (Data Import)
**状态**: ✅ 已完成

**操作描述**:
使用 Python 的 `pandas` 库读取 Spectronaut 生成的 `.xlsx` 报告文件。通过 `pd.read_excel()` 函数，将原始数据加载到一个 DataFrame 变量中，为后续处理做好准备。

### 1.2. 数据清洗与过滤 (Data Cleaning & Filtering)
**状态**: ✅ 已完成

**操作描述**:
此步骤是保证数据质量的关键，目的是剔除会干扰分析的无效数据。此步骤包含以下三个核心任务：

1.  **移除伪数据库匹配 (Decoys)**:
    *   **原因**: 伪数据库（Decoy）匹配是用于计算错误发现率（FDR）的“诱饵”序列，并非真实蛋白质，必须在生物学分析前移除。
    *   **操作**: 筛选并只保留 `EG.IsDecoy` 列为 `False` 的行。

2.  **移除已知的污染物 (Contaminants)**:
    *   **原因**: 样品制备过程中会引入角蛋白、胰蛋白酶等常见污染物，它们并非源自目标样本，需剔除以避免干扰。
    *   **操作**: Spectronaut 通常用 `CON__` 前缀标记污染物。筛选并移除 `PG.ProteinAccessions` 列中所有以此为开头的行。

3.  **基于 Q-value 进行过滤**:
    *   **原因**: `PG.Qvalue` 代表蛋白质组鉴定的可信度。为保证结果可靠性，只保留高可信度的蛋白质。
    *   **操作**: 依据行业标准，筛选 `PG.Qvalue < 0.01` 的数据，这意味着在保留的蛋白质中，预计只有不到 1% 是错误鉴定。

### 1.3. 实验重复性评估 (Assess Reproducibility)
**状态**: ✅ 已完成

**操作描述**:
本步骤旨在评估技术或生物学重复样本之间的一致性，是衡量实验稳定性的重要指标。通过两个函数 `assess_reproducibility` 和 `analyze_quantitative_reproducibility` 实现。

1. **蛋白质组与肽段数目重复性监测**:
    *   **目的**: 检查在重复样本中鉴定到的蛋白质和肽段数量是否稳定，重叠度是否高。
    *   **操作**: 使用 `assess_reproducibility` 函数，该函数可以：
        *   统计每个样本中唯一的蛋白质组和肽段数量，并生成**条形图 (Bar Plot)**进行可视化比较。
        *   当指定分析 2 或 3 个样本时，自动绘制**韦恩图 (Venn Diagram)**，直观展示它们之间的鉴定重叠情况。

2. **蛋白质组定量重复性检验**:
    *   **目的**: 评估重复样本间蛋白质丰度测量的相关性和一致性。
    *   **操作**: 使用 `analyze_quantitative_reproducibility` 函数，该函数在对数据进行 Log2 转换后，自动化生成以下图表：
        *   **相关性热图 (Correlation Heatmap)**: 计算并展示样本间皮尔逊相关系数，数值越接近1，重复性越好。
        *   **主成分分析图 (PCA Plot)**: 通过降维将样本在二维空间中展示，重复性好的样本会聚集在一起。
        *   **定量分布箱线图 (Box Plot)**: 比较各样本定量值的分布情况，检查是否存在需要校正的系统性偏差。

### 1.4. 绝对质量评估 (Absolute Quality Assessment)
**状态**: ✅ 已完成

**操作描述**:
除了样本间的重复性，本流程还包含了对数据绝对质量的评估，以确保鉴定结果本身是可靠的。

1.  **每个蛋白质组的肽段支持数**:
    *   **目的**: 评估蛋白质鉴定的证据强度。一个蛋白质被越多的唯一肽段所支持，其鉴定结果就越可靠。
    *   **操作**: 统计每个蛋白质组对应的唯一肽段数量，并绘制**直方图 (Histogram)**，展示其整体分布。通常，大部分蛋白质应由至少2个肽段支持。

2.  **酶切效率分析 (Missed Cleavages)**:
    *   **目的**: 评估胰蛋白酶的酶切效率。高效的酶切会产生大量没有“漏切位点”的肽段。
    *   **操作**: 统计唯一肽段中包含 0, 1, 2... 个漏切位点的比例，并生成**条形图**。一个高质量的实验，通常超过70%的肽段应为0个漏切位点。



## 如何使用

1.  将您的 Spectronaut 输出的 `.xlsx` 文件放入 `/data` 文件夹。
2.  打开 `/scripts` 文件夹中的主分析 Notebook (`preprocessing.ipynb` 文件)。
3.  根据 Notebook 中的说明，修改文件路径、工作表名称以及用于分析的样本白名单 (`sample_whitelist`) 等参数。
4.  按顺序运行 Notebook 中的所有代码块，生成的图表会自动保存在 `/figures` 目录。

## 作者


- **[Jasper]**
- **联系邮箱**: 1256749831@qq.com
- **GitHub**: [@Jasper-delong](https://github.com/Jasper-delong)