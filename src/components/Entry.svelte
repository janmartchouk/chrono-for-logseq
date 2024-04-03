<script>
    import Markdown from "./Markdown.svelte";

    export let start;
    export let end;
    export let width;
    export let name;
    export let birth;
    export let death;
    export let idx;
    export let type;
    export let disabled_categories;
    export let misc;
    export let fullname;
    export let ongoing;
    export let hide_entry;
    export let cutoff;
    export let content;

    function handle_rightclick(event) {
    // Prevent the default context menu from showing up
    event.preventDefault();
    hide_entry(idx);
  }

    let click_is_double = false;

    let modal_visible = '';

    let animation = 'no-animation';
    setTimeout(() => {
        animation = '';
    }, 500);

    function handle_click() {
        setTimeout(() => {
            if (!click_is_double) {
                modal_visible = 'visible';
                console.log(content);
            }
        }, 200);
    }

    function close_modal() {
        modal_visible = '';
    }
 
    function handle_doubleclick() {
        click_is_double = true;
        if (start >= birth-200 && ( end <= birth+250 || end <= death+200) ) {
            start = birth - 50;
            end = ongoing ? birth + 150 : death + 50;
            console.log('second zoom')
        } 
        if( start < birth-200 || end > birth+250) {
            start = birth - 200;
            end = ongoing ? birth + 250 : death + 200;
            console.log('first zoom')
        } 
        setTimeout(() => {
            click_is_double = false;
        }, 200);
    }

    let left;
    $: left = Math.abs(((start - birth) / width) * 100);

    let display = "block";
    $: if (
        birth < start ||
        death > end ||
        disabled_categories[type] ||
        (ongoing && end < new Date().getFullYear())
    ) {
        display = "none";
        animation = 'no-animation';
    } else {
        display = "block";
        setTimeout(() => {
            animation = '';
        }, 500);
    }

    let entry_width;
    $: entry_width =
        type == "people" || type == "events"
            ? (Math.abs(birth - death) / width) * 100
            : "auto";
    $: if ((type == "people" && !ongoing) || type == "events") {
        entry_width = (Math.abs(birth - death) / width) * 100;
    } else if (ongoing) {
        entry_width =
            (Math.abs(birth - new Date().getFullYear()) / width) * 100;
    } else {
        entry_width = "auto";
    }

    let tooltip;
    let offset = '';
 
    function ensureTooltipVisibility() {
        if (tooltip.getBoundingClientRect().left < 0) {
            offset = 'entry-tooltip-offScreenLeft';
        }
        else if (tooltip.getBoundingClientRect().right > (window.innerWidth -50)) {
            offset = 'entry-tooltip-offScreenRight';
        } else {
            offset = '';
        }
    }
    
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div
    class="entry"
    style="
	left: {left}vw;
	width: {entry_width}vw;
	transform: translateY(-{(idx % cutoff) * 3.5}rem);
    display: {display};
	"
    on:click={() => handle_click()}
    on:dblclick={() => handle_doubleclick()}
    on:mouseenter={() => ensureTooltipVisibility()}
    on:contextmenu={handle_rightclick}
>
    <p class="entry-name entry-{type} {ongoing ? 'entry-ongoing' : ''}">
        {name}
    </p>
    <span
        bind:this={tooltip}
        class="entry-tooltip {offset} {animation}">
        <p class="entry-tooltip-name">{name}</p>
        <p class="entry-tooltip-fullname">{fullname && fullname != name ? fullname : ""}</p>
        <p class="entry-dates">
            {ongoing && type == "people" ? "*" : ""}{birth >= 0
                ? birth
                : birth * -1 + " BC"}{(type == "people" && !ongoing) ||
            (type == "events" && !ongoing)
                ? " – " + (death >= 0 ? death : death * -1 + " BC")
                : ""}
        </p>
        <p>{misc ? misc : ''}</p>
    </span>
</div>
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div on:wheel={(evt)=>{evt.stopPropagation()}} on:click={() => close_modal()} class="{'entry-modal ' + modal_visible} {animation} entry-modal-{type}">
    <div on:click={(evt) => {evt.stopPropagation()}} class='entry-modal-content {animation}'>
    <span on:click={() => close_modal()} class='entry-modal-close close'>&times;</span>
    <h3 class='entry-modal-title'>{name}</h3>
    <Markdown raw_content={content} max_width='50vw' />
    </div>
</div>

<svelte:window on:keydown={(evt) => {if (evt.key == 'Escape') close_modal()}}/>