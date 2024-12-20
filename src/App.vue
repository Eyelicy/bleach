<style>
@property --scroll-position {
    syntax: '<number>';
    inherits: true;
    initial-value: 0;
}
@property --scroll-position-delayed {
    syntax: '<number>';
    inherits: true;
    initial-value: 0;
}

@keyframes adjust-pos {
    to {
        --scroll-position: 1;
        --scroll-position-delayed: 1;
    }
}

:root {
    animation: adjust-pos 0.01s linear both; /* Duration added to cater for Firefox */
    animation-timeline: scroll(root);
}

body {
    transition: --scroll-position-delayed 0.15s linear;
    --scroll-velocity: calc(var(--scroll-position) - var(--scroll-position-delayed));
}

html {
    background: #666;
}

html {
    scroll-snap-type: y mandatory;
}

* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

ul {
    list-style: none;
    pointer-events: none;
    line-height: 1;
    display: block;
}

ul li {
    display: block;
    line-height: 1;
}

.panel {
    scroll-snap-align: center;
    height: 100dvh;
    border: 2px solid #333;
    background: rgb(0 0 0 / 0.15);
    z-index: -1;
}

.panel img {
    display: block;
    width: 100%;
    height: 100%;
    object-fit: cover;
    filter: brightness(30%);
}

@keyframes slide-up {
    to {
        translate: 0 var(--end-pos);
    }
}

.titles {
    position: fixed;
    z-index: 2;
    top: 50dvh;
    font-size: 15dvw;
    left: 0;
    right: 0;
    text-align: center;

    animation: slide-up 0.01s linear; /* Duration added to cater for Firefox */
    animation-timeline: scroll(root block);

    font-family: impact;
    text-transform: uppercase;
    letter-spacing: 1px;

    /* Calculate the part each child can occupy.
	    For 8 children, that’s 12.5%
	*/
    --num-children: 4;
    --num-children-1: calc(var(--num-children) - 1);
    --size-per-child: calc(100% / var(--num-children));

    /* Because the text needs to be in the dead center,
	   we need to shift up the whole thing by half the size per child.
	   If not, it would start with the top edge of the whole box
	   aligned to the center. We want the text to be centered.
	   So in the case of 8 items that is from -6.25% to to -(100% - 6.25%).
	   This range still spans 100% in total, but it is slightly shifted up.
	*/
    --half-size-per-child: calc(var(--size-per-child) / 2);
    --start-pos: calc(var(--half-size-per-child) * -1);
    --end-pos: calc(-100% + var(--half-size-per-child));
    translate: 0 var(--start-pos);
}

.titles.transparent {
    color: transparent;
    -webkit-text-stroke: 2px rgb(255 255 255 / 0.7);
    text-stroke: 1px rgb(255 255 255 / 0.7);
}

.titles.fill {
    color: white;

    /* Determine which element is currently in view. We do this based
	   on the current scroll-distance (exposed via --y-pos) and how
	   much space each child takes up.
	*/
    --y-per-child: calc(1 / var(--num-children));
    --cur-child: calc((var(--scroll-position) / var(--y-per-child)) + 1);
    --cur-child-1: calc(var(--cur-child) - 1);

    /* To adjust the clip path we need to adjust the total range
	   that is spanned. 
	*/
    --clip-min: 0;
    --clip-max: calc(100% - var(--size-per-child));
    --clip-per-child: calc(var(--clip-max) / var(--num-children));
    --fix: calc(
        var(--clip-per-child) / var(--num-children)
    ); /* Not sure why I need a fix like this ¯\_(ツ)_/¯ */
    --extra: 1%;
    clip-path: inset(
        calc((var(--cur-child-1) * var(--clip-per-child)) - var(--extra)) 0%
            calc((100% - (var(--cur-child) * var(--clip-per-child)) - var(--fix)) - var(--extra)) 0%
    );
}

/* And now, for the late scrolling part …*/
.titles li {
    will-change: transform;
    transform-origin: 50% 50%;
    transform: skewY(calc(var(--scroll-velocity) * -75deg));
}

