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
</style>

<script>
    import { page } from '$app/stores';
    import "../style.css";
    export const prerender = true;
    let pages = [
        {url: "./", title: "Home"},
        {url: "./projects", title: "Projects"},
        {url: "https://github.com/Gingertonic", title: "Github"},
    ];
</script>

<nav>
	{#each pages as p }
         <a 
            href={ p.url } 
            class:current={ "." + $page.route.id === p.url }
            target={ p.url.startsWith("http") ? "_blank" : null }>{ p.title }</a>
	{/each}
</nav>

<slot />