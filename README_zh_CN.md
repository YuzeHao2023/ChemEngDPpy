# ChemEngDPpy
用于机械设计、尺寸计算及资本/运营成本（CAPEX/OPEX）估算的 Python 代码

本项目旨在将这些功能快速作为“黑箱”函数用于总拥有成本（TAC）优化中主设备的详细设计（例如：PFR 壁厚要求、泵/压缩机的尺寸等）。

- ***快速开始***：

  - 直接将所需的 `.py` 文件下载到你的工程路径；或

  - 使用 Git 或通过 SVN 使用该 Git URL 检出。
  
  - 然后，参见 `exampleruns.py` 获取使用示例（非常直观）
  
使用时，将所需的 `.py` 文件下载到与你现有代码相同的文件夹，并根据你的情形调用下列函数。

快速提示：

- 始终使用 docstring 检查所需单位以避免单位换算错误！

- 尽量使用具名参数以避免歧义！

------------------------------------------------

## 项目结构

- dsg：用于机械设计 & 辅助设备尺寸计算

- capex：用于 CAPEX 计算与报告

- opex：用于 OPEX 计算与报告

------------------------------------------------

## 实现方法

快速提示：始终使用 docstring 检查所需单位以避免单位换算错误，并尽量使用具名参数以避免歧义！

先说明几个定义：

- ***设计输入（Design inputs）***：对进行设备设计/尺寸计算至关重要并且是必须提供的属性

- ***设计变量（Design variables）***：在完成设备设计/尺寸计算后得到的属性

- ***附加标签（Additional tags）***：对设备设计/尺寸计算并非关键，但对成本估算很重要的属性。
这些可以在创建设备对象时一并提供，便于后续成本估算

例如：

- *压力* 对于容器而言是一个 *设计输入*（用于确定最终壳体厚度），
  对于泵/压缩机而言是一个 *设计变量*（用于确定功率需求），
  对于换热器而言则是一个 *附加标签*（用于确定压力因子）

- *体积* 对于容器而言是一个 *设计变量*，但对其他设备类别并不相关

- 操作温度对于容器而言是一个 *设计输入*（用于依据材料性能确定最终壳体厚度），
  对于换热器是一个 *设计输入*（用于确定换热面积），
  对于压缩机是一个 *设计变量*（具体为出口温度），
  对于泵则通常不相关（本库中的泵尺寸计算假设温度变化可以忽略）

本库提供两种主要使用方式：

1. ***方法一（标量框架）***：使用标量输入 - 标量输出，仅检索关键设计变量；或

2. ***方法二（面向对象框架，OOP）***：使用库的面向对象框架。
通常，常见化工设备（容器、泵、压缩机、换热器等）在方法二中也会创建为可选输出并作为各种函数的替代输入。

例如在设备设计中，可以使用：

```python
# 方法一（标量框架）
comppower, compeff, T2, _ = dsg.sizecompressor(m=1e5, P1=100, P2=300, T1=323.15, cp=1.02, cv=0.72, Z=0.99)
```

只检索所尺寸化的压缩机的功率额定、温度以及估算的绝热效率（如方法一），或者使用：

```python
# 方法二（OOP 框架）
_, _, _, K100 = dsg.sizecompressor(m=1e5, P1=100, P2=300, T1=323.15, cp=1.02, cv=0.72, Z=0.99,
etype='rotary', mat='CS', id='K400')
```

来创建一个 `Compressor()` 对象，该对象包含所有相关的设计输入（即输入参数）、设计变量（例如 `comppower`、`compeff` 和 `T2`）以及附加标签。

可以在创建设备对象时提供附加标签以便后续成本估算，例如：

- `category` 表示 **设备类别**（例如：泵、压缩机等 — 当分别调用相应的 `dsg.size(...)` 或 `dsg.design(...)` 函数时会自动创建 — 参见下文或代码内 docstring）

- `etype` 表示 **设备类型**（例如：离心泵、旋转式泵等 — 支持的设备类型见 `capex` 文档）

- `mat` 表示 **材料类型**（例如：碳钢、不锈钢、铸铁等 — 支持的材料类型见 `capex` 文档）

- `id` 用于指定 **设备名称**（便于语义化识别）

- `P` 用于为那些压力既不是必需设计输入也不是设计变量的设备指定设计操作压力。最明显的情况是换热器，此处的压力规格仅用于确定用于资本成本估算的压力因子 `FP`。

若干关键例外说明：

- 压力/真空容器的机械设计 *仅* 通过 OOP 框架（方法二）进行，因为机械设计涉及大量关键设计变量。

