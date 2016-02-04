---
layout: post
title: "Understanding Gabor Filter"
description: ""
category:
tags: [image processing, fourier transform, convolution, signal processing, feature extraction]
---
{% include JB/setup %}

## Objective:
Formulate  the formulae for gabor filter bank

## Formulate the formulae for gabor filter

### Gaussian function and its fourier transform

The equation for a one dimensional gaussian function is

$$
\begin{equation} \label{eqa}
h(t) = \frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{t^2}{2\sigma^2}}
\end{equation}
$$

For a  general function $$f(t)$$, the fourier transform $$F(w)$$ is given by,


$$
\begin{equation} \label{eqv}
F(\omega) = \int_{-\infty}^{\infty} f(t) e^{-i2\pi\omega t} dt
\end{equation}
$$


From \eqref{eqf}, fourier transform $$H(w)$$ of $$h(t)$$ in \eqref{eqa} is,

$$
\begin{eqnarray} \label{equ}
H(\omega) & = &  \frac{1}{\sqrt{2\pi}\sigma} \sqrt{2\pi}\sigma e^{-2\pi^2\omega^2\sigma^2} \\
 & = &   e^{-2\pi^2\omega^2\sigma^2} \\
\end{eqnarray}
$$


### Introducing one dimensional gabor filter


Consider the functions $$g_{e}(t) = h(t) cos(2 \pi \omega_0 t)$$ and $$g_{o}(t) = h(t) sin(2 \pi \omega_0 t)$$, which are gaussian functions modulated
by a cosine and a sine function respectively.


![fig-0, 1d gabor functions ]({{ site.url }}/assets/content/gabor_1D_plots.png)

The plots of the functions $$h(t), g_e(t)$$ and $$g_o(t)$$ are shown above.

The one dimensional complex gabor filter is given by

$$
\begin{eqnarray} \label{eqq}
g(t) & = & g_e(t) + i g_o(t) \\
     & = & h(t) ( cos(2 \pi \omega_0 t) + i sin(2 \pi \omega_0 t))\\
     & = & h(t) e^{i2\pi\omega_0}
\end{eqnarray}
$$



Please note that the fourier transform of the product of two functions, $$p(t)$$, $$q(t)$$, is the convolution of their respective
fourier transforms, ie $$P(\omega) * Q(\omega)$$.

Let us do a fourier analysis of $$g_{e}(t)$$ and $$g_{o}(t)$$.



$$
\begin{eqnarray} \label{eqn}
G_{e}(\omega) & = & H(\omega) * C(\omega)\\
G_{o}(\omega) & = & H(\omega) * S(\omega)
\end{eqnarray}
$$

Since $$ p(x) * q(x)  =  \int_{-\infty}^{\infty} p(t) q(x - t) dt $$ and since convolution is commutative, and using \eqref{eql}, \eqref{eqf},

$$
\begin{eqnarray} \label{eqo}
G_{e}(\omega) & = & \int_{-\infty}^{\infty} C(u)  H(\omega - u) du \\
 & = &  \frac{1}{2} \int_{-\infty}^{\infty}   H(\omega - u)  (\delta(u + \omega_0) + \delta(u - \omega_0)) du\\
 & = &  \frac{1}{2}  ( H(\omega + \omega_0)   + H(\omega - \omega_0) ) \\
   & = &  \frac{1}{2}  ( e^{-2\pi^2(\omega + \omega_0)^2\sigma^2}  +  e^{-2\pi^2(\omega - \omega_0)^2\sigma^2}  )
\end{eqnarray}
$$


So by \eqref{eqq}, the real part of the  fourier transform of $$g_e(t)$$ is the sum of two gaussian functions separated by $$\pm \omega_0$$, and the imaginary part
is the difference of two gaussian finctions separeted by $$\pm \omega_{0} $$.

