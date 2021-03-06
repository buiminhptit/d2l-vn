<!-- ===================== Bắt đầu dịch Phần 1 ===================== -->
<!-- ========================================= REVISE PHẦN 1 - BẮT ĐẦU =================================== -->

<!--
# Numerical Stability and Initialization
-->

# *dịch tiêu đề phía trên*
:label:`sec_numerical_stability`

<!--
So far, for every model that we have implemented, we needed to initialize our parameters according to some specified distribution.
And until now, we glossed over the details, taking the initialization hyperparameters for granted.
You might even have gotten the impression that these choices are not especially important.
However, the choice of initialization scheme plays a significant role in neural network learning, and can be crucial for maintaining numerical stability.
Moreover, these choices can be tied up in interesting ways with the choice of the nonlinear activation function.
Which function we choose and how we initialize parameters can determine how quickly our optimization algorithm converges.
Failure to be mindful of these issues can lead to either exploding or vanishing gradients.
In this section, we delve into these topics with greater detail and discuss some useful heuristics that you may use frequently throughout your career in deep learning.
-->

*dịch đoạn phía trên*


<!--
## Vanishing and Exploding Gradients
-->

## *dịch tiêu đề phía trên*

<!--
Consider a deep network with $d$ layers, input $\mathbf{x}$ and output $\mathbf{o}$.
Each layer satisfies:
-->

*dịch đoạn phía trên*

$$\mathbf{h}^{t+1} = f_t (\mathbf{h}^t) \text{ and thus } \mathbf{o} = f_d \circ \ldots, \circ f_1(\mathbf{x}).$$

<!--
If all activations and inputs are vectors, we can write the gradient of $\mathbf{o}$ with respect to any set of parameters $\mathbf{W}_t$
associated with the function $f_t$ at layer $t$ simply as
-->

*dịch đoạn phía trên*

$$\partial_{\mathbf{W}_t} \mathbf{o} = \underbrace{\partial_{\mathbf{h}^{d-1}} \mathbf{h}^d}_{:= \mathbf{M}_d} \cdot \ldots, \cdot \underbrace{\partial_{\mathbf{h}^{t}} \mathbf{h}^{t+1}}_{:= \mathbf{M}_t} \underbrace{\partial_{\mathbf{W}_t} \mathbf{h}^t}_{:= \mathbf{v}_t}.$$

<!--
In other words, it is the product of $d-t$ matrices $\mathbf{M}_d \cdot \ldots, \cdot \mathbf{M}_t$ and the gradient vector $\mathbf{v}_t$.
What happens is similar to the situation when we experienced numerical underflow
when multiplying too many probabilities.
At the time, we were able to mitigate the problem by switching from into log-space, 
i.e., by shifting the problem from the mantissa to the exponent of the numerical representation. 
Unfortunately the problem outlined in the equation above is much more serious: initially the matrices $M_t$ may well have a wide variety of eigenvalues.
They might be small, they might be large, and in particular, their product might well be *very large* or *very small*.
This is not (only) a problem of numerical representation but it means that the optimization algorithm is bound to fail.
It receives gradients that are either excessively large or excessively small.
As a result the steps taken are either (i) excessively large (the *exploding* gradient problem), in which case the parameters blow up in magnitude rendering the model useless,
or (ii) excessively small, (the *vanishing gradient problem*), in which case the parameters hardly move at all, and thus the learning process makes no progress.
-->

*dịch đoạn phía trên*

<!-- ===================== Kết thúc dịch Phần 1 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 2 ===================== -->

<!--
### Vanishing Gradients
-->

### *dịch tiêu đề phía trên*

<!--
One major culprit in the vanishing gradient problem is the choices of the activation functions $\sigma$ that are interleaved with the linear operations in each layer.
Historically, the sigmoid function $(1 + \exp(-x))$ (introduced in :numref:`sec_mlp`) was a popular choice owing to its similarity to a thresholding function.
Since early artificial neural networks were inspired by biological neural networks, the idea of neurons that either fire or do not fire (biological neurons do not partially fire) seemed appealing.
Let's take a closer look at the function to see why picking it might be problematic vis-a-vis vanishing gradients.
-->

*dịch đoạn phía trên*

```{.python .input}
%matplotlib inline
import d2l
from mxnet import autograd, np, npx
npx.set_np()

x = np.arange(-8.0, 8.0, 0.1)
x.attach_grad()
with autograd.record():
    y = npx.sigmoid(x)
y.backward()

d2l.plot(x, [y, x.grad], legend=['sigmoid', 'gradient'], figsize=(4.5, 2.5))
```

