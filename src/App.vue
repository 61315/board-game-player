<script setup>
import { onMounted, ref, reactive } from "vue";
import KeyboardEvents from "./components/KeyboardEvents.vue";
import VideoPlayer from "./components/VideoPlayer.vue";

const keyboardEvent = function (event) {
    pressedKey.value = event.code;

    try {
        switch (event.which) {
            case 37: // 37, ArrowLeft
                if (player.currentTime() === 0) {
                    // Load previous playlist item
                    if (player.playlist.previous()) {
                        player.play().catch((reason) => {
                            console.warn(reason);
                        });
                    }
                } else {
                    // Seek to the start of the current video
                    player.pause();
                    player.currentTime(0);
                }
                break;
            case 39: // 39, ArrowRight
                if (player.currentTime() === player.duration()) {
                    // Load next playlist item
                    if (player.playlist.next()) {
                        player.play().catch((reason) => {
                            console.warn(reason);
                        });
                    }
                } else {
                    // Seek to the end of the current video
                    player.pause();
                    player.currentTime(player.duration());
                }
                break;
            case 32: // 32, Space
                if (player.ended()) {
                    // Proceed to the next video clip if the player has finished the current video
                    player.playlist.next();
                } else {
                    // Do a regular play/pause toggle if the player is still playing
                    if (player.paused()) {
                        player.play().catch((reason) => {
                            console.warn(reason);
                        });
                    } else {
                        player.pause();
                    }
                }
                break;
            case 13: // 13, Enter
                // Toggle fullscreen on keydown event with "Enter"
                if (player.isFullscreen()) {
                    player.exitFullscreen();
                } else {
                    player.requestFullscreen();
                }
                break;
            case 77: // 77, m or M
                // Toggle mute on keydown event with "m" or "M"
                player.muted(!player.muted());
                break;
        }
    } catch (error) {
        console.error(error);
    }
};

const videos = [
    "https://user-images.githubusercontent.com/46559594/136930700-b416598a-6ced-4a68-8003-0ae7e95d45d3.mp4",
    "https://user-images.githubusercontent.com/46559594/136930727-5084edf7-dc0d-4b5c-89f2-caf4daab44ba.mp4",
    "https://user-images.githubusercontent.com/46559594/136930408-fa8d3fc4-54c7-47d0-a5d7-ae0f85fc61b4.mp4",
];

const pressedKey = ref("None");
const triggeredEvent = ref("None");
const playlistInfo = ref("");

const videoElement = ref(null);

var player = null;

const videoOptions = reactive({
    autoplay: false,
    controls: false,
    preload: "auto",
    plugins: {
        playlist: videos.map((v) => {
            return {
                sources: [{ src: v, type: "video/mp4" }],
                //poster: thumbnail,
            };
        }),
    },
    userActions: {
        // Disable all the player-wide hotkeys and touchevents
        doubleClick: false,
        hotkeys: false,
        /*
        hotkeys: {
            muteKey: function (event) {
                // Toggle mute on keydown event with "m" or "M"
                return event.which === 77;
            },
            fullscreenKey: function (event) {
                // Toggle fullscreen on keydown event with "Enter"
                return event.which === 13;
            },
            playPauseKey: function (event) {
                // Toggle play/pause on keydown event with "Space"
                return event.which === 32;
            },
        },
        */
    },
});

const onPreviousButtonPressed = function () {
    if (player.currentTime() === 0) {
        // Load previous playlist item
        if (player.playlist.previous()) {
            player.play().catch((reason) => {
                console.warn(reason);
            });
        }
    } else {
        // Seek to the start of the current video
        player.pause();
        player.currentTime(0);
    }
};

const onNextButtonPressed = function () {
    if (player.currentTime() === player.duration()) {
        // Load next playlist item
        if (player.playlist.next()) {
            player.play().catch((reason) => {
                console.warn(reason);
            });
        }
    } else {
        // Seek to the end of the current video
        player.pause();
        player.currentTime(player.duration());
    }
};

onMounted(() => {
    // Get the videojs instance from the child component named `VideoPlayer`
    player = videoElement.value.player;

    const eventsToWatch = [
        "canplay",
        //"canplaythrough",
        "ended",
        "emptied",
        "fullscreenchange",
        "pause",
        "play",
        "playing",
        "ready",
        "seeking",
        "stalled",
        "suspend",
        "volumechange",
        "waiting",
    ];

    player.on(eventsToWatch, function (event) {
        triggeredEvent.value = event.type;
    });

    player.on("playlistitem", function (event) {
        playlistInfo.value = `${player.playlist.currentIndex() + 1} / ${
            player.playlist.lastIndex() + 1
        }`;
    });

    // Load the first item in the playlist and play it
    player.playlist.first();
    player.play().catch((reason) => {
        console.warn(reason);
    });
});
</script>

<template>
    <div class="w-full max-h-screen mx-auto border-green-400 border-2 rounded-md" style="height: calc(100vw * 9 / 16); max-width: calc(100vh * 16 / 9);">
        <div class="w-full h-full absolute top-0 left-0">
            <VideoPlayer ref="videoElement" :options="videoOptions" class="w-full h-full object-center object-cover" />
        </div>
        <div class="flex flex-col justify-between w-full h-full absolute top-0 left-0 bg-none overflow-visible">
            <div class="flex flex-wrap justify-between">
                <button type="button" class="flex-none m-4 px-2.5 py-1.5 border border-transparent text-xs font-medium rounded shadow-sm text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500">Key Pressed: {{ pressedKey }}</button>

                <button type="button" class="flex-none place-self-center m-4 px-2.5 py-1.5 border border-transparent text-xs font-medium rounded shadow-sm text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500">{{ playlistInfo }}</button>

                <button type="button" class="flex-none m-4 px-2.5 py-1.5 border border-transparent text-xs font-medium rounded shadow-sm text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500">Last Event: {{ triggeredEvent }}</button>
            </div>

            <div class="flex justify-between">
                <button type="button" @click="onPreviousButtonPressed" class="flex-none w-12 h-12 m-4 px-2.5 py-1.5 border border-transparent text-2xl font-medium rounded shadow-sm text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" />
                    </svg>
                </button>

                <button type="button" @click="onNextButtonPressed" class="flex-none w-12 h-12 m-4 px-2.5 py-1.5 border border-transparent text-2xl font-medium rounded shadow-sm text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7m0 0l-7 7m7-7H3" />
                    </svg>
                </button>
            </div>
        </div>
    </div>

    <!-- Global keyup event observer -->
    <KeyboardEvents @keyup.prevent="keyboardEvent"></KeyboardEvents>
</template>

<style>
#app {
    font-family: Inter Var, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
}
</style>