Similarly,
$$
\begin{eqnarray} \label{eqp}
G_{o}(\omega) & = & \int_{-\infty}^{\infty} S(u)  H(\omega - u) du \\
 & = &  \frac{i}{2} \int_{-\infty}^{\infty}   H(\omega - u)  (\delta(u + \omega_0) - \delta(u - \omega_0)) du\\
 & = &  \frac{i}{2}  ( H(\omega + \omega_0)   - H(\omega - \omega_0) ) \\
   & = &  \frac{i}{2}  ( e^{-2\pi^2(\omega + \omega_0)^2\sigma^2}  -  e^{-2\pi^2(\omega - \omega_0)^2\sigma^2}  )
\end{eqnarray}
$$


Similarly,
$$
\begin{eqnarray} \label{eqp2}
G(\omega) & = & G_e{\omega} + i G_o{\omega} \\
 & = & \frac{1}{2}  ( H(\omega + \omega_0)   + H(\omega - \omega_0) ) - \frac{1}{2}  ( H(\omega + \omega_0)   - H(\omega - \omega_0) )\\
 & = & H(\omega - \omega_0)
\end{eqnarray}
$$

![fig-0, 1d gabor functions fourier transforms,]({{ site.url }}/assets/content/gabor_1D_plots_fourier.png)

### Introducing two dimensional gabor filter kernel

![fig-1, 2D gaussian]({{ site.url }}/assets/content/gabor_2D_gssn.png)

In fig-1,  we have plotted the function $$h(x,y) = \frac{1}{2\pi\sigma_{x}\sigma_{y}} e ^{-\frac{1}{2}(\frac{x^2}{\sigma_{x}^{2}} + \frac{y^2}{\sigma_{y}^{2}})} $$ and



![fig-1, 2D cosine wave]({{ site.url }}/assets/content/gabor_2D_cos.png)


A general 2D cosine function is given by  $$ c(x,y) = cos(2\pi (u_1 x + v_1 y)) $$, where $$u_1, v_1 $$ are fixed spatial frequencies. This is shown in fig-2.



Just as in the case of the 1D gabor filter kernel, we define the 2D gabor filter kernel by the following equations. The complex
2D gabor filter kernel is given by $$g(x, y)$$.


$$
\begin{eqnarray} \label{eqs2}
g_e(x, y) & = & \frac{1}{2\pi\sigma_{x}\sigma_{y}} e ^{-\frac{1}{2}(\frac{x^2}{\sigma_{x}^{2}} + \frac{y^2}{\sigma_{y}^{2}})} cos(2\pi(u_1 x + v_1 y))\\
g_o(x, y) & = & \frac{1}{2\pi\sigma_{x}\sigma_{y}} e ^{-\frac{1}{2}(\frac{x^2}{\sigma_{x}^{2}} + \frac{y^2}{\sigma_{y}^{2}})} sin(2\pi(u_1 x + v_1 y))\\
g(x, y) & = & g_e(x, y) + i g_o(x, y) \\
 & = & \frac{1}{2\pi\sigma_{x}\sigma_{y}} e ^{-\frac{1}{2}(\frac{x^2}{\sigma_{x}^{2}} + \frac{y^2}{\sigma_{y}^{2}})} e^{i2\pi(u_1 x + v_1 y)}
\end{eqnarray}
$$


![fig-1, 2D  gaussian times cosine wave]({{ site.url }}/assets/content/gabor_2D_gssn_times_cos.png)

In fig-3,  we have plotted the function $$ g_e(x,y) = h(x, y). c(x, y) $$.

 Note that in figure 2, and fig 3, the  3d perspective views
are slightly rotated to accentuate their features for viewing decipherability.



Here is the octave code used for generating fig-2.


<pre>
% octave code for showing  g_e(x, y) = gaussian times cosine function
%author kgeorge

clear all;
clc;
close all;
pkg load image



sigma_x = 0.5;
sigma_y = 0.5;

omega = 1.0;

%kernel size = 7 x 7
N=7;
NBY2 = floor(N/2);
tx=ty=linspace(-NBY2, NBY2, 50);
[xx, yy] = meshgrid(tx, ty);