<!--
As we can see, the gradient of the sigmoid vanishes both when its inputs are large and when they are small.
Moreover, when we execute backward propagation, due to the chain rule, this means that unless we are in the Goldilocks zone, 
where the inputs to most of the sigmoids are in the range of, say $[-4, 4]$, the gradients of the overall product may vanish.
When we have many layers, unless we are especially careful, we are likely to find that our gradient is cut off at *some* layer.
Before ReLUs ($\max(0, x)$) were proposed as an alternative to squashing functions, this problem used to plague deep network training.
As a consequence, ReLUs have become the default choice when designing activation functions in deep networks.
-->

*dịch đoạn phía trên*

<!-- ========================================= REVISE PHẦN 1 - KẾT THÚC ===================================-->

<!-- ========================================= REVISE PHẦN 2 - BẮT ĐẦU ===================================-->

<!--
### Exploding Gradients
-->

### *dịch tiêu đề phía trên*

<!--
The opposite problem, when gradients explode, can be similarly vexing.
To illustrate this a bit better, we draw $100$ Gaussian random matrices and multiply them with some initial matrix.
For the scale that we picked (the choice of the variance $\sigma^2=1$), the matrix product explodes.
If this were to happen to us with a deep network, we would have no realistic chance of getting a gradient descent optimizer to converge.
-->

*dịch đoạn phía trên*

```{.python .input  n=5}
M = np.random.normal(size=(4, 4))
print('A single matrix', M)
for i in range(100):
    M = np.dot(M, np.random.normal(size=(4, 4)))

print('After multiplying 100 matrices', M)
```

<!-- ===================== Kết thúc dịch Phần 2 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 3 ===================== -->

<!--
### Symmetry
-->

### *dịch tiêu đề phía trên*

<!--
Another problem in deep network design is the symmetry inherent in their parametrization.
Assume that we have a deep network with one hidden layer with two units, say $h_1$ and $h_2$.
In this case, we could permute the weights $\mathbf{W}_1$ of the first layer and likewise permute the weights of the output layer to obtain the same function.
There is nothing special differentiating the first hidden unit vs the second hidden unit.
In other words, we have permutation symmetry among the hidden units of each layer.
-->

*dịch đoạn phía trên*

<!--
This is more than just a theoretical nuisance.
Imagine what would happen if we initialized all of the parameters of some layer as $\mathbf{W}_l = c$ for some constant $c$.
In this case, the gradients for all dimensions are identical: thus not only would each unit take the same value, but it would receive the same update.
Stochastic gradient descent would never break the symmetry on its own and we might never be able to realize the networks expressive power.
The hidden layer would behave as if it had only a single unit.
As an aside, note that while SGD would not break this symmetry, dropout regularization would!
-->

*dịch đoạn phía trên*



<!--
## Parameter Initialization
-->

## *dịch tiêu đề phía trên*

<!--
One way of addressing, or at least mitigating the issues raised above is through careful initialization of the weight vectors.
This way we can ensure that (at least initially) the gradients do not vanish and that they maintain a reasonable scale where the network weights do not diverge.
Additional care during optimization and suitable regularization ensures that things never get too bad.
-->

*dịch đoạn phía trên*


<!--
### Default Initialization
-->

### *dịch tiêu đề phía trên*

<!--
In the previous sections, e.g., in :numref:`sec_linear_gluon`, we used `net.initialize(init.Normal(sigma=0.01))` to initialize the values of our weights.
If the initialization method is not specified, such as `net.initialize()`, 
MXNet will use the default random initialization method: each element of the weight parameter is randomly sampled with a uniform distribution $U[-0.07, 0.07]$ and the bias parameters are all set to $0$.
Both choices tend to work well in practice for moderate problem sizes.
-->

*dịch đoạn phía trên*

<!-- ===================== Kết thúc dịch Phần 3 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 4 ===================== -->

<!-- ========================================= REVISE PHẦN 2 - KẾT THÚC ===================================-->

<!-- ========================================= REVISE PHẦN 3 - BẮT ĐẦU ===================================-->

<!--
### Xavier Initialization
-->

### *dịch tiêu đề phía trên*

<!--
Let's look at the scale distribution of the activations of the hidden units $h_{i}$ for some layer. They are given by
-->

*dịch đoạn phía trên*

$$h_{i} = \sum_{j=1}^{n_\mathrm{in}} W_{ij} x_j.$$

