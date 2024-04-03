<script>
    export let raw_content;
    export let max_width;

    let tag_lines = [];
    let lines = [];

    $: if (raw_content) {

        tag_lines = [];
        lines = [];

        for (const line of raw_content.split('\n')) {
        if (line.includes('::')) {
            tag_lines.push(
                `<p class='markdown-line markdown-line-tags'>
                    <span class='markdown-line-tags__tag'>${line.split('::')[0]}</span>
                    <span class='markdown-line-tags__content'>${line.split('::')[1].replace('[[', '').replace(']]', '').replace('#', '')}</span>
                </p>`
            )
        } else if (line != '') {
            lines.push(
                `<li class='markdown-line'>${line.replace('[[', '').replace(']]', '').replace('#', '').replace('-', '').trim()}</li>`
            )
        }
        }
    }
</script>

<div class='markdown-container' style='max-width: {max_width}px;'>
    {#if raw_content != '' && raw_content}
        {#each tag_lines as line}
            {@html line}
        {/each}
        <ul>
        {#each lines as line}
            {@html line}
        {/each}
        </ul>
    {:else}
        <p style="color:var(--dark);">No content. This probably means Chrono has loaded its data from the URL parameters.
            Due to size constraints, the content of your entries can only be shown when 
            Chrono is initialized from your local files. To do that, reset the timeline using
            the red trash can.
        </p>
    {/if}
</div>
