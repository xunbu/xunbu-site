<script>
    import {onDestroy, tick} from "svelte";
    import {fade} from "svelte/transition";

    let isSub = false//是否正在同步
    let src
    let selectVideo
    let syncCodeInput
    let syncCode
    let eventSource
    let lastUpdateTime = 0//最后一次发送的时间（单位s）
    let video
    let publishTimer
    let showCopyTips = false
    let showCode//显示的syncCodeElem，用于选中复制
    let hasVideoListener = false//是否在监听video变化
    let parseSrc


    function addVideoListener() {
        if (!hasVideoListener) {
            console.log("开始监听")
            video.addEventListener("seeking", publishHandler)//改变进度条，准备完成时触发
            video.addEventListener("pause", publishHandler)//暂停时触发
            video.addEventListener("play", publishHandler)//播放时触发
            hasVideoListener = true
        }
    }

    function removeVideoListener() {
        if (hasVideoListener) {
            console.log("移除监听")
            video.removeEventListener("seeking", publishHandler)//改变进度条，准备完成时触发
            video.removeEventListener("pause", publishHandler)//暂停时触发
            video.removeEventListener("play", publishHandler)//播放时触发
            hasVideoListener = false
        }
    }

    async function waitUntilCanplay() {
        return new Promise((resolve) => {
            function canPlayHandler() {
                console.log("可以播放")
                const delTime=(new  Date()-time1)/1000
                resolve(delTime)
                console.log("等待完成时间：",delTime)
                video.removeEventListener("canplay", canPlayHandler)
            }
            let time1=new Date()
            video.addEventListener("canplay", canPlayHandler)
        });
    }

    async function autoPlayandStop() {//自动播放并暂停
        setTimeout(() => {
            removeVideoListener()
            let playPromise = video.play();
            if (playPromise !== undefined) {
                playPromise.then(_ => {
                    video.pause();
                })
            }
            setTimeout(addVideoListener)
        })
    }

    async function localVideoClickHandler() {
        if (isSub){
            syncCodeInput=syncCode
            await unsubscribe()
        }
        src = URL.createObjectURL(selectVideo.files[0])
        await tick()
        await autoPlayandStop()
    }

    async function parseSrcClickHandler() {
        if (isSub){
            syncCodeInput=syncCode
            await unsubscribe()
        }
        src = parseSrc
        await tick()
        await autoPlayandStop()

    }

    async function startSyncCode() {
        //如果正在订阅，则取消当前订阅
        if (isSub) {
            await unsubscribe()
        }
        //开始同步
        if (syncCodeInput) {
            syncCode = syncCodeInput.toLowerCase()
            syncCodeInput = null
        } else {
            //获取同步码
            let response = await fetch("https://xunbu.cc/fastapi/syncmessage/getcode")
            if (response.ok) {
                const json = await response.json()
                syncCode=json.toLowerCase()
            }
        }
        await subscribe()
    }

    async function subscribe() {

        isSub = true
        // 开始订阅
        eventSource = new EventSource(`https://xunbu.cc/fastapi/syncmessage/syncode/${syncCode}`);
        eventSource.onmessage = async function (event) {
            console.log("开始接收")
            console.log("subscribe|eventdata:", event.data)
            //event.data=1686749758（单位s）&currentTime(单位s)&1(1表示暂停0表示播放)
            if (event.data === "") return;
            let [data_utcTime, data_currentTime, data_paused] = event.data.split("&")
            data_utcTime = Number(data_utcTime)
            data_currentTime = Number(data_currentTime)
            data_paused = Boolean(Number(data_paused))
            console.log("差值：", Math.abs(data_utcTime - lastUpdateTime))
            if (data_utcTime - lastUpdateTime > 0.01) {//检查是不是自己发的,且修改时间应在自己最后一次修改之前
                const oldfetchTime = new Date()
                let response = await fetch("https://xunbu.cc/fastapi/syncmessage/getsystime")
                const newfetchTime = new Date()
                const fetchTime = newfetchTime - oldfetchTime
                if (response.ok) {
                    removeVideoListener()
                    let now = await response.json()
                    lastUpdateTime = now = now["UTCtimestamp"]
                    console.log("fetchtime",fetchTime / 1000)
                    // const delTime = now - data_utcTime + data_currentTime - video.currentTime + Math.min(fetchTime / 900, 0.2)
                    const delTime = now - data_utcTime + data_currentTime - video.currentTime
                    let timeChanged = (Math.abs(delTime) > 0.01)//检测进度条是否改变
                    if(data_paused){//如果传来的数据是暂停的
                        video.currentTime = data_currentTime
                        video.pause()
                    }else {//如果传来的数据是播放的
                        video.currentTime += delTime + Math.min(fetchTime / 900, 0.2)
                        video.play()
                    }
                    await tick()
                    if (timeChanged) await waitUntilCanplay();//如果进度条改变则等待canplay
                    addVideoListener()
                }
            }
            console.log("结束接收")
        };

    }

    async function unsubscribe() {
        isSub = false
        console.log("结束订阅")
        eventSource.close()
        await fetch(`https://xunbu.cc/fastapi/syncmessage/syncode/${syncCode}`, {
            method: "POST",
            headers: {'Content-Type': 'application/json;charset=utf-8'},
            body: JSON.stringify({message_type: "unsubscribe"})
        })
        syncCode = null
    }

    async function publishHandler(e) {
        if (publishTimer) clearTimeout(publishTimer)
        publishTimer = setTimeout(async () => {
            await publish();
            publishTimer = null
        }, 500)

        async function publish() {
            if (!isSub) return;
            console.log("开始发布")
            console.log("publish|event:", e)
            //发布目前的状态
            // let response = await fetch("https://worldtimeapi.org/api/timezone/Etc/UTC")
            let response = await fetch("https://xunbu.cc/fastapi/syncmessage/getsystime")
            if (response.ok) {
                let now = await response.json()
                lastUpdateTime = now = now["UTCtimestamp"]//获取当前时间(s)
                await fetch(`https://xunbu.cc/fastapi/syncmessage/syncode/${syncCode}`, {
                    method: "POST",
                    headers: {'Content-Type': 'application/json;charset=utf-8'},
                    body: JSON.stringify({
                        message: `${now}&${video.currentTime}&${video.paused ? 1 : 0}`,
                        message_type: "publish"
                    })
                });
            }
            console.log("结束发布")
        }
    }

    async function copyHandler() {
        if (!isSub) return;
        const range = document.createRange()
        range.selectNode(showCode)
        window.getSelection().removeAllRanges()
        window.getSelection().addRange(range)
        // 执行复制命令
        document.execCommand('copy')
        window.getSelection().removeAllRanges()
        showCopyTips = true
        setTimeout(() => {
            showCopyTips = false
        }, 600)
    }

    onDestroy(async () => {
        if (isSub) {
            //取消订阅
            await unsubscribe()
        }
    })
