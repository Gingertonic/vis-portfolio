<script lang="ts">
    import type { StatItem } from "../types";
    export let data: StatItem[] = [{title: '', value: ''}];
</script>


<dl class="stats">
    {#each data as stat}
        <dt>{stat.title}</dt>
        <dd>{stat.value}</dd>
    {/each}
</dl>