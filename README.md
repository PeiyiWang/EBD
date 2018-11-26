# EBD : Efficient Bessel Decomposition toolbox (matlab)
#### Fast convolution with radial kernels in 2D


This toolbox implements the algorithm described in "Discrete convolution in $\mathbb{R}^2$  with radial kernels using non-uniform fast Fourier transform with non-equispaced frequencies", written by Martin Averseng, and submitted to the journal Numerical Algorithms in 2018. 

To test it, you can directly run Demo.m, or DemoGrad. Here follows a more detailed description of the algorithm and a tutorial. The method is designed to compute fast approximations of vectors $$q$$ which entries are given by 
<a href="https://www.codecogs.com/eqnedit.php?latex=$$q_k&space;=&space;\sum_{l=1}^{N_y}&space;G(X_k&space;-&space;Y_l)&space;f_l,&space;\quad&space;k&space;=&space;1,&space;\cdots,&space;N_x$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$q_k&space;=&space;\sum_{l=1}^{N_y}&space;G(X_k&space;-&space;Y_l)&space;f_l,&space;\quad&space;k&space;=&space;1,&space;\cdots,&space;N_x$$" title="$$q_k = \sum_{l=1}^{N_y} G(X_k - Y_l) f_l, \quad k = 1, \cdots, N_x$$" /></a>

or
<a href="https://www.codecogs.com/eqnedit.php?latex=$$q_k&space;=&space;\sum_{l=1}^{N_y}&space;\nabla&space;G(X_k&space;-&space;Y_l)&space;f_l,&space;\quad&space;k&space;=&space;1,&space;\cdots,&space;N_x$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$q_k&space;=&space;\sum_{l=1}^{N_y}&space;\nabla&space;G(X_k&space;-&space;Y_l)&space;f_l,&space;\quad&space;k&space;=&space;1,&space;\cdots,&space;N_x$$" title="$$q_k = \sum_{l=1}^{N_y} \nabla G(X_k - Y_l) f_l, \quad k = 1, \cdots, N_x$$" /></a>

where <a href="https://www.codecogs.com/eqnedit.php?latex=$G$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$G$" title="$G$" /></a> is a radial function i.e. <a href="https://www.codecogs.com/eqnedit.php?latex=$G(x)&space;=&space;g(\lvert&space;x\rvert)$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$G(x)&space;=&space;g(\lvert&space;x\rvert)$" title="$G(x) = g(\lvert x\rvert)$" /></a> for some function <a href="https://www.codecogs.com/eqnedit.php?latex=$g$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$g$" title="$g$" /></a>,<a href="https://www.codecogs.com/eqnedit.php?latex=$X$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$X$" title="$X$" /></a> and <a href="https://www.codecogs.com/eqnedit.php?latex=$Y$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$Y$" title="$Y$" /></a>  are two clouds of $N_x$ and $N_y$ points in $$\mathbb{R}^2$$ and $$f$$ is a complex vector. 

### 1°) Description of the algorithm

The method first decomposes $$G$$ in finite Bessel series 
$$$G(r) = \sum_{p = 1}^P \alpha_p J_0(\rho_p r) $$$
where $$J_0$$ is the Bessel function of first kind, $$\rho$$ is the sequence of its positive zeros, and $$\alpha$$ are called the EBD (Efficient Bessel Decomposition) coefficients of $$G$$. The coefficients are chosen as the minimizers of the Sobolev $$H^1_0$$ norm error in this approximation on a ring $$r_{min} < r < r_{max}$$ where $r_{min}$ is a cutoff parameter and $$r_{max}$$ is the greatest distance occurring between two points $$X_k$$ and $$Y_l$$. The method takes the paramter $$a:= \frac{r_{min}}{r_{max}}$$ as an input. 

Then, each $$J_0(\rho_p r)$$ is approximated by $$J_0(\rho_p |x|) = \frac{1}{M_p}\sum_{m = 1}^{M_p} e^{i \rho_p \xi_{m}^p \cdot x}$$ where $$\xi_m^p = e^{i\frac{2m\pi}{M_p}}$$. This is the trapezoidal rule applied to the formula 
$$J_0(|x|) = \int_{\partial B} e^{i x \cdot \xi} d\xi$$
where the integration takes place on the boundary of the unit disk $B$ in $\mathbb{R}^2$. 