<!--
The weights $W_{ij}$ are all drawn independently from the same distribution. 
Furthermore, let's assume that this distribution has zero mean and variance $\sigma^2$ (this does not mean that the distribution has to be Gaussian, just that mean and variance need to exist).
We do not really have much control over the inputs into the layer $x_j$ but let's proceed with the somewhat unrealistic assumption 
that they also have zero mean and variance $\gamma^2$ and that they are independent of $\mathbf{W}$.
In this case, we can compute mean and variance of $h_i$ as follows:
-->

*dịch đoạn phía trên*

$$
\begin{aligned}
    E[h_i] & = \sum_{j=1}^{n_\mathrm{in}} E[W_{ij} x_j] = 0, \\
    E[h_i^2] & = \sum_{j=1}^{n_\mathrm{in}} E[W^2_{ij} x^2_j] \\
        & = \sum_{j=1}^{n_\mathrm{in}} E[W^2_{ij}] E[x^2_j] \\
        & = n_\mathrm{in} \sigma^2 \gamma^2.
\end{aligned}
$$

<!--
One way to keep the variance fixed is to set $n_\mathrm{in} \sigma^2 = 1$.
Now consider backpropagation.
There we face a similar problem, albeit with gradients being propagated from the top layers.
That is, instead of $\mathbf{W} \mathbf{w}$, we need to deal with $\mathbf{W}^\top \mathbf{g}$, where $\mathbf{g}$ is the incoming gradient from the layer above.
Using the same reasoning as for forward propagation, we see that the gradients' variance can blow up unless $n_\mathrm{out} \sigma^2 = 1$.
This leaves us in a dilemma: we cannot possibly satisfy both conditions simultaneously.
Instead, we simply try to satisfy:
-->

*dịch đoạn phía trên*

$$
\begin{aligned}
\frac{1}{2} (n_\mathrm{in} + n_\mathrm{out}) \sigma^2 = 1 \text{ or equivalently }
\sigma = \sqrt{\frac{2}{n_\mathrm{in} + n_\mathrm{out}}}.
\end{aligned}
$$

<!--
This is the reasoning underlying the eponymous Xavier initialization :cite:`Glorot.Bengio.2010`.
It works well enough in practice.
For Gaussian random variables, the Xavier initialization picks a normal distribution with zero mean and variance $\sigma^2 = 2/(n_\mathrm{in} + n_\mathrm{out})$.
For uniformly distributed random variables $U[-a, a]$, note that their variance is given by $a^2/3$.
Plugging $a^2/3$ into the condition on $\sigma^2$ yields that we should initialize uniformly with $U\left[-\sqrt{6/(n_\mathrm{in} + n_\mathrm{out})}, \sqrt{6/(n_\mathrm{in} + n_\mathrm{out})}\right]$.
-->

*dịch đoạn phía trên*


<!-- ===================== Kết thúc dịch Phần 4 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 5 ===================== -->

<!--
### Beyond
-->

### *Ngoài ra*

