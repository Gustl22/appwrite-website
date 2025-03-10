<script lang="ts" context="module">
    export type Theme = 'dark' | 'light' | 'system';
    export const currentTheme = (function () {
        const store = writable<Theme>(getPreferredTheme());

        const set: typeof store.set = (value) => {
            store.set(value);
            if (browser) {
                localStorage.setItem('theme', value);
            }
        };

        return { ...store, set };
    })();

    export const themeInUse = derived(currentTheme, (theme) => {
        return theme === 'system' ? getSystemTheme() : theme;
    });

    function isTheme(theme: unknown): theme is Theme {
        return ['dark', 'light', 'system'].includes(theme as Theme);
    }

    function getSystemTheme(): Theme {
        return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
    }

    function getPreferredTheme() {
        if (!browser) {
            return 'dark';
        }

        const theme = localStorage.getItem('theme');

        if (!isTheme(theme)) {
            return 'dark';
        }

        return theme;
    }
</script>

<script lang="ts">
    import '$icons/output/aw-icon.css';
    import '$scss/index.scss';

    import { browser, dev } from '$app/environment';
    import { navigating, page } from '$app/stores';
    import { onMount } from 'svelte';
    import { derived, writable } from 'svelte/store';

    function applyTheme(theme: Theme) {
        const resolvedTheme = theme === 'system' ? getSystemTheme() : theme;
        const className = `theme-${resolvedTheme}`;
        document.body.classList.remove('theme-dark', 'theme-light');
        document.body.classList.add(className);
    }

    onMount(() => {
        const initialTheme = $page.route.id?.startsWith('/docs') ? getPreferredTheme() : 'dark';

        applyTheme(initialTheme);

        navigating.subscribe((n) => {
            if (!n?.to) {
                return;
            }

            const isDocs = n.to.route.id?.startsWith('/docs');

            if (isDocs) {
                if (!document.body.classList.contains(`theme-${$currentTheme}`)) {
                    applyTheme($currentTheme);
                }
            } else {
                applyTheme('dark');
            }
        });
    });

    $: if (browser) currentTheme.subscribe((theme) => applyTheme(theme));
</script>

<svelte:head>
    {#if !dev}
        <script defer data-domain="appwrite.io" src="https://plausible.io/js/script.js"></script>
    {/if}
</svelte:head>

<a class="skip" href="#main">Skip to content</a>

<slot />

<style lang="scss">
    .skip {
        position: absolute;
        inset-block-start: 0;
        z-index: 9999;

        display: block;
        background-color: hsl(var(--aw-color-mint-500));
        color: hsl(var(--aw-color-black));
        text-decoration: underline;
        opacity: 0;

        padding: 0.75rem 1.25rem;
        pointer-events: none;
    }

    .skip:focus {
        opacity: 1;
        position: relative;
        pointer-events: all;
    }
</style>
