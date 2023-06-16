.. raw:: latex

   \chapter{Making ESPEI datasets}

.. _Input data:


===================
构建ESPEI数据集
===================

JSON格式
===========

ESPEI使⽤JSON格式的单⼀输入样式来进⾏所有数据录入。
对于不熟悉JSON的⼈来说，它与Python字典非常相似，但有⼀些严格的要求

	•	所有的字符串引号必须是双引号。使用 ``"key"`` 而不是 ``'key'``。
	•	数字不能有前导零。 ``00.123`` 应该写成 ``0.123`` 并且 ``012.34`` 必须写成 ``12.34``。
	•	列表和嵌套的键值对不能有尾逗号。 ``{"nums": [1,2,3,],}`` 是无效的，应该写成 ``{"nums": [1,2,3]}``。

这些错误在Python中的JSON错误消息中可能很难追踪到。
 建议使⽤可视化编辑器来调试JSON文件，如 `JSONLint`_。
可以在此处快速地找到json格式参考 `Learn JSON in Y minutes <https://learnxinyminutes.com/docs/json/>`_。

ESPEI⽀持检查所有输入数据集的错误，在运⾏ESPEI之前应始终使⽤该功能。
此错误检查将⼀次性报告所有错误，所有错误都应该修复。
数据集中的错误将阻⽌拟合。
通过运行 ``espei --check-datasets my-input-data`` 来检查路径 ``my-input-data/`` 中的数据集。

.. _JSONLint: https://jsonlint.com

.. _input_phase_descriptions:

描述相
==================

描述CALPHAD相的JSON文件在概念上类似于Thermo-Calc的PARROT模块中的setup文件。
文件顶部有一个 ``refdata`` 键， ，⽤于描述您想要选择的参考态。
当前的参考态是指向 ``pycalphad.refdata`` 字典中的字典的字符串，目前只实现了 ``"SGTE91"``。

每个相都使⽤相名称作为字典的键来进⾏描述。
该相的详细信息是该键的值的字典。
描述相有4个可能的条⽬： ``sublattice_model``， ``sublattice_site_ratios``， ``equivalent_sublattices``和 ``aliases``。
``sublattice_model`` 是一个列表的列表, 其中每个内部列表包含该亚点阵的所有组分。
``BCC_B2`` 的亚点阵模型是  ``[["AL", "NI", "VA"], ["AL", "NI", "VA"], ["VA"]]``， 因此有三个亚点阵，前两个有Al、Ni和空位。
``sublattice_site_ratios`` 应该与亚点阵模型长度相同 （例. ``BCC_B2`` 的 ``sublattice_site_ratios`` 为3）。
亚点阵的站点比例可以是分数或整数，不必总和为1。

可选的 ``equivalent_sublattices`` 键是描述哪些亚点阵在对称上是等效的列表的列表。
``equivalent_sublattices`` 中的每个子列表描述了等效的亚点阵的索引（从零开始）。
对于 ``BCC_B2`` ，等效亚点阵是 ``[[0, 1]]``， 这意味着索引为0和1的亚点阵是等效的。
在⼀个亚点阵中可以有多个不同的等效亚点阵集合（多个⼦列表），并且在⼀个亚点阵中可以有多个等效亚点阵 （参见 ``FCC_L12``）。
如果不存在 ``equivalent_sublattice`` 键，则假定没有等效亚点阵。

最后， ``aliases`` 键⽤于当考虑对称性时引⽤此亚点阵模型可以描述的其他相。
这里使用Aliases来描述 ``BCC_A2`` 和 ``FCC_A1``，它们分别是 ``BCC_B2`` 和 ``FCC_L12`` 的无序相。
注意aliased相在输入文件中没有其他描述。
多个相可以有相同的aliases相，例 ``FCC_L12`` 和 ``FCC_L10`` 同时可以将 ``FCC_A1`` 作为alias。

