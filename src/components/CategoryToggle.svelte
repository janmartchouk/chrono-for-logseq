<script>
    export let handle_toggle;
    export let category;
    export let disabled_categories;
    let disabled_class = "";

    $: if (disabled_categories[category]) {
        disabled_class = "disabled";
    } else {
        disabled_class= "";
    }

    const onToggle = e => {
        e.preventDefault();
        handle_toggle(category);
    }
</script>

<input on:click={onToggle} id="toggle-{category}" class="category-toggle btn {disabled_class}" type="button" value={category.charAt(0).toUpperCase() + category.slice(1)}/>
