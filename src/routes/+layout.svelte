<style>
    nav {
        display: flex;
        border-bottom-width: 1px;
        border-bottom-style: solid;
        border-bottom-color: var(--colour-highlight);
    }

    nav a {
        flex: 1;
        text-decoration: none;
        color: inherit;
        text-align: center;
        padding: 0.5em;
    }

    nav a.current {
        border-bottom-width: 0.4em;
        border-bottom-style: solid;
        border-bottom-color: var(--colour-highlight);
    }

    nav a:hover {
        border-bottom-width: 0.4em;
        border-bottom-style: solid;
        border-bottom-color: var(--colour-accent);
        background-color: var(--colour-highlight-30);
    }

    label.color-scheme {
        position: absolute;
        top: 1rem;
        right: 1rem;
        font-size: 80%;
        font-family: inherit;
    }
</style>

<script>
    import { page } from '$app/stores';
    let pages = [
        {url: "./", title: "Home"},
        {url: "./projects", title: "Projects"},
        {url: "./meta", title: "Meta"},
        {url: "https://github.com/Gingertonic", title: "Github"},
    ];

    let localStorage = globalThis.localStorage ?? {};
    let colorScheme = localStorage.colorScheme || "light dark";
    let root = globalThis?.document?.documentElement;
    $: root?.style.setProperty("color-scheme", colorScheme);
    $: localStorage.colorScheme = colorScheme;
</script>

<label class="color-scheme">
    Theme:
    <select bind:value={ colorScheme }>
        <option value="light dark">Automatic</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
    </select>
</label>

<nav>
	{#each pages as p }
         <a 
            href={ p.url } 
            class:current={ "." + $page.route.id === p.url }
            target={ p.url.startsWith("http") ? "_blank" : null }>{ p.title }</a>
	{/each}
</nav>

<slot />