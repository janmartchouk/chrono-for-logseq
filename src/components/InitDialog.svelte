<script>
    export let entries;
    export let init;
    import Logo from '../style/logo.svelte';
    let files;
    let tmp_entries = [];

    function processYearLine(line) {
        let y = line.split('::')[1].replace('#', '').replace('[[', '').replace(']]', '').replace('ca. ', '');
        y= y.replace(/^[^\d]*/, ''); //remove everything leading up to the first digit; parseInt works with that
        if (y.includes('Chr') || y.includes('BC')) {
            y = -1 *  parseInt(y);
        }
        return parseInt(y);
    }

    let german_property_names = {
        people: {
            birth: 'geboren',
            death: 'gestorben',
        },
        events: {
            birth: 'beginn',
            death: 'ende',
        },
        works: {
            birth: 'datum',
            author: 'autor',
        }
    }

    let english_property_names = {
        people: {
            birth: 'born',
            death: 'died',
        },
        events: {
            birth: 'start',
            death: 'end',
        },
        works: {
            birth: 'date',
            auhor: 'author',
        }
    }

    let russian_property_names = {
        people: {
            birth: 'родился',
            death: 'умер',
        },
        events: {
            birth: 'начало',
            death: 'конец',
        },
        works: {
            birth: 'дата',
            author: 'автор',
        }
    }

    let property_names=german_property_names;

    $ : if (files && !init) {
		// Note that `files` is of type `FileList`, not an Array:
		// https://developer.mozilla.org/en-US/docs/Web/API/FileList
		for (const file of files) {
            let reader = new FileReader()
            reader.onload = function(e) {
                let result = e.target.result;
                let lines = result.split('\n');
                var entry = {
                    name: file.name.replace('.md', ''),
                    birth: 0,
                    death: undefined,
                    type: '',
                    misc: '',
                    fullname: '',
                    ongoing: false,
                    content: ''
                }
                if (result.toLowerCase().includes(property_names.people.birth+'::')) {
                    // we are processing a person
                    entry.type = 'people';
                    for (const line of lines) {
                        if (line.toLowerCase().includes(property_names.people.birth+'::')) {
                            entry.birth = processYearLine(line);
                        }
                        if (line.toLowerCase().includes(property_names.people.death+'::')) {
                            entry.death = processYearLine(line);
                            if (!entry.death) {
                                entry.ongoing = true;
                            }
                        }
                    }
                } else if (result.toLowerCase().includes(property_names.events.birth+'::')) {
                    // we are processing an event
                    entry.type = 'events';
                    for (const line of lines) {
                        if (line.toLowerCase().includes(property_names.events.birth+'::')) {
                            entry.birth = processYearLine(line);
                        }
                        if (line.toLowerCase().includes(property_names.events.death+'::')) {
                            entry.death = processYearLine(line);
                        }
                    }
                } else if (result.toLowerCase().includes(property_names.works.author+'::')) {
                    // we are processing a work
                    entry.type = 'works';
                    for (const line of lines) {
                        if (line.toLowerCase().includes(property_names.works.author+'::')) {
                            entry.misc = line.replace(property_names.works.author+'::', '').replace('[[', '').replace(']]', '').replace('#', '').trimStart();
                        }
                        if (line.toLowerCase().includes(property_names.works.birth+'::')) {
                            entry.birth = processYearLine(line);
                            entry.death = entry.birth + 100;
                        }
                    }
                }
                for (const line of lines) {
                    // look if we have a defined full name
                    if (line.toLowerCase().includes('name::') || line.toLowerCase().includes('titel::') || line.toLowerCase().includes('title::')) {
                            entry.fullname = line.replace('name::', '').replace('titel::', '').replace('[[', '').replace(']]', '').replace('#', '').trimStart();
                    }
                }
                if (entry.death == undefined) {
                    entry.death = entry.birth + 50;
                    entry.ongoing = true;
                }
                if(entry.birth != entry.death && entry.type != '' && entry.birth) {
                    entry.content = result;
                    tmp_entries = [...tmp_entries, entry];
                }
            }
            reader.readAsText(file);
		}
        setTimeout(() => {
            entries = tmp_entries;
            init = true;
        }, (100 * tmp_entries.length + 1000));
        init = true;
	}

</script>

<div id="init">
    <Logo width='20rem'/>
    <div id="init-horizontal">
        <span id="init-horizontal-left">
            <h2 class="init-title">1. Configure your property names</h2>
            <p>Chrono will search throughout your graph to find time-able entries.</p>
            <form id='init-form-autofill'>
                <group id='autofill-group'>
                    <!-- svelte-ignore a11y-click-events-have-key-events -->
                    <span on:click={() => {property_names = german_property_names}} class="btn">🇩🇪</span>
                    <!-- svelte-ignore a11y-click-events-have-key-events -->
                    <span on:click={() => {property_names = english_property_names}} class="btn">🇬🇧</span>
                    <!-- svelte-ignore a11y-click-events-have-key-events -->
                    <span on:click={() => {property_names = russian_property_names}} class="btn">🇷🇺</span>
                </group>
            </form>
            <form id='init-form-people' class='init-form'>
                <p class='init-form-title'>People</p>
                <span class='init-form-inputs'>
                    <group>
                        <label for="people-birth">Birth:</label>
                        <input type="text" id="people-birth" bind:value={property_names.people.birth} />
                    </group>
                    <group>
                        <label for="people-death">Death:</label>
                        <input type="text" id="people-death" bind:value={property_names.people.death} />
                    </group>
                </span>
            </form>
            <form id='init-form-events' class='init-form'>
                <p class='init-form-title'>Events</p>
                <span class='init-form-inputs'>
                <group>
                <label for="events-birth">Start:</label>
                <input type="text" id="events-birth" bind:value={property_names.events.birth} />
            </group>
            <group>
                <label for="events-death">End:</label>
                <input type="text" id="events-death" bind:value={property_names.events.death} />
            </group>
        </span>

            </form>
            <form id='init-form-works' class='init-form'>
                <p class='init-form-title'>Works</p>
                <span class='init-form-inputs'>
                <group>
                <label for="works-birth">Date:</label>
                <input type="text" id="works-birth" bind:value={property_names.works.birth} />
            </group>
            <group>
                <label for="works-author">Author:</label>
                <input type="text" id="works-author" bind:value={property_names.works.author} />
            </group>
        </span>

            </form>
        </span>
        <span id='init-horizontal-right'>
            <h2 class="init-title">2. Provide your Logseq graph</h2>
            <label for="pages">Upload the 'pages' folder:</label>
            <input bind:files id="pages" directory webkitdirectory type="file" />
        </span>
    </div>
</div>