/* Show warning for browsers without support */
.warning {
    border: 1px solid black;
    z-index: 9999;
    color: black;
    background: rgba(255 255 225 / 0.9);
    z-index: 1;

    position: fixed;
    top: 1em;
    left: 1em;
    right: 1em;
    text-align: center;
}

.warning p {
    margin: 1em 0;
}

/* Hide warning when native view-timeline support detected */
@supports (view-timeline: --works) {
    .warning {
        display: none;
    }
}

.music {
    position: fixed;
    top: 1em;
    right: 1em;
    z-index: 9999;
    animation: slide-up 0.01s linear; /* Duration added to cater for Firefox */
    animation-timeline: scroll(root block);
    cursor: pointer;
    border: 2px solid white;
    border-radius: 50%;
    animation: slide-up 0.01s linear;
}

.music.rotate {
    animation: rotate 2s infinite linear;
}

@keyframes rotate {
    from {
        transform: rotate(0deg);
    }
    to {
        transform: rotate(360deg);
    }
}
</style>

<template>
    <div>
        <svg
            xmlns="http://www.w3.org/2000/svg"
            height="24px"
            viewBox="0 -960 960 960"
            width="24px"
            fill="#fff"
            class="music"
            :class="{ rotate: audioRef }"
            @click="handleMusic"
        >
            <path
                v-if="audioRef"
                d="M400-120q-66 0-113-47t-47-113q0-66 47-113t113-47q23 0 42.5 5.5T480-418v-422h240v160H560v400q0 66-47 113t-113 47Z"
            />
            <path
                v-else
                d="M792-56 56-792l56-56 736 736-56 56ZM560-514l-80-80v-246h240v160H560v166ZM400-120q-66 0-113-47t-47-113q0-66 47-113t113-47q23 0 42.5 5.5T480-418v-62l80 80v120q0 66-47 113t-113 47Z"
            />
        </svg>
        <ul class="titles transparent">
            <li>Do not</li>
            <li>fear</li>
            <li>deception</li>
            <li>BLEACH</li>
        </ul>
        <ul class="titles fill">
            <li>Do not</li>
            <li>fear</li>
            <li>deception</li>
            <li>BLEACH</li>
        </ul>
        <div class="panels">
            <div class="panel">
                <img src="./assets/1.jpg" alt="photo" height="900" width="1600" />
            </div>
            <div class="panel">
                <img src="./assets/4.jpg" alt="photo" height="900" width="1600" />
            </div>
            <div class="panel">
                <img src="./assets/3.jpg" alt="photo" height="900" width="1600" />
            </div>
            <div class="panel">
                <img src="./assets/2.jpg" alt="photo" height="900" width="1600" />
            </div>
            <!-- <div class="panel">
                <img
                    src="./assets/5.jpg"
                    alt="photo"
                    height="900"
                    width="1600"
                />
            </div>
            <div class="panel">
                <img
                    src="https://picsum.photos/seed/6/1600/900"
                    alt="photo"
                    height="900"
                    width="1600"
                />
            </div>
            <div class="panel">
                <img
                    src="https://picsum.photos/seed/7/1600/900"
                    alt="photo"
                    height="900"
                    width="1600"
                />
            </div>
            <div class="panel">
                <img
                    src="https://picsum.photos/seed/8/1600/900"
                    alt="photo"
                    height="900"
                    width="1600"
                />
            </div> -->
        </div>

        <div class="warning">
            <p>
                ⚠️ Your browser does not support CSS Scroll-Linked Animations with
                <code>view-timeline</code>. Please use Chrome Canary 115+ to check out this demo.
            </p>
        </div>

        <audio src="/bg.mp3" loop="true" hidden="true"></audio>
    </div>
</template>

<script setup>
import { nextTick, onMounted, ref } from 'vue';

const audioRef = ref(null)

onMounted(async () => {
    await nextTick()
})

const handleMusic = () => {
    const audio = document.querySelector('audio')
    if (audio.paused) {
        audio.play()
        audioRef.value = true
    } else {
        audio.pause()
        audioRef.value = false
    }
}
</script>