- 对于反应器和蒸馏塔，需分别设计压力/真空容器与内部件（例如：塔盘、搅拌器/叶轮、填料），**分开**进行（即作为独立对象），因为目前尚不支持将两者合并为单一对象（#TODO）。

- CAPEX 报告 *只能* 通过 OOP 框架（方法二）进行。

再举一个资本成本估算的例子，使用

1. 方法一（标量框架）：

```python
# 方法一（标量框架）
# 手动提供成本系数、有效范围的最小/最大容量（可选），
# 指数用于外推（可选）...
capex.eqptpurcost(A=20, Ktuple=(3.5565, 0.3776, 0.0905, 0.1, 628., 0.5))
# ... 或者直接从 capex.eqptcostlib 中检索
capex.eqptpurcost(A=20, Ktuple=capex.eqptcostlib['vessel']['horizontal'])
```

对体积 `A` = 20 m^3 的水平容器进行采购成本估算，然后调用后续的所有函数（见 `capex` 文档）。但是，使用

2. 方法二（OOP 框架）：

```python
# 方法二（OOP 框架）
capex.eqptpurcost(eqpt=V100)
```

其中 `V100` 是由 `dsg.designvertpres` 返回的（在此上下文中为 `MechDesign()` 对象），会更简单方便。

实际上，使用 OOP 框架（方法二）时，我们可以仅创建所有相关设备对象的列表并直接进行整个资本成本估算：

```python
eqptlist = [V100, V200, V300, K400, K500, P600, P700, HX800, HX900]
FCI, capexreport = capex.econreport(eqptlist, planttype='green', pbp=3, year=2019, currency='SGD', \
                                    reporttype='numpy', verbose=True)
```

详见 `exampleruns.py` 获取更多示例用法。

OOP 框架（方法二）的其它可能用途包括：

- 将设备类型或材料选择作为分类决策变量进行优化，以最小化资本成本（capex），使用 `etype`、`mat` 等标签；

- 通过最小化 capex 与 opex 来确定最佳的换热器类型与公用事业配置。这与上例类似，公用事业选择作为额外的分类决策变量，会影响换热面积与公用事业成本；

- 在多级压缩系统中寻找最优配置以最小化电耗与流股冷却（例如：压缩机数量、压缩比、冷却的换热面积等作为决策变量）。

但这些实现留给用户自行完成（#TODO 添加示例）。

------------------------------------------------

## 机械设计与辅助设备尺寸计算 - dsg

常量：

- `dsg.Patm` = 14.696 - 标准大气压（psi）

- `dsg.Patmb` = 1.01325 - 标准大气压（bar）

- `dsg.Troom` = 77 - 环境温度（°F）

- `dsg.tmin` = 1/4 - 通用最小允许容器厚度（in）

- `dsg.tc` = 0.125 - 腐蚀余量（in），默认对腐蚀/非腐蚀条件均为 1/8 in

- `dsg.rhosteel` = 0.2836 - SA-285C/SA-387B/碳钢/低合金钢的密度（lb/in^3）

- `dsg.g` = 9.80665 - 标准重力加速度（m/s^2）

- `dsg.R` = 8.31446261815324 - 通用理想气体常数（J/(K·mol)）

- `dsg.Ta` = 10. - 换热器最小温差（K）

函数：

- `dsg.designhorzpres` - 完成水平压力容器的全部机械设计

- `dsg.designvertpres` - 完成垂直压力容器的全部机械设计

- `dsg.designvac` - 完成真空容器的全部机械设计

- `dsg.sizecompressor` - 压缩机尺寸计算

- `dsg.sizepump` - 泵的尺寸计算

- `dsg.sizeHE_heater` - 加热工艺的换热器尺寸计算

- `dsg.sizeHE_cooler` - 冷却工艺的换热器尺寸计算

若要调用中间函数（例如计算壳体厚度、计算允许最大应力、计算风载等），请参考代码内的文档说明（docstrings）。

------------------------------------------------

## 资本支出（CAPEX）计算 - capex

常量：

- `capex.CEPCI[20yy]` - 获取 20yy 年的 CEPCI 指数（其中 yy = 01、18 或 19）

- `capex.USSG[20yy]` - 获取 20yy 年度平均 USD:SGD 汇率（其中 yy = 01、18 或 19）

- `capex.CPI['zz'][20yy]` - 获取国家 zz 在 20yy 年的消费者价格指数（CPI）年度平均值
  （其中 yy = 01、16、18 或 19，zz = 'SG' 或 'US'）

  - CAPCOST 的参考年份为 2001（依据 Turton 等，第五版）

  - 公用事业成本的参考年份为 2016（依据 Turton 等，第五版）
  
