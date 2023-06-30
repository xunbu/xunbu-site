<script>
</script>

<div class="container">
    <h1>本页面使用svelte制作</h1>
    <ul class="svelte-list">
        <li><a href="https://svelte.dev">svelte<img src="./svelte.png" alt=""></a></li>
        <li><a href="https://kit.svelte.dev">sveltekit<img src="./svelte.png" alt=""></a></li>
        <li><a href="https://learn.svelte.dev">svelte交互式学习<img src="./svelte.png" alt=""></a></li>
        <li><a href="https://svelte.dev/repl">svelte演练场<img src="./svelte.png" alt=""></a></li>
    </ul>
</div>
<footer>
    <a href="https://beian.miit.gov.cn/">桂ICP备2022003415号</a>
    <a href="https://www.beian.gov.cn/portal/registerSystemInfo?recordcode=31012002005566">沪公网安备31012002005566号</a>
</footer>

<style>
.container{
    /*width: 100%;*/
    margin-top: 18rem;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}

.svelte-list img{
    height: 1em;
}
.svelte-list{
    font-size: 2rem;
}
.svelte-list>li{
    margin: 1rem;
}
footer {
    /*position: fixed;*/
    display: flex;
    flex-direction: row;
    justify-content: space-around;
    align-items: center;
    margin-top: auto;
    margin-bottom: 10px;
    width: 100%;
}
</style>