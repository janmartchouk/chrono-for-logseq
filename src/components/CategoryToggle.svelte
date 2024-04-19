<script>
    export let handle_toggle;
    export let category;
    export let disabled_categories;
    let disabled_class = "";
    let disp_color = 'secondary';

    $: if (disabled_categories[category]) {
        disabled_class = "disabled";
        disp_color = 'secondary';
    } else {
        disabled_class= "";
        disp_color = color;
    }

    const onToggle = e => {
        e.preventDefault();
        handle_toggle(category);
    }

    let color;
    switch(category) {
        case "people":
            color = "blue c-white";
            break;
        case "events":
            color = "red c-white";
            break;
        case "works":
            color = "green";
            break;
        default:
            color = "secondary";
    }
</script>

<input on:click={onToggle} id="toggle-{category}" class="space-h-s category-toggle btn {disp_color} {disabled_class}" type="button" value={category.charAt(0).toUpperCase() + category.slice(1)}/>