</script>


<div class="background"></div>
<div class="container main-container">
    <div class="video-container">
        <video {src} controls preload="auto"
               bind:this={video}
        ></video>
    </div>
    <div class="row-container" style="margin-top: 1rem">
        <div class="col-container">
            <label>本地视频：<label class="label-btn" for="selectVideo">选择本地视频</label></label>

            <input style="display: none" id="selectVideo" type="file" accept="video/*" bind:this={selectVideo}
                   on:change={localVideoClickHandler}>
            <div><label>解析链接： <input type="text" placeholder="请输入视频解析链接" bind:value={parseSrc}></label>
                <button on:click={parseSrcClickHandler}>播放</button>
            </div>
            <div>获取解析链接：</div>
            <div class="flex-start row-container href-list">
                <a href="https://pv.vlogdownloader.com/" target="_blank" style="color:lightskyblue">parsevideo</a>
                <a href="https://superparse.com/zh" target="_blank" style="color:lightskyblue">superparse</a>
                <a href="https://xbeibeix.com/api/bilibili/" target="_blank" style="color:lightskyblue">贝贝bilibili</a>
            </div>
            <div class="row-container flex-start href-list">
                <a href="http://zhouql.vip/bilibili/" target="_blank" style="color:lightskyblue">zhouql_bilibili</a>
            </div>
        </div>
        <div class="col-container">
            <button class="show-code" on:click={copyHandler} style="cursor: pointer">同步码: <span
                    bind:this={showCode}>{syncCode ? syncCode : "无"}</span>
                {#if showCopyTips}
                    <span transition:fade style="color: deepskyblue;font-size: 1.2rem;position: fixed">已复制</span>
                {/if}
            </button>
            <div class="col-container">
                <input style="min-width: 12em" type="text" placeholder="请输入同步码" bind:value={syncCodeInput}>
                {#if !isSub}
                    <button style="min-width: 12em"
                            on:click={startSyncCode}>{syncCodeInput ? "开始同步" : "生成同步码并同步"}</button>
                {:else }
                    <button style="min-width: 12em" on:click={unsubscribe}>停止同步</button>
                {/if}
            </div>
        </div>
    </div>
</div>

<style>
    .background{
        z-index: -1;
        position: fixed;
        height: 100vh;
        width: 100vw;
        background-color: rgb(95, 135, 73);
        background-image: url("/forest.svg");
        background-position: center;
        background-size: 100% auto
    }
    .main-container{
        z-index: 0;
        color: white;
    }
    .flex-start{
        justify-content: flex-start !important;
    }
    .container {
        height: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: flex-start;
    }
    .video-container{
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        width: 100vw;
        height: min(70vw,600px);
    }
    video {
        max-width: 100%;
        height: 100%;
    }

    .row-container {
        width: 100%;
        display: flex;
        align-items: stretch;
        flex-direction: row;
        justify-content: space-around;
        flex-wrap: wrap;
    }

    .col-container {
        display: flex;
        align-items: flex-start;
        flex-direction: column;
        justify-content: space-around;
        flex-wrap: wrap;
    }

    .col-container > * {
        margin-top: 1em;
    }

    .href-list > * {
        min-width: fit-content;
        margin: 0 1rem;
    }

    .label-btn {
        padding: 6px 25px;
        background: #00bfff;
        border-radius: 4px;
        color: white;
        cursor: pointer;
    }
    .show-code{
        font-size: 2em;
        font-weight: bold;
        border: none;
        background-color: transparent;
        color: inherit;
    }
</style>