<script>
    export let start;
    export let width;
    export let era;
    export let noborder;

    let left;
    $: left = (era.start - start) / width * 100;
    $: if (left < 0 && start < era.end) left = 0;
    let display;
    $: display = start > era.end || (width) > 3000 ? 'none' : 'block';
</script>

<div class='era {noborder ? "noborder" : ""}' style="display:{display};left:{left}vw;" id='era-{era.name}'>
    <span class='era-name'>{era.name}</span>
</div>