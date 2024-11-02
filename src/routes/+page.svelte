<script>
    import Hls from "hls.js";
    import { onMount } from "svelte";

    let main_video;
    let streams = [];
    let currentVideoSrc = '';
    const FISHTANK_STREAMS = "https://ft.93.gay/streams.m3u8";

    onMount(async () => {
        main_video = document.getElementById("video");
        try {
            const response = await fetch(FISHTANK_STREAMS);
            if (!response.ok) {
                throw new Error('Network response was not ok');
            }
            let lines = await response.text();
            let lines_arr = lines.split("\n").slice(1, 49);
            for (let i = 0; i < lines_arr.length; i += 2) {
                streams.push({ "description": lines_arr[i].slice(36), "stream": lines_arr[i + 1] });
            }
            streams = [...streams];
            
            if (streams.length > 0) {
                currentVideoSrc = streams[0].stream;
                set_up_main_video();
            }
        } catch (err) {
            console.error(err);
        }
    });

    function set_up_main_video() {
        if (main_video.canPlayType("application/vnd.apple.mpegurl")) {
            main_video.src = currentVideoSrc;
        } else if (Hls.isSupported()) {
            const hls = new Hls();
            hls.loadSource(currentVideoSrc);
            hls.attachMedia(main_video);
        }
        main_video.play();
    }

    function selectVideo(stream) {
        currentVideoSrc = stream;
        set_up_main_video();
    }
</script>

<style>

</style>

<div class="flex flex-col xl:flex-row">
    <video controls autoplay muted id="video" class="w-full max-h-screen h-auto xl:w-11/12"></video>

    <div class="w-full xl:w-1/12">
        {#if streams.length > 0}
            <div class="flex flex-wrap">
                {#each streams as stream}
                    <div class="btn btn-outline btn-sm grow m-1 text-xs" on:click={() => selectVideo(stream.stream)}>
                        <p>{stream.description}</p>
                    </div>
                {/each}
            </div>
        {:else}
            <p>No streams available.</p>
        {/if}
    </div>
</div>