%plot of a simple 2D gaussian
I = exp(-0.5 * ( (xx ./ sigma_x) .^2 +  (yy ./ sigma_y) .^2  ) );
I =  I .* (1.0/(2.0 * pi * sigma_x * sigma_y));


%plot of a 2D cosine wave
J = cos((xx + yy) .* (2 * pi * omega) );

colormap(copper)
s1 = subplot(2,1,2);
imshow(I .* J)
xlabel('x');
ylabel('y');

title(s1, sprintf("top view"));


colormap(copper)
s2 = subplot(2,1,1);
mesh(xx, yy, I .* J);
grid off;
view(33, 24);
xlabel('x');
ylabel('y');

title(s2, sprintf("perspective view, rotated for convenience"));



ha = axes('Position',[0 0 1 1],'Xlim',[0 1],'Ylim',[0 1],'Box','off','Visible','off','Units','normalized', 'clipping' , 'off');
text(0.5, 0.05, sprintf("2d gaussian times cos function,  \\sigma_x=%.1f,\\sigma_y=%.1f, \\omega=%.1f", sigma_x, sigma_y, omega),'HorizontalAlignment','center','VerticalAlignment', 'top');
%save using a uility function
%saveInGithubBlog("gabor_2D_gssn_times_cos");
</pre>

There are a number of varying parameters in the above formulae for a gabor filter kernel ie $$ \sigma_x, \sigma_y, u_1$$ and $$v_1 $$. We can add a few more
parameters to make it a bit more general and useful to us so that we can use a proper specialization of this kernel for
a proper image segmentation problem. So let us play around with these parameters.

Since we are just studying the effects of these parameters, let us continue to focus on the real part $$g_e(x,y)$$ of this filter.

![fig-3, real part of 2D gabor function with varying sigma-s]({{ site.url }}/assets/content/gabor_2D_real_part_with_varying_sigmas.png)
In fig-3, we have plotted the real part of the gabor 2D filter with varying values of $$\sigma_x$$ and $$\sigma_y$$. Note that,
as the $$\sigma$$-values vary, the eccentricities of the gaussian ellipse, changes, but nevertheless remains axially aligned.

Let us now try varying  $$u_1 $$ and $$v_1$$. To illustrate this, just for the purpose of clarity, we are going to plot
only the cosine function, ie $$cos(2\pi (u_1 x +  v_1 y))$$. Its effect on the modulation with gaussian function
is self explanatory. Though we could indepentally vary $$u_1$$ and $$v_1$$, for the case of simplicity we are varying these parameters by forcing  $$u_1 == v_1$$.


![fig-4,2D cosine function with varying u-v-s]({{ site.url }}/assets/content/gabor_2D_cos_with_varying_omegas.png)

You can see from fig 4, the changes in the frequency of the cosine wave.

After playing around with these parameters, one feature that is evidently missing is the ability to change the angle at which
the cosine pattern forms w.r.t the x-axis. In fact we need to change this angle irrespective of the spatial frequency parameters $$u_1$$ and $$v_1$$.

Such a rotation, can be accomplished by the rotation of the x and y co-ordinates while evaluating the cosine function.

![fig-5,2D rotation of axes ]({{ site.url }}/assets/content/gabor_2D_rotation_of_axes.png)

The above rotation can be accomplished by the following co-ordinate transform,


$$
\begin{eqnarray} \label{eqt}
R & = &  \begin{bmatrix} cos(\alpha) & -sin(\alpha) \\ sin(\alpha) & cos(\alpha)\\ \end{bmatrix}\\
\begin{bmatrix} x'  \\ y' \\ \end{bmatrix} & = &  R \begin{bmatrix} x  \\ y \\ \end{bmatrix}
\end{eqnarray}
$$

The cosine function we have been following now becomes, $$cos(2\pi(u_1 x' + v_1 y'))$$, Plotting that for various values
of $$\alpha$$ yields the following figure.


![fig-6,2D cosine function with varying rotation around axes]({{ site.url }}/assets/content/gabor_2D_cos_with_axes_rotation.png)

In fact, given above is how the cosine function would look when the axes are rotated by the transform given by \eqref{eqt} for various values
of $$\alpha$$.



