<script lang="ts">
	import { tick, onMount } from "svelte";

    let span: HTMLSpanElement;
    let textarea: HTMLTextAreaElement;
    let text = 'some large text';
    let mounted = false;
    let resize = Symbol();

    onMount(() => {
        mounted = true;
    });

    function isNear(value: number, target: number, threshold = 1) {
        return Math.abs(value - target) < threshold;
    }

    const timeout = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

    async function calculateFont(alpha = 1, beta = 100_000) {
        if (text.length === 1) {
            textarea.style.fontSize = `${window.innerWidth / 2}px`;
            return window.innerWidth / 2;
        }

        span.style.fontSize = `${(alpha + beta) / 2}px`;
        textarea.style.fontSize = `${(alpha + beta) / 2 - 1}px`;
        textarea.style.height = `${
            span.getBoundingClientRect().height
            + (text.endsWith('\n') ? (span.getBoundingClientRect().height / (text.split('\n').length - 1)) : 0)
        }px`;
        await tick();
        const calculatedWidth = span.getBoundingClientRect().width;
        const wantedWidth = window.innerWidth;
        if (isNear(calculatedWidth, wantedWidth)) {
            return (alpha + beta) / 2;
        } else if (calculatedWidth > wantedWidth) {
            // we need to decrease the font size
            return calculateFont(alpha, (alpha + beta) / 2);
        } else {
            // we need to increase the font size
            return calculateFont((alpha + beta) / 2, beta);
        }
    }

    $: if (text && mounted && resize) calculateFont();
</script>

<svelte:window on:resize={() => resize = Symbol()} />

<div class="hidden" aria-hidden>
    <span bind:this={span}><pre>{text}</pre></span>
</div>

<main>
    <textarea
        tabindex="0"
        bind:value={text}
        bind:this={textarea}
        data-gramm="false"
        data-gramm_editor="false"
        data-enable-grammarly="false"
    />
</main>

<style>
    :global(body, html) {
        margin: 0;
        padding: 0;
        height: 100%;
    }

    main, .hidden {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
    }

    span {
        white-space: nowrap;
        user-select: none;
    }

    pre {
        font-family: inherit;
        user-select: none;
        margin: 0;
    }

    .hidden {
        user-select: none;
        pointer-events: none;
        opacity: 0;
        position: fixed;
    }

    textarea {
        width: 100%;
        outline: none;
        border: none;
        padding: 0;
        resize: none;
        text-align: center;
        font-family: inherit;
        vertical-align: middle;
    }
</style>
