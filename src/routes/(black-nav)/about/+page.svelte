<script >
    import {fade} from "svelte/transition";
</script>
<div class="container" in:fade={{duration: 1000 }}>
    <div class="card-container">
        <img class="card-img" src="/漩涡鸣人.jpg" alt="站主头像">
        <a href="https://github.com/xunbu/"><h1>寻步</h1></a>
        <h2>业余时间做一点简单页面。。。</h2>
        <a href="https://xunbu.cc/messageboard?key=xunbu"><h2>留言板</h2></a>
    </div>
</div>

<style>
    .container{
        margin-top: 3rem;
        display: flex;
        flex-direction: column;
        align-items: center;
    }
    .card-container{
        padding: 3rem;
        display: flex;
        flex-direction: column;
        align-items: center;
        border: solid black;
        border-radius: 10px;
    }
    .card-container>*{
        margin: 1rem;
    }
    .card-img {
        width: 10vw;
        height: 10vw;
    }
</style>