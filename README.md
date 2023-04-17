# stressed

This repository hosts the code used to enumerate stressed words in the article ["Sub-Fibonacci behavior in numerical semigroup enumeration"](https://arxiv.org/abs/2202.05755), where a mathematical description of the relevant problem can be found (see in particular Sections 3 and 7). The code consists of two files: `engine.cpp` and `stressed_parser.py`, and we will describe both briefly below.

## `engine.cpp`

This program represents the main workhorse of the algorithm. As currently written, it relies on GCC builtins, global x64 register variables, and the POPCNT instruction, so both x64 and SSE4 are required to run it. On my machine (Intel i7-8550U), I have used `g++ -O2 -march=native`, but I would not be surprised if there are better options.

The program takes in two integer arguments `n >= 1` and `l >= 0`, where `l` defaults to `0` if not provided. Then, it iterates through all subsets `A` of `{1, 2, ..., n-1}` of size `s` at least `l` such that `n` is not an element of `A + A`. For each such subset,  it records `t`, the number of elements in `{1, 2, ..., n-1}` that are neither in `A` nor in `A + A`. Its output is a `floor((n+1)/2) × n` matrix of integers, where the entry in the `i`th row and the `j`th column (both zero-indexed) is the number of subsets `A` with `s = i` and `t = j`.

I will not discuss implementation in detail here (though I do not envy anyone trying to understand the code for the first time), but there are two important facts that the user should know. First of all, for a given `n` with `l = 0`, there are exactly `3^floor((n-1)/2)` subsets to iterate through, so the running time is exponential in `n`. Also, in this program subsets are represented using 64-bit integers, so **as `n` approaches `64` something will eventually break**. I believe that the code should work up to and including `n = 65`, but I cannot guarantee this. Extending this program to `n > 65` without sacrificing too much speed would be an interesting task.

To both save the user from having to recompute values and serve as a reference, I have included all the outputs of the program needed for the paper in the `output` directory. The filenames refer to the arguments and are produced by `stressed_parser.py`. For benchmarking purposes, I have also included a log of runtimes produced by `stressed_parser.py` as `runtime_log`. One can see that across a wide range of inputs, the speed on my machine is roughly 1.6 billion subsets per second, or roughly 2 clock cycles per subset.

## `stressed_parser.py`

This script is a wrapper around the C++ program meant to produce usable results. Here is a brief description of the functions:

* `runengine(n, l)`: Calls the engine with the given arguments and stores the output in the correpsonding file.

* `runall()`: Makes all the necessary engine calls to produce the data in the paper.

* `loaddata(ddir)`: Reads engine output from output files in the given directory.

* `count_stressed_n(n)`: Returns a list `l` of length `3n+1` where `l[i]` is the number of stressed words of genus `i` and length `n`.

* `count_stressed_g(maxg)`: Returns a list `l` of length `maxg+1` where `l[i]` is the number of stressed words of genus `i`.

* `compute()`: Outputs all the data present in the paper. Specifically, it prints
  
  * The number of stressed words of every length from 0 to 56. This data is used in the article ["Counting numerical semigroups by Frobenius number, multiplicity, and depth"](https://arxiv.org/abs/2208.14587) by Sean Li.
  
  * The partial sums of `sum_(w stressed) phi^(-g(w))` with maximum lengths ranging from 0 to 56.
  
  * The number of stressed words of every genus from 0 to 95. 

For reference, I will reproduce the output of the `compute()` function below:

```
[0, 1, 2, 7, 14, 50, 96, 343, 667, 2249, 4513, 15349, 28897, 100425, 197268, 636988, 1264258, 4215132, 8051166, 26991332, 52219388, 167869363, 335811042, 1088912364, 2061900838, 6827159829, 13424984452, 42195919228, 83374340587, 269006491243, 514570562660, 1675924761549, 3260563309970, 10226788893396, 20391731774615, 64492588219388, 123297229488909, 399014138303512, 783212435011160, 2425228785559883, 4789418078046239, 15182994877727803, 29235444078764327, 93330716828074728, 182169304991649599, 563400466799404781, 1123123932176762798, 3515289384328363733, 6748185987886118499, 21482645364583893141, 42180209153883948485, 129644921982559989678, 256751992776208115484, 803119580525790882344, 1553823126150392917494, 4893440472071899127094, 9583422277969715823101]
[0, 0.23606797749978967, 0.3819660112501051, 0.6180339887498947, 0.7639320225002101, 1.0050249987406412, 1.145898033750315, 1.380046632524605, 1.517814047519137, 1.7317965443695091, 1.8623395483158274, 2.0672696129661814, 2.1823602686738206, 2.368954529398356, 2.478676322668224, 2.6407335164997323, 2.737483482401566, 2.8867117042256147, 2.9709798281545865, 3.1023906514668242, 3.177470663034282, 3.288559025366443, 3.355010316558181, 3.454252745606137, 3.5090956346959743, 3.593944061551897, 3.643206259974442, 3.713998746074065, 3.7554136190743352, 3.817254945657911, 3.851634784541005, 3.9038924639966646, 3.933559206251074, 3.976293657375772, 4.001513367167984, 4.038159755186909, 4.058413325264224, 4.089100335584411, 4.1066844490483785, 4.131525408620234, 4.145897162377026, 4.167027229987079, 4.178757381006412, 4.1962076839539915, 4.206071745296852, 4.220026326538674, 4.228223625974912, 4.240024385571461, 4.246499836373734, 4.256156393199098, 4.261649047410081, 4.269348986196757, 4.2738027318281295, 4.280242287331251, 4.283811777970492, 4.289037101296391, 4.291988111697301]
[0, 0, 0, 1, 0, 1, 3, 2, 4, 9, 12, 20, 32, 50, 84, 132, 208, 331, 526, 841, 1333, 2145, 3401, 5314, 8396, 13279, 20952, 33029, 51927, 81527, 128102, 201700, 317461, 498911, 782868, 1226255, 1919070, 3000905, 4687213, 7315975, 11419861, 17833383, 27857264, 43511423, 67908811, 105857661, 164837336, 256493732, 398937594, 620308837, 964299016, 1498722966, 2328886172, 3618215600, 5619924806, 8725957048, 13542732051, 21008277551, 32573278946, 50480678072, 78197851828, 121086932116, 187445618110, 290118087627, 448979969989, 694763898132, 1074945125010, 1662803652299, 2571392093291, 3975074031374, 6142744228748, 9489097657132, 14653792397521, 22623678356496, 34921396940988, 53896060190628, 83170721269779, 128330590081463, 197980950188515, 305373013675616, 470903017814500, 725955935033818, 1118822820960919, 1723818676661774, 2655337007585162, 4089504559715309, 6297510121544894, 9696835494675713, 14929924846668923, 22984927653735759, 35381018243394036, 54452928150272573, 83788019525979642, 128897984242944822, 198251405524002501, 304861061425644202]
```