.. code-block:: JSON

    {
      "refdata": "SGTE91",
      "components": ["AL", "NI", "VA"],
      "phases": {
          "LIQUID" : {
          "sublattice_model": [["AL", "NI"]],
          "sublattice_site_ratios": [1]
          },
          "BCC_B2": {
          "aliases": ["BCC_A2"],
          "sublattice_model": [["AL", "NI", "VA"], ["AL", "NI", "VA"], ["VA"]],
          "sublattice_site_ratios": [0.5, 0.5, 1],
          "equivalent_sublattices": [[0, 1]]
          },
          "FCC_L12": {
          "aliases": ["FCC_A1"],
          "sublattice_model": [["AL", "NI"], ["AL", "NI"], ["AL", "NI"], ["AL", "NI"], ["VA"]],
          "sublattice_site_ratios": [0.25, 0.25, 0.25, 0.25, 1],
          "equivalent_sublattices": [[0, 1, 2, 3]]
          },
          "AL3NI1": {
          "sublattice_site_ratios": [0.75, 0.25],
          "sublattice_model": [["AL"], ["NI"]]
          },
          "AL3NI2": {
          "sublattice_site_ratios": [3, 2, 1],
          "sublattice_model": [["AL"], ["AL", "NI"], ["NI", "VA"]]
          },
          "AL3NI5": {
          "sublattice_site_ratios": [0.375, 0.625],
          "sublattice_model": [["AL"], ["NI"]]
          }
        }
    }


单位
=====

- 能量单位为J/mol-atom（其导数也是如此）
- 所有的组分都为摩尔分数
- 温度单位为Kelvin
- 压力单位为Pascal

.. _non_equilibrium_thermochemical_data:

非平衡热化学数据
===================================

非平衡热化学数据⽤于已知相的内部⾃由度的情况。这种类型的数据是⽣成参数的唯⼀数据，但也可以在⻉叶斯参数估计中使⽤。

以下是两个⽰例。第⼀个数据集中包含了⼀些BCC_B2的形成热容数据。

