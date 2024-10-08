---
title: "Scaling Machine Learning with Rust"
subtitle: "My Journey to High Performance"
date: 2024-09-28T13:57:45+03:00
draft: false

tags: ["rust", "machine-learning", "optimization", "scaling", "data science"]
categories: ["Rust", "Machine Learning", "Engineering"]
---

In the past few months, I faced a challenge: creating a service to serve a machine learning model at high scale with fast response times. My choice of language was Rust 🦀, and after tests seemed fine, live results showed high CPU consumption and failure to hit the needed scale. Needless to say, this was far from my expectations, so I started my investigation 🕵🏻‍♂️.

## Profiling the Code

My first go-to solution was to **profile my code**. Results showed the Logistic Regression `predict` function was **single-threaded** and **CPU-heavy**. I initially tried parallelizing it, assuming that would improve performance—but it didn’t. The bottleneck persisted.

Frustrated, I asked a colleague for a fresh perspective. After some thought, his advice was to **measure execution time** to get a clearer picture of what was happening. That’s when it clicked—his advice reminded me that **CPU consumption and execution time are linked**.

```rust
// Original (inefficient) predict function
INTERCEPT: f32 = ... ;
COEFFICIENTS: CsVec<f32> = ... ;

fn predict(features:&CsVec<f32>) -> f32 { 
    COEFFICIENTS.dot(&features) + INTERCEPT
}
```

## The Problem with the Predict Function

The `predict` function in Logistic Regression is pretty straightforward—compute the dot product of feature and coefficient vectors, then add the intercept. 

The real challenge boiled down to two optimizations:

1. How to store feature values and coefficients.
2. How to calculate the dot product.

What I initially thought was a no-brainer (using sparse matrices with built-in `.dot` functions) turned out to be the bottleneck. A simple test showed that combining a **dense vector for coefficients** and **sparse vector for features** reduced computation latency from **20 ms to less than 1 ms**, and CPU consumption dropped from **80% to below 5%**.

## Why This Works

You might wonder why this approach worked. The answer lies in Rust’s **dense vectors** allowing `O(1)` access to coefficients, while **sparse vectors** efficiently keep feature values and indices. This resulted in a major performance boost—cutting latency and CPU usage dramatically.

```rust
// New (efficient) predict function
INTERCEPT: f32 = ... ;
COEFFICIENTS: Vec<f32> = ... ;

fn predict(features:&CsVec<f32>) -> f32 { 
    features.indices().iter().map(|&i| COEFFICIENTS[i] * features.get(i).unwrap()).sum::<f32>() + INTERCEPT
}
```

## Key Takeaways:

1. **Don’t hesitate to ask for help**—fresh eyes can spot what you miss.
2. **Always profile and measure execution time early**—it saves time and reveals the real issues.
3. **Don’t blindly trust built-in functions**. While they’re good general solutions, custom implementations may be better suited for your needs.


---
If you like this post,  please let me know by following me on X or sharing it with your network. I'd love to hear your thoughts! - {{< person url="https://x.com/theshlomigreen" name="Shlomi Green" nick="@theshlomigreen" text="Author" picture="https://pbs.twimg.com/profile_images/1707313015517859840/8lWHx6VR_400x400.jpg" >}}
