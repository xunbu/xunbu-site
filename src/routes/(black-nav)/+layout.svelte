<script>

</script>

<svelte:head>
    <link rel="stylesheet" href="/base.css">
</svelte:head>


<div class="container">
    <nav>
        <a href="/">首页</a>
        <a rel="external" href="https://xunbu.cc/search">木鱼起始页</a>
        <a href="/syncvideo">视频同步</a>
        <a href="/about">关于</a>
    </nav>
    <slot/>

</div>


<style>
    html {
        font-size: 16px;
    }

    @media (max-width: 768px) {
        :global(html) {
            font-size: 14px;
        }
    }

    @media (max-width: 480px) {
        :global(html) {
            font-size: 12px;
        }
    }

    .container {
        z-index: 999;
        display: flex;
        flex-direction: column;
        min-height: 100vh;
        justify-content: flex-start;
    }

    nav {
        /*padding: 0 5rem;*/
        margin: 1rem 0;
        width: 100%;
        height: 5vh;
        display: flex;
        justify-content: space-around;
        align-items: center;
        font-size: 2rem;
    }


</style>