Another parameter that need be introduced is the amount by which the cosine pattern should shift. Let us modify the cosine funstion as
 $$cos(2\pi(u_1 x' + v_1 y') + \phi)$$

![fig-6,2D cosine function with varying phase]({{ site.url }}/assets/content/gabor_2D_cos_with_phase.png)


Putting all these together, the gabor kernel is defined as


$$
\begin{eqnarray} \label{eqw}
x' & = & x cos(\alpha) - y sin(\alpha) \\
y' & = & x sin(\alpha) + y cos(\alpha) \\
g_e(x, y) & = & \frac{1}{2\pi\sigma_{x}\sigma_{y}} e ^{-\frac{1}{2}(\frac{x^2}{\sigma_{x}^{2}} + \frac{y^2}{\sigma_{y}^{2}})} cos(2\pi(u_1 x' + v_1 y') + \phi)\\
g_o(x, y) & = & \frac{1}{2\pi\sigma_{x}\sigma_{y}} e ^{-\frac{1}{2}(\frac{x^2}{\sigma_{x}^{2}} + \frac{y^2}{\sigma_{y}^{2}})} sin(2\pi(u_1 x' + v_1 y') + \phi)\\
g(x, y) & = & g_e(x, y) + i g_o(x, y) \\
 & = & \frac{1}{2\pi\sigma_{x}\sigma_{y}} e ^{-\frac{1}{2}(\frac{x^2}{\sigma_{x}^{2}} + \frac{y^2}{\sigma_{y}^{2}})} e^{i(2\pi(u_1 x' + v_1 y') + \phi)}
\end{eqnarray}
$$

There is another school of thought, where by the $$x'$$ and $$y'$$ is used in the gaussian function part as well, instead of $$x$$, and
$$y$$, giving the gaussian the same orientation as the cosine waves according to which the formulae becomes,


$$
\begin{eqnarray} \label{eqx}
x' & = & x cos(\alpha) - y sin(\alpha) \\
y' & = & x sin(\alpha) + y cos(\alpha) \\
g_e(x, y) & = & \frac{1}{2\pi\sigma_{x}\sigma_{y}} e ^{-\frac{1}{2}(\frac{x'^{2}}{\sigma_{x}^{2}} + \frac{y'^{2}}{\sigma_{y}^{2}})} cos(2\pi( u_1 x' +  v_1 y') + \phi)\\
g_o(x, y) & = & \frac{1}{2\pi\sigma_{x}\sigma_{y}} e ^{-\frac{1}{2}(\frac{x'^{2}}{\sigma_{x}^{2}} + \frac{y'^{2}}{\sigma_{y}^{2}})} sin(2\pi( u_1 x' + v_1 y') + \phi)\\
g(x, y) & = & g_e(x, y) + i g_o(x, y) \\
 & = & \frac{1}{2\pi\sigma_{x}\sigma_{y}} e ^{-\frac{1}{2}(\frac{x'^2}{\sigma_{x}^{2}} + \frac{y'^2}{\sigma_{y}^{2}})} e^{i(2\pi( u_1 x' + v_1 y') + \phi)}
\end{eqnarray}
$$


## Appendix

###  Derivation of fourier transform of a 1-D gaussian function.


How do we find the fourier transform $$H(\omega)$$ of a one dimensional gaussian function $$h(t)$$?

For a  general function $$f(t)$$, the fourier transform $$F(w)$$ is given by \eqref{eqv}.


On a simpler note, let us work on a simple function $$h(t) = e^{-at^2}$$ and try to find its fourier transform $$H(\omega)$$.


$$
\begin{eqnarray} \label{eqc}

H(\omega) & = & \int_{-\infty}^{\infty} h(t) e^{-i2\pi\omega t} dt \\
         & = & \int_{-\infty}^{\infty} e^{-at^2} e^{-i2\pi\omega t} dt \\
         & = & \int_{-\infty}^{\infty} e^{-at^2} (cos(2\pi\omega t) - i. sin(2\pi\omega t)) dt \\
         & = & \int_{-\infty}^{\infty} e^{-at^2} cos(2\pi\omega t) - i. \int_{-\infty}^{\infty}  e^{-at^2} sin(2\pi\omega t) dt
