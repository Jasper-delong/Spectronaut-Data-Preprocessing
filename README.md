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
    本项目依赖于以下 Python 库。建议在虚拟环境中安装。
    ```bash
    pip install pandas openpyxl matplotlib seaborn
    ```

## 分析流程大纲 (Analysis Workflow)


## 目录 (Table of Contents)

*   [1. 阶段一：数据质控 (Data Quality Control)](#1-阶段一数据质控-data-quality-control)
    *   [1.1. 数据导入 (Data Import)](#11-数据导入-data-import)
    *   [1.2. 数据清洗与过滤 (Data Cleaning & Filtering)](#12-数据清洗与过滤-data-cleaning--filtering)
    *   [1.3. 评估实验重复性 (Assess Reproducibility)](#13-评估实验重复性-assess-reproducibility)
*   [2. 阶段二：数据整理与归一化 (Data Wrangling & Normalization)](#2-阶段二数据整理与归一化-data-wrangling--normalization)
    *   [2.1. 构建定量矩阵 (Build Quantification Matrix)](#21-构建定量矩阵-build-quantification-matrix)
    *   [2.2. 数据预处理 (Matrix Preprocessing)](#22-数据预处理-matrix-preprocessing)
*   [3. 阶段三：统计分析与可视化 (Statistical Analysis & Visualization)](#3-阶段三统计分析与可视化-statistical-analysis--visualization)
    *   [3.1. 探索性数据分析 (Exploratory Data Analysis)](#31-探索性数据分析-exploratory-data-analysis)
    *   [3.2. 差异蛋白分析 (Differential Expression Analysis)](#32-差异蛋白分析-differential-expression-analysis)
    *   [3.3. 核心结果可视化 (Key Results Visualization)](#33-核心结果可视化-key-results-visualization)

---

## 1. 阶段一：数据质控 (Data Quality Control)
<!-- 在这里总体介绍第一阶段的目标 -->
本阶段的核心目标是确保我们分析的数据是干净、可靠的。我们将移除低质量和无关的匹配，并评估实验本身的稳定性。

### 1.1. 数据导入 (Data Import)
**状态**: ✅ 已完成

**操作描述**:
在这一步，我们使用了 Python 的 `pandas` 库来读取 Spectronaut 生成的 `.xlsx` 报告文件。通过 `pd.read_excel()` 函数，我们指定了文件的路径和要读取的工作表 (Sheet) 名称，成功将原始数据加载到一个名为 `df` 的 DataFrame 变量中，为后续处理做好了准备。

### 1.2. 数据清洗与过滤 (Data Cleaning & Filtering)
**状态**: ✅ 已完成

**操作描述**:
这是数据质控最关键的步骤，目的是为了剔除会干扰分析的无效数据。此步骤包含以下三个子任务：

1.  **移除伪数据库匹配 (Decoys)**:
    *   **为什么这么做？** 伪数据库（Decoy）是用于计算错误发现率（FDR）的“诱饵”序列，它们不是真实的蛋白质。在进行生物学分析前，必须将其完全移除。
    *   **如何做？** 我们将筛选 DataFrame，只保留 `EG.IsDecoy` 列为 `False` 的行。

2.  **移除已知的污染物 (Contaminants)**:
    *   **为什么这么做？** 样品处理过程中不可避免地会引入一些常见污染物（如角蛋白、胰蛋白酶等），这些蛋白质并非来自我们的目标样本，会干扰定量和后续分析。
    *   **如何做？** Spectronaut 通常会用 `CON__` 前缀来标记这些污染物蛋白。我们将筛选并移除 `PG.ProteinAccessions` 列中所有以此为开头的行。

3.  **基于 Q-value 进行过滤**:
    *   **为什么这么做？** `PG.Qvalue` 代表蛋白质组鉴定的可信度（即FDR）。为了保证分析结果的可靠性，我们只保留那些高可信度的蛋白质。
    *   **如何做？** 行业标准是筛选 `PG.Qvalue < 0.01` 的数据，这意味着在我们保留的所有蛋白质中，预计只有不到 1% 是错误的鉴定。

### 1.3. 评估实验重复性 (Assess Reproducibility)
**状态**: 📋 进行中

**操作描述**:
重复性检验包括以下步骤
1. **蛋白质组与肽段数目重复性监测**：
    *   **逻辑** 从已经清洗的列表“df_filtered”中选择三列进行分析，分别是：
    样本名称列：“R.FileName”
    蛋白质组列：“PG.ProteinGroups”
    肽段列：“PEP.StrippedSequence”
    然后统计每组中的唯一肽段和唯一蛋白质组数目，然后进行统计作图
    *   **操作** 先找出所有组的名称，然后根据需求进行所有组的bar图分析或进行某组内的重复性检测，包括bar图和venn图绘制

2. **蛋白质组定量重复性检验**：
    *   **逻辑** 从原始列表中找出三列，作为一个新的定量性矩阵，分别是index='PG.ProteinGroups',  columns='R.FileName', values='PG.Quantity'后面对这个图进行热图分析、PCA分析和定量分布箱线图分析。



---

## 2. 阶段二：数据整理与归一化 (Data Wrangling & Normalization)
### 2.1. 构建定量矩阵 (Build Quantification Matrix)
**状态**: 📋 待办

**操作描述**:
<!-- 在这里详细解释你将如何构建定量矩阵... -->

### 2.2. 数据预处理 (Matrix Preprocessing)
**状态**: 📋 待办

**操作描述**:
<!-- 在这里详细解释你将如何对矩阵进行Log2转换和缺失值填充... -->

---

## 3. 阶段三：统计分析与可视化 (Statistical Analysis & Visualization)
### 3.1. 探索性数据分析 (Exploratory Data Analysis)
**状态**: 📋 待办

**操作描述**:
<!-- 在这里详细解释你将如何进行PCA和相关性分析... -->

### 3.2. 差异蛋白分析 (Differential Expression Analysis)
**状态**: 📋 待办

**操作描述**:
<!-- 在这里详细解释你将如何进行差异分析... -->

### 3.3. 核心结果可视化 (Key Results Visualization)
**状态**: 📋 待办

**操作描述**:
<!-- 在这里详细解释你将如何绘制火山图和热图... -->

### 阶段一：数据质控 (Data Quality Control)

- [√] **步骤 1.1: 数据导入** - 从 Spectronaut 的 `.xlsx` 报告中加载数据。
- [ ] **步骤 1.2: 数据清洗与过滤**
    - [√] 移除伪数据库匹配 (Decoys)。
    - [ ] 移除已知的污染物 (Contaminants)。
    - [ ] 基于 `PG.Qvalue < 0.01` 筛选高可信度的蛋白质。
- [ ] **步骤 1.3: 评估实验重复性**
    - [ ] 统计每个样本中鉴定到的蛋白/肽段数量。
    - [ ] 绘制条形图进行可视化比较。

### 阶段二：数据整理与归一化 (Data Wrangling & Normalization)

- [ ] **步骤 2.1: 构建定量矩阵** - 将长格式数据重塑为 "蛋白质 vs 样本" 的宽格式矩阵。
- [ ] **步骤 2.2: 数据预处理**
    - [ ] 对定量矩阵进行 Log2 转换。
    - [ ] 对缺失值 (`NaN`) 进行插补/填充。
- [ ] **步骤 2.3: 数据归一化** - (可选) 使用中位数归一化等方法消除样本间的系统误差。

### 阶段三：统计分析与可视化 (Statistical Analysis & Visualization)

- [ ] **步骤 3.1: 探索性数据分析 (EDA)**
    - [ ] 绘制主成分分析 (PCA) 图，查看样本聚类情况。
    - [ ] 绘制样本相关性热图。
- [ ] **步骤 3.2: 差异蛋白分析**
    - [ ] 定义实验分组（如：对照组 vs. 处理组）。
    - [ ] 计算 Log2 Fold Change 和 p-value (例如，通过 t-检验)。
- [ ] **步骤 3.3: 核心结果可视化**
    - [ ] 绘制火山图 (Volcano Plot) 以展示差异蛋白。
    - [ ] 绘制热图 (Heatmap) 展示显著差异蛋白在各样本中的表达模式。

### 阶段四：生物学功能注释 (Biological Interpretation)
<!-- 这是一个可选的未来步骤，可以在完成核心分析后进行 -->
- [ ] 对上调/下调的蛋白列表进行 GO / KEGG 功能富集分析。

## 如何使用

1.  将您的 Spectronaut 输出的 `.xlsx` 文件放入 `/data` 文件夹。
2.  打开 `/scripts` 文件夹中的主分析 Notebook (`.ipynb` 文件)。
3.  根据 Notebook 中的说明，修改文件路径等参数。
4.  按顺序运行 Notebook 中的所有代码块。

## 作者


- **[Jasper]**
- **联系邮箱**: 1256749831@qq.com
- **GitHub**: [@Jasper-delong](https://github.com/Jasper-delong)