* ``components`` 和 ``phases`` 键仅描述了该条⽬中包含的组分和相。
* 使用 ``reference`` 键记录数据的来源信息。
* ``comment`` 键和值可在数据的任何位置⽤于保留您的参考注释，它不会产⽣任何影响。
* ``solver`` 描述了相的内部⾃由度和站点比例。

   ``sublattice_configurations`` 是⼀个不同配置的列表，应与相描述中的亚点阵对应。
   非混合亚点阵表⽰为字符串，⽽混合亚点阵表⽰为列表。
   因此， ``BCC_B2`` 的端元（如本示例）是 ``["AL", "NI", VA"]`` ，如果有混合（如下一个示例）可能是 ``["AL", ["AL", "NI"], "VA"]``。
   混合还意味着必须指定 ``sublattice_occupancies`` 键，但本⽰例中不需要。
   重要的是，所有混合的配置都必须删除理想混合贡献。
   ⽆论是否存在混合，该列表的⻓度始终应与相中的亚点阵相符，尽管⼦列表可以在该亚格中具有与该亚点阵中的组分数相同的混合。
   注意， ``sublattice_configurations`` 是这些列表的列表。 
   也就是说，单个数据集中可以有多个亚点阵配置。
   请参阅本节的第⼆个⽰例。

* ``conditions`` 描述了温度(``T``)和压强(``P``) ，这些可以是标量或一维列表。
* 使用 ``output`` 键来表⽰数量的类型。这理论上可以是任何热⼒学量，但⽬前仅⽀持 ``CPM*`` ， ``SM*`` ， ``HM*`` （其中 ``*`` 可以是空， ``_MIX`` 或 ``_FORM``）。更新计划中拟将支持更改参考状态，因此所有热⼒学量都必须是形成量（例 ``HM_FORM`` 或者 ``HM_MIX`` 等）。 这在Github的 issue:`85` 有记录。
* ``values``是⼀个三维数组，其中每个值是来⾃外到内的特定压⼒、温度和亚格配置的输出。或者，数组的⼤⼩必须为 ``(len(P), len(T), len(subl_config))``。在下⾯的⽰例中， ``values`` 数字形状为 (1, 12, 1) ），其含义是有⼀个压⼒标量，⼀个亚点阵配置和12个温度。
* 还有⼀个 ``excluded_model_contributions`` 键， 当进⾏参数选择或MCMC时，将不对这些贡献的pycalphad的模型进⾏拟合。这对于所使⽤的数据类型不包括某些特定模型贡献的情况非常有⽤，⽽参数
可能已经存在于这些贡献中。例如，DFT形成能量不包括理想混合或（CALPHAD类型的）磁转变贡献，实验形成能会包括这些贡献，因此不应排除实验形成能。

.. code-block:: JSON

    {
      "reference": "Yi Wang et al 2009",
      "components": ["AL", "NI", "VA"],
      "phases": ["BCC_B2"],
      "solver": {
        "mode": "manual",
	      "sublattice_site_ratios": [0.5, 0.5, 1],
	      "sublattice_configurations": [["AL", "NI", "VA"]],
	      "comment": "NiAl sublattice configuration (2SL)"
      },
      "conditions": {
	      "P": 101325,
	      "T": [  0,  10,  20,  30,  40,  50,  60,  70,  80,  90, 100, 110]
      },
      "excluded_model_contributions": ["idmix", "mag"],
      "output": "CPM_FORM",
      "values":   [[[ 0      ],
                    [-0.0173 ],
                    [-0.01205],
                    [ 0.12915],
                    [ 0.24355],
                    [ 0.13305],
                    [-0.1617 ],
                    [-0.51625],
                    [-0.841  ],
                    [-1.0975 ],
                    [-1.28045],
                    [-1.3997 ]]]
    }


在下⾯的第⼆个⽰例中，存在多个亚晶格配置的形成焓数据。
所有的键和值在概念上是相似的。
在这⾥，我们只比较不同亚点阵配置下的 ``HM_FORM`` 值，而不描述 ``output`` 量随温度或压强的变化。
与前⼀个⽰例相比，主要的区别在于通过 ``sublattice_configurations`` 和 ``sublattice_occupancies`` 描述了9种不同的亚点阵配置。
请注意， ``sublattice_configurations`` 和 ``sublattice_occupancies`` 应该具有完全相同的形状。
没有混合的亚点阵应该只有单个字符串和占有率为1。
具有混合的亚点阵应该对该亚点阵中的每个激活的组分有⼀个位点比例。
如果⼀个相的亚点阵是 ``["AL", "NI", "VA"]`` ，那么只有在亚点阵配置中只有 ``["AL", "NI"]`` 是激活的情况下，他才应该具有两个点位占有率。

最后需要注意的是 ``values`` 数组的形状。
在这⾥，有⼀个压⼒，⼀个温度和9个亚点阵配置，形状为 (1, 1, 9)。

.. code-block:: JSON

    {
      "reference": "C. Jiang 2009 (constrained SQS)",
      "components": ["AL", "NI", "VA"],
      "phases": ["BCC_B2"],
      "solver": {
	      "sublattice_occupancies": [
				         [1, [0.5, 0.5], 1],
				         [1, [0.75, 0.25], 1],
				         [1, [0.75, 0.25], 1],
				         [1, [0.5, 0.5], 1],
				         [1, [0.5, 0.5], 1],
				         [1, [0.25, 0.75], 1],
				         [1, [0.75, 0.25], 1],
				         [1, [0.5, 0.5], 1],
				         [1, [0.5, 0.5], 1]
				        ],
	      "sublattice_site_ratios": [0.5, 0.5, 1],
	      "sublattice_configurations": [
				            ["AL", ["NI", "VA"], "VA"],
				            ["AL", ["NI", "VA"], "VA"],
				            ["NI", ["AL", "NI"], "VA"],
				            ["NI", ["AL", "NI"], "VA"],
				            ["AL", ["AL", "NI"], "VA"],
				            ["AL", ["AL", "NI"], "VA"],
				            ["NI", ["AL", "VA"], "VA"],
				            ["NI", ["AL", "VA"], "VA"],
				            ["VA", ["AL", "NI"], "VA"]
				           ],
	      "comment": "BCC_B2 sublattice configuration (2SL)"
      },
      "conditions": {
	      "P": 101325,
	      "T": 300
      },
      "output": "HM_FORM",
      "values":   [[[-40316.61077, -56361.58554,
	             -49636.39281, -32471.25149, -10890.09929,
	             -35190.49282, -38147.99217, -2463.55684,
	             -15183.13371]]]
    }

平衡热化学数据
===============================

当内部⾃由度未知时，使⽤平衡热化学数据。这通常适⽤于实验热化学数据。与非平衡热化学数据相比，以下情况下使⽤此类数据更为有效：

1. 活度数据
#. 在存在两个或多个平衡相的区域种的形成焓数据
#. 具有多个亚点阵的相的形成焓，例如 σ 相


这类数据不能⽤于参数选择，因为参数选择的核⼼假设是已知位点分数。


.. note::

  目前只支持活度数据。对其他类型的数据的支持开发进度请见 :issue:`104`

活度数据与非平衡热化学数据类似，唯⼀的区别是我们需要输入参考态，并且不再需要 ``solver`` 键，因为我们不知道自由度。其中有一个细节很关键 ``phases`` 键必须指定所有可能形成的相。

以下是关于Cu-Mg种Mg活度的示例，数据来源于：S.P. Garg, Y.J. Bhatt, C. V. Sundaram, Thermodynamic study of liquid Cu-Mg alloys by vapor pressure measurements, Metall. Trans. 4 (1973) 283–289. doi:10.1007/BF02649628.

.. code-block:: JSON

    {
      "components": ["CU", "MG", "VA"],
      "phases": ["LIQUID", "FCC_A1", "HCP_A3"],
      "reference_state": {
        "phases": ["LIQUID"],
        "conditions": {
          "P": 101325,
          "T": 1200,
          "X_MG": 1.0
        }
      },
      "conditions": {
        "P": 101325,
        "T": 1200,
        "X_CU": [0.9, 0.8, 0.7, 0.6, 0.5, 0.4, 0.3, 0.2, 0.1, 0.0]
      },

      "output": "ACR_MG",
        "values":   [[[0.0057,0.0264,0.0825,0.1812,0.2645,0.4374,0.5852,0.7296,0.882,1.0]]],
      "reference": "garg1973thermodynamic",
      "comment": "Digitized Figure 3 and converted from activity coefficients."
    }

.. _phase_boundary_data:

相图数据
==================

ESPEI can consider multi-component phase diagram data with an arbitrary number of phases in equilibrium.
Phase diagram data JSON datasets are distingished by using ``"output": "ZPF"`` [1]_.
Each entry in the JSON ``values`` corresponds to a *phase region* where one or
more phases are participating in equilibrium under the given temperature and
pressure conditions.

Each phase in the phase region must give its *phase composition*, i.e. the
internal composition of that phase (*not* the overall composition).
The "phase composition" is the same as a "tie-line composition" in a two-phase
region of a binary phase diagram, but is a more general term for cases where
the meaning of a tie-line is ambiguous like a single phase equilibrum or an
equilibrium with three or more phases.

Sometimes there may be a phase equilibrium where one or more of the phase
compositions are unknown. This is especially common for phase diagram data
determined by equilibrated alloys or by scanning calorimetry in binary systems,
where one phase composition is determined, but the phase composition of the
other phase(s) in equilibrium are not. In these cases, phase compositions can
be given as ``null`` and ESPEI will estimate the phase composition.

.. admonition:: Important
   :class: important

   Each phase region must have at least one phase with a prescribed phase composition.
   If all phases in a phase region have ``null`` phase compositions, the
   *target hyperplane* (described by Figure 1 in [Bocklund2019]_)
   will be undefined and no driving forces will be computed.

.. admonition:: Important
   :class: important

   For a dataset with ``c`` components, each phase composition must be specified by ``c-1`` components.
   There is an implicit ``N=1`` condition.

Example
-------

.. code-block:: JSON

   {
     "components": ["AL", "NI"],
     "phases": ["AL3NI2", "BCC_B2", "LIQUID"],
     "conditions": {
       "P": 101325,
        "T": [2500, 1348, 1176, 977]
     },
     "output": "ZPF",
     "values": [
       [["LIQUID", ["NI"], [0.5]]],
       [["AL3NI2", ["NI"], [0.4083]], ["BCC_B2", ["NI"], [0.4340]]],
       [["AL3NI2", ["NI"], [0.4114]], ["BCC_B2", ["NI"], [null]]],
       [["BCC_B2", ["NI"], [0.71]], ["LIQUID", ["NI"], [0.752]], ["FCC_L12", ["NI"], [0.76]]]
     ],
     "reference": "37ALE"
   }

Each entry in the ``values`` list is a list of all phases in equilibrium in a phase region.
There are four phase regions:

``[["LIQUID", ["NI"], [0.5]]]``
   Single phase equilibrium with ``LIQUID`` having a phase composition of ``X(NI,LIQUID)=0.5``.

``[["AL3NI2", ["NI"], [0.4083]], ["BCC_B2", ["NI"], [0.4340]]]``
   Two phase equilibrium between ``AL3NI2`` and ``BCC_B2``, which have phase compositions of ``X(NI,AL3NI2)=0.4083`` and ``X(NI,BCC_B2)=0.4340``, respectively.

``[["AL3NI2", ["NI"], [0.4114]], ["BCC_B2", ["NI"], [null]]]``
   Two phase equilibrium between ``AL3NI2`` and ``BCC_B2`` where the phase composition of ``BCC_B2`` is unknown.

``[["BCC_B2", ["NI"], [0.71]], ["LIQUID", ["NI"], [0.752]], ["FCC_L12", ["NI"], [0.76]]]``
   Eutectic reaction between ``LIQUID``, ``BCC_B2`` and ``FCC_L12``.

.. admonition:: Tip: Multi-component phase regions
   :class: Tip

   To describe multi-component phase regions, simply include more components and compositions in each phase composition.
   For example, a two-phase equilibrium in a three component system could be described by
   ``[["ALPHA", ["CR", "NI"], [0.1, 0.25]], ["BETA", ["CR", "NI"], [null, null]]]``

.. _Datasets Tags:

Tags键
====

Tags are a flexible method to adjust many ESPEI datasets simultaneously and drive them via the ESPEI's input YAML file.
Each dataset can have a ``"tags"`` key, with a corresponding value of a list of tags, e.g. ``["dft"]``.
Any tag modifications present in the input YAML file are applied to the datasets before ESPEI is run.

They can be used in many creative ways, but some suggested ways include to add weights or to exclude model contributions, e.g. for DFT data that should not have contributions for a CALPHAD magnetic model or ideal mixing energy.
An example of using the tags in an input file looks like:

.. code-block:: JSON

   {
     "components": ["CR", "FE", "VA"],"phases": ["BCC_A2"],
     "solver": {"mode": "manual", "sublattice_site_ratios": [1, 3],
                "sublattice_configurations": [[["CR", "FE"], "VA"]],
     "sublattice_occupancies": [[[0.5, 0.5], 1.0]]},
     "conditions": {"P": 101325, "T": 300},
     "output": "HM_MIX",
     "values": [[[10000]]],
     "tags": ["dft"]
   }


An example input YAML looks like

.. code-block:: YAML

   system:
     phase_models: CR-FE.json
     datasets: FE-NI-datasets-sep
     tags:
       dft:
         excluded_model_contributions: ["idmix", "mag"]

   generate_parameters:
     excess_model: linear
     ref_state: SGTE91
     ridge_alpha: 1.0e-20
   output:
     verbosity: 2
     output_db: out.tdb

This will add the key ``"excluded_model_contributions"`` to all datasets that have the ``"dft"`` tag:

.. code-block:: JSON

   {
     "components": ["CR", "FE", "VA"],"phases": ["BCC_A2"],
     "solver": {"mode": "manual", "sublattice_site_ratios": [1, 3],
                "sublattice_configurations": [[["CR", "FE"], "VA"]],
     "sublattice_occupancies": [[[0.5, 0.5], 1.0]]},
     "conditions": {"P": 101325, "T": 300},
     "output": "HM_MIX",
     "values": [[[10000]]],
     "excluded_model_contributions": ["idmix", "mag"]
   }


常见错误与提示
=========================

1. A single element sublattice is different in a phase model (``[["A", "B"], ["A"]]]``) than a sublattice configuration (``[["A", "B"], "A"]``).
#. Make sure you use the right units (``J/mole-atom``, mole fractions, Kelvin, Pascal)
#. Mixing configurations should not have ideal mixing contributions.
#. All types of data can have a ``weight`` key at the top level that will weight the standard deviation parameter in MCMC runs for that dataset. If a single dataset should have different weights applied, multiple datasets should be created.


.. [1] ``ZPF`` after the "Zero Phase Fraction" method [Bocklund2019]_ used to compute the likelihood. "Zero phase fraction" is a little misleading as a name, since the prescribed phase compositions in the datasets actually correspond to the overall composition where the phase fraction of the desired phase should be *one*.
