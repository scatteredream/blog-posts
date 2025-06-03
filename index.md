---
date: 2025-05-17 20:56:30
---

<blockquote class="blockquote-center">树欲静而风不止。</blockquote>


<style>

/* 新增包裹容器 */
.grid-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    /* min-height: 100vh;  如果不需要全屏居中可删除这行 */
    padding: 2rem;
}
.grid-container {
    display: grid;
    gap: 0.3rem;
    padding: 2rem;
    background: ;
    border-radius: 16px;
    box-shadow: 0 8px 30px rgba(0,0,0,0.12);
    max-width: 1200px;
}
/* 默认：手机端（1行3列） */
.grid-container {
    grid-template-columns: repeat(3, 1fr);
}

/* 平板端（1行4列） */
@media (min-width: 768px) {
    .grid-container {
        grid-template-columns: repeat(4, 1fr);
    }
}

/* 电脑端（1行5列） */
@media (min-width: 1024px) {
    .grid-container {
        grid-template-columns: repeat(5, 1fr);
    }
}

/* 可选：超大屏幕（1行6列） */
@media (min-width: 1440px) {
    .grid-container {
        grid-template-columns: repeat(6, 1fr);
    }
}


.grid-item {
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 1rem;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.grid-link {
    position: relative;
    display: block;
    width: 100%;
    padding: 1.2rem 1.5rem;
    text-decoration: none;
    color: #2c3e50;
    font-weight: 600;
    font-size: 1rem;
    background: white;
    border-radius: 10px;
    box-shadow: 0 3px 12px rgba(0,0,0,0.1);
    transition: all 0.3s ease;
    text-align: center;
}

/* 悬停动画 */
.grid-link:hover {
    transform: translateY(-3px);
    box-shadow: 0 6px 20px rgba(52,152,219,0.2);
    color: #3498db;
}

/* 点击效果 */
.grid-link:active {
    transform: translateY(0);
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

/* 下划线动画 */
.grid-link::after {
    content: '';
    position: absolute;
    width: 0;
    height: 2px;
    bottom: 8px;
    left: 50%;
    background: #3498db;
    transition: all 0.3s ease;
}

.grid-link:hover::after {
    width: 70%;
    left: 15%;
}

/* 状态指示
.grid-link:visited {
    color: #8e44ad;
} */

/* 九宫格边框动画 */
.grid-item:hover::before {
    content: '';
    position: absolute;
    top: -2px;
    left: -2px;
    right: -2px;
    bottom: -2px;
    border: 2px solid rgba(52,152,219,0.3);
    border-radius: 12px;
    animation: borderGlow 1.5s infinite;
}

@keyframes borderGlow {
    0% { opacity: 0.8; }
    50% { opacity: 0.3; }
    100% { opacity: 0.8; }
}
</style>
<div class="grid-wrapper">
<div class="grid-container">
<div class="grid-item">
        <a href="https://scatteredream.github.io/2025/05/13/algorithm-sort/" class="grid-link">Sort</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/2025/05/17/algorithm-leetcode-hot-100/" class="grid-link">Hot100</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/categories/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/" class="grid-link">Design</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/2025/02/03/rpc-interpretation/" class="grid-link">RPC</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/2024/10/20/redis-review-optimization/" class="grid-link">Review</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/categories/spring/" class="grid-link">Spring</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/categories/redis/" class="grid-link">Redis</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/categories/mysql/" class="grid-link">MySQL</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/categories/web-development/" class="grid-link">Web</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/categories/jdk/" class="grid-link">Java</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/categories/juc/" class="grid-link">JUC</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/categories/jvm/" class="grid-link">JVM</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/categories/%E8%AE%A1%E7%BD%91/" class="grid-link">计网</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/categories/OS/" class="grid-link">OS</a>
    </div>
    <div class="grid-item">
        <a href="https://scatteredream.github.io/categories/review/" class="grid-link">Interview</a>
    </div>
</div>
</div>