Combining these two steps, we obtain an approximation for $$G$$ of the form 
$$G(x) \approx \sum_{\nu = 1}^{N_\xi} \hat{\omega}_\nu e^{i \xi_\nu \cdot x}$$
valid for $$|x| > r_{min}$$. If this is replaced in the expression of $$q_k$$, we see that  $$q$$ can be approximated by non-uniform Fourier transform for any vector $$f$$. The interactions $$|X_k - Y_l| < r_{min}$$ where the approximation is not valid are corrected by a sparse matrix product. 


The code is in Matlab language. The implementation of the NUFFT is borrowed from Leslie Greengard, June-Yub Lee and Zydrunas Gimbutas (see license file in the libGgNufft2D folder). The ideas come from a similar method in 3D called Sparse Cardinal Sine Decomposition, developped by François Alouges and Matthieu Aussal, also published in Numerical Algorithms. 

### 2°) Tutorial

#### A. Simple EBD
- Create the arrays $$X$$ and $$Y$$ of sizes $$N_x \times  2$$ and $$N_y \times 2$$ (points in R^2). 
- Create a kernel by calling 
```
G = Kernel(fun,der);
```

where fun is an anonymous function of your choice and der is an anonymous function 
repesenting the derivative of fun. For example, 
```
G = Kernel(@(x)(1./x),@(x)(-1./x.^2));
```

Be aware that the anonymous functions provided in argument must accept arrays as input. For some classical kernels, the methods have been optimized. You can create one of those special kernels using the Kernel library:
```
G = LogKernel;  % represents G(x) = log(x);
G = ThinPlate(a,b) % represents G(x) = a*x^2*log(b*x);
```

(and others, see folder Kernels).

- Define the tolerance in the error of approximation. The method guarantees that 
the Bessel decomposition of $$G$$ in the ring is accurate at the tolerance level. 
This implies that the maximal error on an entry of $$q_k$$ is `tol*norm(q,1)`.  

- Define the parameter $$a$$ (the ratio between $$r_{min}$$ and $$r_{max}$$.) $$a$$ is roughly the proportion of interactions that will be computed exactly. There is an optimal value of $$a$$ for which the evaluation of the convolution is the fastest, but it is not possible to know it in advance. However, when $$X$$ and $$Y$$ are uniformly distributed on a disk, the optimal $$a$$ is of the order $$\frac{1}{(N_x N_y)^{1/4}}$$, and if they are uniformly distributed on a curve, the optimal $$a$$ is of the order $$\frac{1}{(N_x N_y)^{1/3}}$$. If $$a$$ is large, a lot of interactions are computed exactly while the Bessel decomposition will have only a few terms. If $$a$$ is small, the opposite will happen. 

- You can now call
```
[onlineEBD, rq, loc] = offlineEBD(G,X,Y,a,tol);
```

The variable `onlineEBD` contains a handle function. If f is a vector with length equal to size(Y,1)
```
q = onlineEBD(f);
```
returns the approximation of the convolution. The variable `rq` contains a RadialQuadrature object. You can visualize a representation of the radial approximation with
```
 rq.show();
```
and also check all its properties (including coefficients, frenquencies used, accuracy, ...). Finally, `loc` is the local correction matrix (used inside onlineEBD). 

### B. Derivative EBD 

If you rather want to compute the vectors 
$$[q_1(i),q_2(i)] = \sum_{j=1}^{N_y} \nabla{G}(Y_j - X_i)f_j$$ 
where for a point $$x$$ in $$\mathbb{R}^2$$,  $$\nabla G(x) = G'(|x|)\frac{ x}{|x|}$$, you may call `offline_dEBD`instead of `offlineEBD`. The syntax is
```
[MVx,MVy, rq, loc] = offlineEBD(G,X,Y,a,tol);
```
where, for example, `MVx`contains the handle function such that 
```
q1 = MVx(f)
```
is the component $$q_1$$ of the convolution with $$\nabla G$$. 

