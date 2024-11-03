<script>
    import Hls from "hls.js";
    import { onMount } from "svelte";

    let main_video;
    let streams = [];
    let currentVideoSrc = '';
    let activeIndex = 0;
    let blip;
    let chunk;
    const FISHTANK_STREAMS = "https://ft.93.gay/streams.m3u8";
    //const FISHTANK_STREAMS = "https://ft-hetzner.3045x.com/streams";

    const config = {
        autoStartLoad: true,
        startPosition: -1,
        liveSyncDurationCount: 3,
        debug: false,
        enableWorker: true,
    };

    onMount(async () => {
        main_video = document.getElementById("video");
        blip = new Audio('blip.mp3');
        chunk = new Audio('chunk-short.mp3')
        try {
            const response = await fetch(FISHTANK_STREAMS);
            if (!response.ok) {
                throw new Error('Network response was not ok');
            }
            let lines = await response.text();
            let lines_arr = lines.split("\n").slice(1, 49);
            for (let i = 0; i < lines_arr.length; i += 2) {
                streams.push({ "description": lines_arr[i].slice(36), "stream": lines_arr[i + 1], "active": false });
            }
            streams = [...streams];
            console.log(streams);
            
            
            if (streams.length > 0) {
                currentVideoSrc = streams[0].stream;
                set_up_main_video();
            }
        } catch (err) {
            console.error(err);
        }
    });

    function parse_links_old(lines) {
        let lines_arr = lines.split("\n").slice(1, 49);
        for (let i = 0; i < lines_arr.length; i += 2) {
            streams.push({ "description": lines_arr[i].slice(36), "stream": lines_arr[i + 1] });
        }
    }

    function set_up_main_video() {
        if (main_video.canPlayType("application/vnd.apple.mpegurl")) {
            main_video.src = currentVideoSrc;
        } else if (Hls.isSupported()) {
            const hls = new Hls(config);
            hls.loadSource(currentVideoSrc);
            hls.attachMedia(main_video);
        }
        main_video.play();
    }

    function selectVideo(stream, index) {
        activeIndex = index;
        currentVideoSrc = stream;
        play_chunk();
        set_up_main_video();
    }

    function handle_key_down(e) {
        if (![37, 39].includes(e.keyCode)) {
            return;
        }
        console.log(e.keyCode);
        console.log(activeIndex);
        let nextIndex = activeIndex;

        switch (e.keyCode) {
            case 37: // left arrow (move left)
                nextIndex = activeIndex > 0 ? activeIndex - 1 : streams.length - 1; // Wrap around to the last item if at the beginning
                break;
            case 39: // right arrow (move right)
                nextIndex = activeIndex < streams.length - 1 ? activeIndex + 1 : 0; // Wrap around to the first item if at the end
                break;
        }

        // Ensure nextIndex is within bounds before accessing streams[nextIndex]
        if (streams[nextIndex]) {
            selectVideo(streams[nextIndex].stream, nextIndex);
        }
    }

    function play_blip() {
        if (blip) {
            blip.play();
        }
    }

    function play_chunk() {
        if (chunk) {
            chunk.play();
        }
    }
</script>

<style>
    .active {
        border-color: red;
    }
</style>

<div class="flex flex-col xl:flex-row">
    <video controls autoplay muted id="video" class="w-full max-h-screen h-auto xl:w-11/12"></video>

    <div class="w-full xl:w-1/12">
        {#if streams.length > 0}
            <div class="flex flex-wrap">
                {#each streams as stream, index}
                    <!-- svelte-ignore a11y_click_events_have_key_events -->
                    <!-- svelte-ignore a11y_no_static_element_interactions -->
                    <div class="btn btn-outline btn-sm grow m-1 text-xs {activeIndex === index ? 'active' : ''}" 
                    on:click={() => selectVideo(stream.stream, index)}
                    on:mouseenter={() => play_blip()}>
                        <p>{stream.description}</p>
                    </div>
                {/each}
            </div>
        {:else}
            <p>No streams available.</p>
        {/if}
    </div>
</div>

<svelte:window on:keydown|preventDefault={handle_key_down}/>