- `capex.eqptcostlib['eqptcategory']['eqpttype']` - 检索设备成本相关参数元组
  `(K1, K2, K3, Amin, Amax, n)`，其中 `(K1, K2, K3)` 为成本相关系数，`(Amin, Amax)` 为有效容量范围（适用范围），`n` 为用于外推的成本指数，
  可由此计算采购设备成本 `Cpo`。

- `capex.pressurefaclib['eqptcategory']['eqpttype']` - 检索设备压力因子相关参数元组
  `(C1, C2, C3, Pmin, Pmax)`，其中 `(C1, C2, C3)` 为压力因子相关系数，`(Pmin, Pmax)` 为适用的压力范围，从而计算压力因子 `FP`。

- `capex.matfaclib['eqptcategory']['eqpttype']['mat']` - 检索设备材料因子 `FM`（浮点数）

- `capex.baremodlib['eqptcategory']['eqpttype']` - 检索裸模块系数相关参数元组 `(B1, B2)`，使得裸模块系数 `FBM = B1 + B2 * FP * FM`。

支持的设备类别、类型与材料（`mat`）：
 
 - `compressor`
   - `centrifugal`, `axial`, `reciprocating`, `rotary`
       - `CS`（碳钢）
       - `SS`（不锈钢）
       - `Ni`（镍合金）
   
 - `pump`
    - `reciprocating`, `positivedisp`
        - `Fe`（铸铁）
        - `CS`
        - `SS`
        - `Ni`
        - `Ti`（钛合金）
    - `centrifugal`
        - `Fe`
        - `CS`
        - `SS`
        - `Ni`
       
 - `heatexc`
    - `fixedtube`, `utube`, `kettle`, `doublepipe`, `multipipe`
       - `CS/CS`
       - `CS/SS` 与 `SS/CS`（壳程/管程顺序不影响）
       - `SS/SS`
       - `CS/Ni` 与 `Ni/CS`
       - `Ni/Ni`
       - `CS/Ti` 与 `Ti/CS`
       - `Ti/Ti`
       
 - `vessel`
    - `horizontal`, `vertical`
        - `CS`
        - `SS`
        - `Ni`
        - `Ti`
        
 - `trays`
    - `sieve`, `valve`
        - `CS`
        - `SS`
        - `Ni`
    - `demister`
        - `SS`
        - `FC`（氟碳材料）
        - `Ni`

函数：

- `capex.eqptpurcost` - 计算采购设备成本（`Cpo`）

- `capex.pressurefacves` - 计算容器的压力因子（`FP`）

- `capex.pressurefacanc` - 计算辅助设备的压力因子（`FP`）

- `capex.baremodfac` - 计算裸模块系数（`FBM`）

- `capex.baremodcost` - 计算裸模块成本（`CBM`）

- `capex.totmodcost` - 计算模块总成本（`CTM`）

- `capex.grasscost` - 计算草创成本（grassroots cost，`CGR`）

- `capex.annualcapex` - 基于假定的回收期（pbp）计算年化资本支出（`ACC`）。如果指定了 pbp，请注意该值仅应在用于优化估算 ACC 时使用！或者，可以基于预计收入来计算 pbp。

------------------------------------------------

## 运营支出（OPEX）计算 - opex

常量：

- `opex.SF` - 工艺流因子（stream factor）

- `opex.runtime` - 年运行时间

- `opex.shiftdur` - 单班时长

- `opex.shiftperweek` - 每年班次数（注：原字段名可能表示每周班次）

- `opex.yearww` - 扣除休假后的年工作周数

- `opex.util['xxx']` - 各类公用事业成本（以 USD/GJ 为基准），其中 xxx 包括：

  - `"HPS"` 表示高压蒸汽（41 barg, 254 °C）

  - `"MPS"` 表示中压蒸汽（10 barg, 184 °C）

  - `"LPS"` 表示低压蒸汽（5 barg, 160 °C）

  - `"CW"` 表示冷却水（30–45 °C）

  - `"ChW"` 表示冷冻水（5 °C）

  - `"LTR"` 表示低温制冷剂（-20 °C）

  - `"VLTR"` 表示极低温制冷剂（-50 °C）

  - `"elec"` 表示电力（110–440 V）

函数：

- `opex.labourcost` - 计算年化人工成本（`COL`）

- `opex.costofraw` - 计算年化原料成本（`CRM`）

- `opex.costofutil` - 计算年化公用事业成本（`CUT`）

- `opex.costofwaste` - （占位函数，供用户自定义废弃物处理成本计算）

- `opex.costofmanfc` - 计算年化制造总成本（`COM`）的所有组成部分

若要调用中间函数（例如计算每班操作人数等），请参考代码内的文档说明（docstrings）。