\end{eqnarray}
$$

Please remember that for an odd function $$o(t)$$,  $$\int_{-\infty}^{0} o(t) =  -\int_{0}^{\infty} o(t)$$ and so $$\int_{-\infty}^{\infty} o(t) = 0 $$.

Since  $$e^{-at^2}$$ is an even function and $$sin(2\pi\omega t)$$ is an odd function, $$ \int_{-\infty}^{\infty}  e^{-at^2} sin(2\pi\omega t) = 0$$.

So the above equation simplifies to

$$
\begin{eqnarray} \label{eqd}
H(\omega) & = &  \int_{-\infty}^{\infty} e^{-at^2} cos(2\pi\omega t) \space dt\\
& = &  2 \int_{0}^{\infty} e^{-at^2} cos(2\pi\omega t) \space dt
\end{eqnarray}
$$

Using [[1], Abramovitz et. al.](#references), $$\int_{0}^{\infty} e^{-at^2} cos(2xt) \space dt = \frac{1}{2}\sqrt{\frac{\pi}{a}} e^{-\frac{x^2}{a}} $$.
So
the above equation translates to,


$$
\begin{eqnarray} \label{eqe}
H(\omega) & = &  \sqrt{\frac{\pi}{a}} e^{-\frac{\pi^2\omega^2}{a}}
\end{eqnarray}
$$

Putting $$a = \frac{1}{2\sigma^2}$$, in \eqref{eqe}, fourier transform $$H(w)$$ of $$h(t)$$ in \eqref{eqa} is,

$$
\begin{eqnarray} \label{eqf}
H(\omega) & = &  \frac{1}{\sqrt{2\pi}\sigma} \sqrt{2\pi}\sigma e^{-2\pi^2\omega^2\sigma^2} \\
 & = &   e^{-2\pi^2\omega^2\sigma^2} \\
\end{eqnarray}
$$

###  Derivation of fourier transform of a 2D gaussian function.

A 2D gaussian function is given by \eqref{eqaa}
$$
\begin{eqnarray} \label{eqaa}
h(x,y) & = & \frac{1}{2\pi\sigma_{x}\sigma_{y}} e ^{-\frac{1}{2}(\frac{x^2}{\sigma_{x}^{2}} + \frac{y^2}{\sigma_{y}^{2}})}
\end{eqnarray}
$$

Note that \eqref{eqaa} can be written as,
$$
\begin{eqnarray} \label{eqab}
h_1(x) & = & \frac{1}{\sqrt{2\pi}\sigma_{x}}  e ^{ - 0.5 \frac{x^2}{\sigma_{x}^{2}}}  \\
h_2(y) & = & \frac{1}{\sqrt{2\pi}\sigma_{y}}  e ^{ - 0.5 \frac{y^2}{\sigma_{y}^{2}}}  \\
h(x, y) & = & h_1(x) h_2(y)
\end{eqnarray}
$$


Given any 2D function $$f(x,y)$$, its fourier transform $$F(u,v)$$ is given by

$$
\begin{eqnarray} \label{eqac}
F(u, v) & = & \int_{-\infty}^{\infty} \int_{-\infty}^{\infty} f(x, y) e^{-i2\pi(ux + vy)} \space dx dy
\end{eqnarray}
$$

A 2D function $$f(x, y)$$ is separable, if it can be written as $$f(x,y) = f_{1}(x).f_{2}(y)$$. If $$F_{1}(u)$$ and $$F_{2}(v)$$ are the
fourier transforms of $$f_{1}$$ and $$f_{2}$$ respectively, then,

$$
\begin{eqnarray} \label{eqad}
F(u, v) & = & \int_{-\infty}^{\infty} \int_{-\infty}^{\infty} f_1(x) f_2(y) e^{-i2\pi(ux + vy)} \space dx dy \\
    & = & \int_{-\infty}^{\infty} \int_{-\infty}^{\infty} f_1(x)  e^{-i2\pi ux} dx \space f_2(y) e^{-i2\pi vy} dy \\
    & = & F_1(u)  F_2(v)
\end{eqnarray}
$$

From \eqref{eqab}, \eqref{eqad}, and \eqref{eqf}, we derive the fourier transform $$H(u, v)$$ of a gaussian as,

$$
\begin{eqnarray} \label{eqaf}
H(u, v) & = &  e^{-2\pi^2(u^2\sigma_{x}^2 + v^2\sigma_{y}^2 )}
\end{eqnarray}
$$

### Derivation of fourier transform of sine and cosine functions


Consider a simple cosine and a sine function, $$c(t)$$ and $$s(t)$$ respectively, what would be their fourier transforms $$C(w)$$ and $$S(w)$$?


$$
\begin{eqnarray} \label{eqh}
c(t) & = & cos(2\pi \omega_{0}t) \\
 & = & \frac{1}{2} ( e^{-i2\pi\omega_{0} t} + e^{i2\pi\omega_{0} t} )
\end{eqnarray}
$$

$$
\begin{eqnarray} \label{eqg}
s(t) & = & sin(2\pi \omega_{0}t) \\
& = & \frac{i}{2} ( e^{-i2\pi\omega_{0} t} - e^{i2\pi\omega_{0} t} )
\end{eqnarray}
$$


From \eqref{eqh},


$$
\begin{eqnarray} \label{eqi}
C(\omega) & = & \int_{-\infty}^{\infty} c(t) e^{-i2\pi\omega t} dt \\
& = & \int_{-\infty}^{\infty} \frac{1}{2} ( e^{-i2\pi\omega_{0} t} + e^{i2\pi\omega_{0} t} ) e^{-i2\pi\omega t} dt \\
& = & \int_{-\infty}^{\infty} \frac{1}{2} ( e^{-i2\pi(\omega + \omega_{0}) t} + e^{-i2\pi (\omega - \omega_{0}) t} ) dt \\
\end{eqnarray}
$$

Consider the integral $$ \int_{-\infty}^{\infty}  e^{-i2\pi \omega t} dt $$, That is the fourier transform $$I(\omega)$$ of the identity function $$i(t)=1$$.




The fourier inversion theorem states the following, if $$ F(\omega) = \int_{-\infty}^{\infty}  e^{-i2\pi \omega t} f(t) \space dt$$, then
 $$ \int_{-\infty}^{\infty}  e^{i2\pi \omega x} F(\omega) d\omega = f(x) $$. For a  proof, please
see [[2], Proof of Fourier inversion theorem](#references)



Now considering, the dirac delta function $$\delta(t)$$ with the property $$ \int_{-\infty}^{\infty} \delta(t-a) f(t) = f(a)$$, the integral below,
evaluates to $$1$$ or the identity function $$i(x)$$.


$$
\begin{eqnarray} \label{eqj}
\int_{-\infty}^{\infty}  e^{i2\pi \omega x} \delta(\omega) d\omega & = &  1 \\
 & = &  i(x)
\end{eqnarray}
$$

Applying the fourier inversion theorem, \eqref{eqj} translates to

$$
\begin{eqnarray} \label{eqk}
\int_{-\infty}^{\infty}  e^{-i2\pi \omega x} i(x) dx & = &  \delta(\omega)
\end{eqnarray}
$$

So \eqref{eqi} can be continued as,



$$
\begin{eqnarray} \label{eql}
C(\omega) & = & \int_{-\infty}^{\infty} \frac{1}{2} ( e^{-i2\pi(\omega + \omega_{0}) t} + e^{-i2\pi (\omega - \omega_{0}) t} ) dt \\
& = &  \frac{1}{2} (\delta(\omega + \omega_0) + \delta(\omega - \omega_0)) \\

\end{eqnarray}
$$

Similarly, from \eqref{eqg}, fourier transform $$S(\omega)$$of sine function $$s(t) = sin(2\pi\omega_0 t)$$ is given by,

$$
\begin{eqnarray} \label{eqm}
S(\omega) & = & \frac{i}{2} (\delta(\omega + \omega_0) - \delta(\omega - \omega_0))
\end{eqnarray}
$$


###  Derivation of fourier transform of a 2D gabor function.

From \eqref{eqs2},


$$
\begin{eqnarray} \label{eqs3}
g(x, y) & = & g_e(x, y) + i g_o(x, y) \\
 & = & \frac{1}{2\pi\sigma_{x}\sigma_{y}} e ^{-\frac{1}{2}(\frac{x^2}{\sigma_{x}^{2}} + \frac{y^2}{\sigma_{y}^{2}})} e^{i2\pi(u_1 x + v_1 y)} \\
 g_1(x) & = & \frac{1}{\sqrt{2\pi}\sigma_{x}} e ^{- \frac{x^2}{2\sigma_{x}^{2}}}  e^{i2\pi u_1 x} \\
 g_2(y) & = & \frac{1}{\sqrt{2\pi}\sigma_{y}} e ^{- \frac{y^2}{2\sigma_{y}^{2}}}  e^{i2\pi v_1 y} \\
 g(x, y) & = & g_1(x) \space g_2(y)
\end{eqnarray}
$$

So by \eqref{eqad}, \eqref{eqp2}, the fourier transform $$G(u, v)$$ of a 2D gabor function given in \eqref{eqs3} is,



$$
\begin{eqnarray} \label{eqs4}
G(u, v) & = & G_1(u) \space G_2(v) \\
 & = & H(u - u_1) H(v - v_1)
\end{eqnarray}
$$

The same principles can be extended to the generalized gabor filter function we formulated in \eqref{eqw}, by replacing
$$u'_1 = u_1 cos(\alpha) - v_1 sin(\alpha) $$ and $$v'_1 = u_1 sin(\alpha) + v_1 cos(\alpha) $$.

$$
\begin{eqnarray} \label{eqs5}
g(x, y) & = & \frac{1}{2\pi\sigma_{x}\sigma_{y}} e ^{-\frac{1}{2}(\frac{x^2}{\sigma_{x}^{2}} + \frac{y^2}{\sigma_{y}^{2}})} e^{i(2\pi(u_1 x' + v_1 y') + \phi)}\\
e^{i(2\pi(u_1 x' + v_1 y') + \phi)}& = &  e^{i(2\pi(u_1 ( x cos(\alpha) + y sin(\alpha)) + v_1 ( -x sin(\alpha) + y cos(\alpha)) ) + \phi)}\\
& = &  e^{i(2\pi(x (u_1 cos(\alpha) - v_1 sin(\alpha))))} e^{i(2\pi(y (u_1 sin(\alpha) + v_1 cos(\alpha))))}\\
& = &  e^{i 2\pi x u'_1 }  e^{i 2\pi y v'_1 }\\
g(x, y) & = &  \frac{1}{\sqrt{2\pi}\sigma_{x}} e ^{- \frac{x^2}{2\sigma_{x}^{2}}}  e^{i 2\pi x u'_1 } \space \frac{1}{\sqrt{2\pi}\sigma_{y}}  e ^{- \frac{y^2}{2\sigma_{y}^{2}}} e^{i 2\pi y v'_1 } \space e^{i\phi}
\end{eqnarray}
$$

This will give rise to the following formula for the fourier transform of $$g(x,y)$$ as written in \eqref{eqw},


$$
\begin{eqnarray} \label{eqs6}
G(u, v) & = &  H(u - u'_1) H(v - v'_1) \space e^{i\phi}
\end{eqnarray}
$$


## References

1. Abramowitz, M. and Stegun, I. A. (Eds.). Handbook of Mathematical Functions with Formulas, Graphs, and Mathematical Tables, 9th printing. New York: Dover, p. 302,

2. [Proof of Fourier inversion theorem] (http://math.columbia.edu/~nsnyder/tutorial/lecture3.pdf)