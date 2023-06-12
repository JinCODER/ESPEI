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

Non-equilibrium thermochemical data is used where the internal degrees of freedom for a phase are known. This type of data is the only data that can be used for parameter generation, but it can also be used in Bayesian parameter estimation.

Two examples follow. The first dataset has some data for the formation heat capacity for BCC_B2.

* The ``components`` and ``phases`` keys simply describe those found in this entry.
* Use the ``reference`` key for bookkeeping the source of the data.
* The ``comment`` key and value can be used anywhere in the data to keep notes for your reference. It takes no effect.
* The ``solver`` the internal degrees of freedom and and site ratios are described for the phase.

   ``sublattice_configurations`` is a list of different configurations, that should correspond to the sublattices for the phase descriptions.
   Non-mixing sublattices are represented as a string, while mixing sublattices are represented as a lists.
   Thus an endmember for ``BCC_B2`` (as in this example) is ``["AL", "NI", VA"]`` and if there were mixing (as in the next example) it might be ``["AL", ["AL", "NI"], "VA"]``.
   Mixing also means that the ``sublattice_occupancies`` key must be specified, but that is not the case in this example.
   It is important to note that any mixing configurations must have any ideal mixing contributions removed.
   Regardless of whether there is mixing or not, the length of this list should always equal the number of sublattices in the phase, though the sub-lists can have mixing up to the number of components in that sublattice.
   Note that the ``sublattice_configurations`` is a *list* of these lists.
   That is, there can be multiple sublattice configurations in a single dataset.
   See the second example in this section for such an example.

* The ``conditions`` describe temperatures (``T``) and pressures (``P``) as either scalars or one-dimensional lists.
* The type of quantity is expressed using the ``output`` key. This can in principle be any thermodynamic quantity, but currently only ``CPM*``, ``SM*``, and ``HM*`` (where ``*`` is either nothing, ``_MIX`` or ``_FORM``) are supported. Support for changing reference states is planned but not yet implemented, so all thermodynamic quantities must be formation quantities (e.g. ``HM_FORM`` or ``HM_MIX``, etc.). This is tracked by :issue:`85` on GitHub.
* ``values`` is a 3-dimensional array where each value is the ``output`` for a specific condition of pressure, temperature, and sublattice configurations from outside to inside. Alternatively, the size of the array must be ``(len(P), len(T), len(subl_config))``. In the example below, the shape of the ``values`` array is (1, 12, 1) as there is one pressure scalar, one sublattice configuration, and 12 temperatures.
* There is also a key, ``excluded_model_contributions``, which will make those contributions of pycalphad's ``Model`` not be fit to when doing parameter selection or MCMC. This is useful for cases where the type of data used does not include some specific ``Model`` contributions that parameters may already exist for. For example, DFT formation energies do not include ideal mixing or (CALPHAD-type) magnetic model contributions, but formation energies from experiments would include these contributions so experimental formation energies should not be excluded.

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


In the second example below, there is formation enthalpy data for multiple sublattice configurations.
All of the keys and values are conceptually similar.
Here, instead of describing how the ``output`` quantity changes with temperature or pressure, we are instead only comparing ``HM_FORM`` values for different sublattice configurations.
The key differences from the previous example are that there are 9 different sublattice configurations described by ``sublattice_configurations`` and ``sublattice_occupancies``.
Note that the ``sublattice_configurations`` and ``sublattice_occupancies`` should have exactly the same shape.
Sublattices without mixing should have single strings and occupancies of one.
Sublattices that do have mixing should have a site ratio for each active component in that sublattice.
If the sublattice of a phase is ``["AL", "NI", "VA"]``, it should only have two occupancies if only ``["AL", "NI"]`` are active in the sublattice configuration.

The last difference to note is the shape of the ``values`` array.
Here there is one pressure, one temperature, and 9 sublattice configurations to give a shape of (1, 1, 9).

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

Equilibrium thermochemical data is used when the internal degrees of freedom are not known. This is typically true for experimental thermochemical data. Some cases where this type of data is useful, compared to non-equilibrium thermochemical data are:

1. Activity data
#. Enthalpy of formation data in region with two or more phases in equilibrium
#. Enthalpy of formation for a phase with multiple sublattice, e.g. the σ phase


This type of data can not be used in parameter selection, because a core assumption of parameter selection is that the site fractions are known.


.. note::

  Only activity data is supported at the moment. Support for other data types is tracked by :issue:`104`.

Activity data is similar to non-equilibrium thermochemical data, except we must enter a reference state and the ``solver`` key is no longer required, since we do not know the internal degrees of freedom. A key detail is that the ``phases`` key must specify all phases that are possible to form.

An example for Mg activties in Cu-Mg follows, with data digitized from S.P. Garg, Y.J. Bhatt, C. V. Sundaram, Thermodynamic study of liquid Cu-Mg alloys by vapor pressure measurements, Metall. Trans. 4 (1973) 283–289. doi:10.1007/BF02649628.

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