<!--
The reasoning above barely scratches the surface of modern approaches to parameter initialization.
In fact, MXNet has an entire [`mxnet.initializer`](https://mxnet.apache.org/api/python/docs/api/initializer/index.html) module implementing over a dozen different heuristics.
Moreover, initialization continues to be a hot area of inquiry within research into the fundamental theory of neural network optimization.
Some of these heuristics are especially suited for when parameters are tied 
(i.e., when parameters of in different parts the network are shared), for super-resolution, sequence models, and related problems.
We recommend that the interested reader take a closer look at what is offered as part of this module, and investigate the recent research on parameter initialization.
Perhaps you may come across a recent clever idea and contribute its implementation to MXNet, or you may even invent your own scheme!
-->

*dịch đoạn phía trên*
Lí do được đưa ra ở phần trên hiếm khi ảnh hưởng nhiều đến những kĩ thuật hiện đại trong việc khởi tạo tham số.
Thực tế thì MXNet có hẳn cả một module [`mxnet.initializer`](https://mxnet.apache.org/api/python/docs/api/initializer/index.html) thực thi hàng chục các phương pháp thực nghiệm.
Hơn thế nữa thì khởi tạo tiếp tục là một lĩnh vực nghiên cứu nóng hổi về tìm tòi trong nghiên cứu về học thuyết cơ bản trong việc tối ưu mạng neuron.
Một số phương pháp thực nghiệm này thực sự phù hợp khi tham số bị ràng buộc.
(ví dụ : Khi mà tham số của các phần khác nhau trong mạng được chia sẻ với nhau),trong việc tăng chất lượng hình ảnh,mô hình chuỗi và những vấn đề liên quan. 
Chúng tôi gợi ý rằng những người đọc cảm thấy hứng thú có thể quan sát kỹ hơn những gì được đưa ra như một phần của module và tìm hiểu những nghiên cứu gần đây về vấn đề khởi tạo tham số.
Có thể bạn sẽ bắt gặp những ý tưởng thông minh gần đây và đóng góp thực thi của chúng vào MXNET, hoặc thậm chí là tự phát minh ra cách thức cho riêng mình.

<!--
## Summary
-->

## Tóm tắt

<!--
* Vanishing and exploding gradients are common issues in very deep networks, unless great care is taking to ensure that gradients and parameters remain well controlled.
* Initialization heuristics are needed to ensure that at least the initial gradients are neither too large nor too small.
* The ReLU addresses one of the vanishing gradient problems, namely that gradients vanish for very large inputs. This can accelerate convergence significantly.
* Random initialization is key to ensure that symmetry is broken before optimization.
-->

*dịch đoạn phía trên*
* Tiêu biến hay bùng nổ gradients đều là những vấn đề chung trong những mạng rất sâu (nhiều layers), trừ khi có sự cẩn thận đảm bảo rằng gradients và các tham số vẫn được kiểm soát tốt.
* Các thực nghiệm trong khởi tạo là cần thiết để đảm bảo ít nhất rằng gradients ban đầu không bị quá lớn hay quá nhỏ.
* ReLU giải quyết một trong những vấn đề về tiêu biến gradients, cụ thể là việc tiêu biến gradients cho các đầu vào rất lớn. Điều này có thể tăng tốc độ hội tụ đáng kể.
* Khởi tạo ngẫu nhiên là chìa khóa để đảm bảo tính đối xứng bị phá vỡ trước khi tối ưu hóa.

<!--
## Exercises
-->

## Bài tập

<!--
1. Can you design other cases of symmetry breaking besides the permutation symmetry?
2. Can we initialize all weight parameters in linear regression or in softmax regression to the same value?
3. Look up analytic bounds on the eigenvalues of the product of two matrices. What does this tell you about ensuring that gradients are well conditioned?
4. If we know that some terms diverge, can we fix this after the fact? Look at the paper on LARS for inspiration :cite:`You.Gitman.Ginsburg.2017`.
-->

*dịch đoạn phía trên*
1. Bạn có thể thiết kế các trường hợp phá vỡ đối xứng khác bên cạnh đối xứng hoán vị?
2. Chúng ta có thể khởi tạo tất cả trọng số ở trong mạng hồi quy tuyến tính hoặc trong hồi quy softmax cùng một giá trị hay không?
3. Dựa trên những phân tích ràng buộc về trị riêng của phép nhân 2 ma trận (quá lớn sẽ khiến exploding gradient và quá nhỏ sẽ khiến vanishing gradient). Những phân tích này cho ta biết điều gì để đảm bảo gradient có độ lớn vừa phải ?
4. Nếu biết rằng trong vài trường hợp, mô hình không thể hội tụ, liệu ta có thể sửa chữa vấn đề này (mô hình bị phân kỳ do vanishing/exploding gradient). Có thể xem gợi ý ở paper LARS :cite:You.Gitman.Ginsburg.2017.
<!-- ===================== Kết thúc dịch Phần 5 ===================== -->

<!-- ========================================= REVISE PHẦN 3 - KẾT THÚC ===================================-->

<!--
## [Discussions](https://discuss.mxnet.io/t/2345)
-->

## Thảo luận
* [Tiếng Anh](https://discuss.mxnet.io/t/2345)
* [Tiếng Việt](https://forum.machinelearningcoban.com/c/d2l)

## Những người thực hiện
Bản dịch trong trang này được thực hiện bởi:
<!--
Tác giả của mỗi Pull Request điền tên mình và tên những người review mà bạn thấy
hữu ích vào từng phần tương ứng. Mỗi dòng một tên, bắt đầu bằng dấu `*`.

Lưu ý:
* Nếu reviewer không cung cấp tên, bạn có thể dùng tên tài khoản GitHub của họ
với dấu `@` ở đầu. Ví dụ: @aivivn.

* Tên đầy đủ của các reviewer có thể được tìm thấy tại https://github.com/aivivn/d2l-vn/blob/master/docs/contributors_info.md.
-->

* Đoàn Võ Duy Thanh
<!-- Phần 1 -->
*

<!-- Phần 2 -->
*

<!-- Phần 3 -->
*

<!-- Phần 4 -->
*

<!-- Phần 5 -->
*
