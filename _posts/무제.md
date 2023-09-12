Rejection sampling
================











## 목적

- 고등학생 미적분시간에 적분을 못하는 함수가 있다는 걸 경험 했다. ex)
  $e^{x^2}$ , $\cdots$

- 통계를 공부하는 입장으로써 적분이 필요한 계산이 많이 있다. ex) n차
  기댓값, 분산, $\cdots$

- 그런데 직접적분이 어려운 분포들이 있을 때 난감하다. ex) 베이지안의
  사후분포, 다변량 분포

- 그렇다면 포기해야 할까?

$\hspace{20em}$

## Monte carlo

문제를 해결하는 방법으로 Monte carlo 방법이 있다.

by Law of Large Number

## 

``` r
n <- 10^4
a <- numeric(n)
y <- rnorm(n)
for (i in 1:n){
  a[i] <- mean(y[1:i])
}
iteration <- 1:n 
sample_mean <- a
X <- data.frame(iteration,sample_mean)
```

``` r
library(ggplot2)
ggplot() +
  ggtitle("N(0,1)") +
  ylim(-max(sample_mean),max(sample_mean)) +
  geom_point(mapping = aes(x=iteration,y=sample_mean), data=X)
```

    ## Warning: Removed 2 rows containing missing values (`geom_point()`).

![](무제_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

## 다른 문제점

- 내가 가정한 분포의 샘플(데이터)가 많으면 바로 MC를 사용하면 된다.

- 그렇지 않을 때는 어떻게 해야 할까.

-\> 직접 샘플을 만들어야 한다.

- 보통 rnorm, rpoi, rgamma, $\cdots$ 으로 난수를 발생시켜야 한다.

- 그런데, 친숙한 분포가 아니라면? ex) 베이지안 사후분포

# Rejection Sampling

## 그전에

분포함수 $F(X) = \int^x_{-\infty} f(t) dt$ 가 $U[0,1]$과 관계가 있다.

- $F(X)$는 monotone function 이다.

- $\lim_{X\to -\infty} F(X) = 0, \lim_{X\to\infty} F(X) = 1$

- 그러므로 $F(X)$는 $U[0,1]$ 이 일대일 관계를 갖기에 역함수를 가질 수
  있다.

- $F(X) = U$ 라고 잡으면 밑에 처럼 볼 수 있다.

## Rejection Sampling

- “acceptance-rejection method” 또는 “accept-reject algorithm” 이라고도
  부른다.

1.  내가 다루기 쉬운 proposal density $g(y)$ 가 필요하다.

2.  목적에 맞게끔 M을 설정해야 한다.

3.  $u_i$ ~ U$$0,1$$ 인 난수를 만든다.

4.  $y_i$ 난수를 만든다.

5.  알고리즘을 실행한다.

- $u_i \leq \frac{f(y_i)}{M g(y_i)}$ 를 만족하면 accept 그러지 않으면
  reject 한다.

## 

확률로써 저 방법도 맞는 건가 확인해보자.

## 

예시로 쓸 함수를 설정해보자

$E(h(x))$ 를 구할 때, $h$ 와 $X$의 밀도함수는 밑에와 같다.

``` r
h <- function(x){exp(-x + cos(x))}
f <- function(x){exp(-x)}
```

where, $x\in [0,2.5]$

## 

``` r
curve(h(x)*f(x), 0, 2.5, add = FALSE, ylim=c(0,3),
      type = "l", lwd=3)
abline(a = 1/2.5, b = 0, col = "green", lwd=3)
abline(a = exp(1), b = 0, col = "orange" ,lwd=3)
legend("right", legend = c("h(x)*f(x)","Proposal",
                           "M*Proposal"),
       col = c("black","green","orange"), lwd=3, cex=1.2)
```

## 

``` r
curve(h(x)*f(x), 0, 2.5, add = FALSE, ylim=c(0,3), type = "l", lwd=3)
abline(a = 1/2.5, b = 0, col = "green", lwd=3)
abline(a = exp(1), b = 0, col = "orange" ,lwd=3)
legend("right", legend = c("h(x)*f(x)","Proposal","M*Proposal"),
       col = c("black","green","orange"), lwd=3, cex=1.2)
```

![](무제_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

## 

``` r
m <- 1000
M <- exp(1)*2.5
X <- runif(m, 0, 2.5)
U <- runif(m, 0, 1)

target <- function(x){h(x)*f(x)}
proposal <- function(x){dunif(x, 0, 2.5)}

accept <- U < (target(X)/(M*proposal(X)))
```

## 

``` r
curve(target, 0, 2.5, ylim=c(0,2.8), lwd=3)
curve(proposal, add=TRUE, col="green", lwd=3)
curve(M*proposal(x), add=TRUE, col="orange", lwd=3)
points(X, exp(1)*U, pch=ifelse(accept,1,4),
       col=ifelse(accept,"blue","red"),lwd=2)
legend("topright", c("target","proposal","M*proposal",
                     "accepted","rejected"), 
       lwd=c(3,3,3,NA,NA), col=c("black","green","orange",
                                 "blue","red"),
       pch=c(NA,NA,NA,1,4), bg="white", cex=1.2)
```

## 

``` r
curve(target, 0, 2.5, ylim=c(0,2.8), lwd=3)
curve(proposal, add=TRUE, col="green", lwd=3)
curve(M*proposal(x), add=TRUE, col="orange", lwd=3)
points(X, exp(1)*U, pch=ifelse(accept,1,4),
       col=ifelse(accept,"blue","red"),lwd=2)
legend("topright", c("target","proposal","M*proposal","accepted","rejected"), 
       lwd=c(3,3,3,NA,NA), col=c("black","green","orange","blue","red"),
       pch=c(NA,NA,NA,1,4), bg="white", cex=1.2)
```

![](무제_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

## 

target 함수의 정확한 적분값은 1.16… 정도이다.

``` r
mean(h(accept)) 
```

    ## [1] 2.369785

``` r
sqrt(var(accept)/sum(accept))
```

    ## [1] 0.02887618

``` r
c(sum(accept)/m, 1/M)
```

    ## [1] 0.1670000 0.1471518

## 결론

장점

- 내가 잘 모르는 함수의 샘플을 만들 수 있다는 걸 알게 되었다.

- Monte Carlo 가 쉬우면서 중요하단 걸 알게 되었다.

단점

- 제안 분포를 설정해야 한다.

- 데이터의 분포가 어떠한 분포인지 잘 알아야 한다.

- M이란 값을 셋팅해야 한다.

- 

## 다음시간에는

버려지는 샘플을 보안하기 위해 Importace Sampling 을 설명하려 합니다.

그 후에는 Markov Chain Monte Carlo 방법을 이용한 샘플링도 설명하려
합니다.
