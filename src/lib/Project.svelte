<script>
    export let data = { title: '', image: '', description: '' };
    export let hLevel = '2';
</script>

<style>
    article {
        display: grid;
        grid-template-rows: subgrid;
        grid-row: span 3;
    }

    article h2, article h3 {
        margin: 0;
    }
</style>

<article>
    <svelte:element this={"h" + hLevel}>{ data.title }</svelte:element>
    <img src={data.image} alt={`Preview of ${data.title}`}>
    <p>{data.description}</p>
</article>