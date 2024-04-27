<script>
    import Era from './Era.svelte';
    export let start;
    export let end;
    export let epoch = {};
    export let width;

    let left;
    $: left = (epoch.start - start) / width * 100;
    $: if (left < 0 && start < epoch.end) left = 0;
    let display;
    $: display = start > epoch.end ? 'none' : 'block';
</script>

<div class='epoch' style="display:{display};left:{left}vw;" id='epoch-{epoch.name}'>
    <span class='epoch-name title secondary'>{epoch.name} ({epoch.start} – {epoch.end})</span>
</div>
{#each epoch.eras as era} 
<Era start={start} end={end} era={era} width={width} noborder={era.start == epoch.start ? true : false}/>
{/each}
