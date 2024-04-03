<script>
	import "./style/main.css";
	import InitDialog from "./components/InitDialog.svelte";
	import Entry from "./components/Entry.svelte";
	import CategoryToggle from "./components/CategoryToggle.svelte";
	import Logo from "./style/logo.svelte";
	import ZoomFit from "./style/zoom-fit.svelte";
	import Hidden from "./style/hidden.svelte";
	import Trash from "./style/trash.svelte";
	import Epoch from "./components/Epoch.svelte";

	const urlParams = new URLSearchParams(window.location.search);

	let init = urlParams.has("init")
		? JSON.parse(urlParams.get("init"))
		: false;
	let start = -2000
	let end = 2000;
	let disabled_categories = urlParams.has("disabled_categories")
		? JSON.parse(urlParams.get("disabled_categories"))
		: {
				people: false,
				events: false,
				works: false,
			};

	let epochs = [
		{
			start: "-3000",
			end: "-1200",
			name: "Bronze Age",
			eras: [
				{
					start: "-3000",
					end: "-1900",
					name: "Early Bronze Age",
				},
				{
					start: "-1900",
					end: "-1200",
					name: "Late Bronze Age",
				},
			],
		},
		{
			start: "-1200",
			end: "-500",
			name: "Iron Age",
			eras: [
				{
					start: "-1200",
					end: "-800",
					name: "Greek Dark Ages",
				},
				{
					start: "-800",
					end: "-500",
					name: "Archaic Period",
				},
			],
		},
		{
			start: "-500",
			end: "500",
			name: "Antiquity",
			eras: [
				{
					start: "-500",
					end: "-323",
					name: "Classical Antiquity",
				},
				{
					start: "-323",
					end: "-31",
					name: "Hellenistic Period",
				},
				{
					start: "-31",
					end: "476",
					name: "Roman Empire",
				},
			],
		},
		{
			start: "500",
			end: "1500",
			name: "Middle Ages",
			eras: [
				{
					start: "500",
					end: "1000",
					name: "Early Middle Ages",
				},
				{
					start: "1000",
					end: "1300",
					name: "High Middle Ages",
				},
				{
					start: "1300",
					end: "1500",
					name: "Late Middle Ages",
				},
			],
		},
		{
			start: "1500",
			end: "1800",
			name: "Renaissance",
			eras: [
				{
					start: "1500",
					end: "1600",
					name: "High Renaissance",
				},
				{
					start: "1600",
					end: "1700",
					name: "Baroque Period",
				},
				{
					start: "1700",
					end: "1800",
					name: "Neoclassicism",
				},
			],
		},
		{
			start: "1800",
			end: "1945",
			name: "Modernity",
			eras: [
				{
					start: "1800",
					end: "1850",
					name: "Romanticism",
				},
				{
					start: "1850",
					end: "1914",
					name: "Realism and Naturalism",
				},
				{
					start: "1914",
					end: "1945",
					name: "Modernism",
				},
			],
		},
		{
			start: "1945",
			end: "2100",
			name: "Contemporary",
			eras: [
				{
					start: "1945",
					end: "1980",
					name: "Cold War Period",
				},
				{
					start: "1980",
					end: "present",
					name: "Contemporary Era",
				},
			],
		},
	];

	let init_display = false;
	let entries = [];
	let simplified_entries =
		init && urlParams.has("entries")
			? JSON.parse(urlParams.get("entries"))
			: [];
	if (simplified_entries.length > 0) {
		for (const entry of simplified_entries) {
			entries.push({
				name: entry[0],
				birth: entry[1],
				death: entry[2],
				type: entry[3],
				ongoing: (entry[2] == null),
			});
		}
	}
	let hidden_entries = [];
	let hidden_modal_visible = "";

	function toggle_hidden_modal() {
		hidden_modal_visible = hidden_modal_visible == "" ? "visible" : "";
		hidden_entries = [...hidden_entries];
	}

	let min_start = -3000;
	let max_end = 3000;

	let max_start = end - 100;
	let min_end = start + 100;
	$: max_start = end - 100;
	$: min_end = start + 100;

	let width = end - start;
	$: width = end - start;

	let display_zero = "block";
	$: display_zero = start < 0 && end > 0 ? "block" : "none";

	let cutoff = 30;

	//import-export-functionality thru url params
	function saveState(variable, name) {
		urlParams.set(name, JSON.stringify(variable));
		window.history.replaceState(
			{},
			"",
			`${location.pathname}?${urlParams}`,
		);
	}
	$: saveState(init, "init");
	$: saveState(disabled_categories, "disabled_categories");
	
	function sort_entries() {
		entries.sort((a, b) => a.birth - b.birth); // b - a for reverse sort
		hidden_entries.sort((a, b) => a.birth - b.birth);
	}

	function render_entries() {
		// this function is called whenever the entries array is updated
		// re-calculating the bounds and width of the timeline
		start = Math.floor(entries[0].birth / 100) * 100; // round down to nearest 100
		end = start + 200;
		for (const entry of entries) {
			// we need to do this because some entries don't have a death date (ongoing events, etc.)
			if (entry.ongoing) {
				end = Math.ceil(new Date().getFullYear() / 100) * 100;
				// if we have an ongoing entry, the end of the timeline should be the
				// current year (rounded up to the next 100)
			}
			if (entry.death > end) {
				end = Math.ceil(entry.death / 100) * 100;
				// otherwise, just the latest death of all entries rounded up to nearest 100
			}
		}
		min_start = start - 500;
		max_end = end + 500;
		width = end - start;
		max_start = end - 100;
		min_end = start + 100;
	}

	$: if (entries[0] && !init_display) {

		if(simplified_entries.length == 0) {
		for (const entry of entries) {
			simplified_entries.push([
				entry.name,
				entry.birth,
				entry.death,
				entry.type,
			]);
		}
		if (!disabled_categories["works"]) {
			toggle_category("works");
		} // i dont want all works to be displayed by default
	}
		saveState(simplified_entries, "entries");
		sort_entries();
		render_entries();
		init_display = true;
	}

	const hide_entry = (idx) => {
		hidden_entries.push(entries[idx]);
		entries = entries.filter((e) => e !== entries[idx]);
		check_empty_timeline();
		sort_entries();
	};

	const show_hidden_entry = (idx) => {
		if (disabled_categories[hidden_entries[idx].type]) {
			disabled_categories[hidden_entries[idx].type] = false;
		}
		entries = [...entries, hidden_entries[idx]];
		hidden_entries = hidden_entries.filter(
			(e) => e !== hidden_entries[idx],
		);
		sort_entries();
		render_entries();
	};

	const toggle_category = (category) => {
		disabled_categories[category] = !disabled_categories[category];

		for (var entry of entries) {
			if (entry.type == category && disabled_categories[category]) {
				hidden_entries = [...hidden_entries, entry]; //hide affected elements
				entries = entries.filter((e) => e !== entry);
			}
		}

		for (var hidden_entry of hidden_entries) {
			if (
				hidden_entry.type == category &&
				!disabled_categories[category]
			) {
				hidden_entries = hidden_entries.filter(
					(e) => e !== hidden_entry,
				);
				entries = [...entries, hidden_entry]; //re-show affected, currently hidden, entries
			}
		}

		check_empty_timeline();
		sort_entries();
		// render_entries();
	};

	function check_empty_timeline() {
		if (entries.length == 0) {
			start = -2000;
			end = 2000;
			width = end - start;
		}
	}

	function parseScroll(evt) {
		if (
			evt.target.className.includes("modal") ||
			evt.target.className.includes("markdown")
		) {
			return;
		}
		if (!evt.shiftKey) {
			//zooming
			if (evt.wheelDelta > 0) {
				start += 100;
				end -= 100;
			} else {
				start -= 100;
				end += 100;
			}
		} else {
			//panning
			if (evt.wheelDelta < 0 && end < max_end - 101) {
				start += 100;
				end += 100;
			} else if (start > min_start + 100) {
				start -= 100;
				end -= 100;
			}
		}
	}
</script>

<main>
	{#if !init}
		<InitDialog bind:entries bind:init></InitDialog>
	{:else}
		<div id="main-logo">
			<Logo width="10rem" />
		</div>
		<div
			class="timeline"
			on:wheel={(evt) => {
				parseScroll(evt);
			}}
		>
			{#each epochs as epoch}
				<Epoch {epoch} {start} {end} {width} />
			{/each}
			<span id="timeline-start">{start}</span>
			<span id="timeline-end">{end}</span>
			<span
				id="timeline-zero"
				style="
			left:{Math.abs((start / width) * 100)}vw;
			display: {display_zero};
		">0</span
			>
			<div class="timeline-entries">
				{#each entries as entry}
					<Entry
						bind:start
						bind:end
						bind:width
						name={entry.name}
						birth={entry.birth}
						death={entry.death}
						idx={entries.indexOf(entry)}
						type={entry.type}
						bind:disabled_categories
						bind:misc={entry.misc}
						fullname={entry.fullname}
						ongoing={entry.ongoing}
						{hide_entry}
						{cutoff}
						content={entry.content}
					/>
				{/each}
			</div>
		</div>
		<!-- svelte-ignore a11y-click-events-have-key-events -->
		<div
			on:click={() => {
				toggle_hidden_modal();
			}}
			class="hidden-modal {hidden_modal_visible}"
		>
			<div
				on:click={(evt) => {
					evt.stopPropagation();
				}}
				class="hidden-modal-content"
			>
				<span on:click={() => toggle_hidden_modal()} class="close"
					>&times;</span
				>
				<h3>Hidden Entries</h3>
				<p>Left-click to un-hide</p>
				{#each hidden_entries as entry}
					<span
						on:click={() =>
							show_hidden_entry(hidden_entries.indexOf(entry))}
						class="hidden-entry entry-{entry.type}"
						>{entry.name}</span
					><br />
				{/each}
			</div>
		</div>
	{/if}
</main>
{#if init}
	<footer id="settings">
		<!-- svelte-ignore a11y-click-events-have-key-events -->
		<span
			class="settings-group"
			id="settings-zoom-fit"
			on:click={() => {
				render_entries();
			}}
		>
			<ZoomFit width="2.4rem" id="reset-timeline" />
		</span>
		<span class="settings-group" id="settings-timespan">
			<input
				id="start-slider"
				class="limit-slider"
				type="range"
				min={min_start}
				max={max_start}
				step="100"
				bind:value={start}
			/>
			<input
				id="end-slider"
				class="limit-slider"
				type="range"
				min={min_end}
				max={max_end}
				step="100"
				bind:value={end}
			/>
		</span>
		<span class="settings-group" id="settings-category-toggles">
			<CategoryToggle
				handle_toggle={toggle_category}
				category="people"
				bind:disabled_categories
			/>
			<CategoryToggle
				handle_toggle={toggle_category}
				category="events"
				bind:disabled_categories
			/>
			<CategoryToggle
				handle_toggle={toggle_category}
				category="works"
				bind:disabled_categories
			/>
		</span>
		<span class="settings-group" id="settings-cutoff">
			<input
				id="cutoff-slider"
				type="range"
				min="1"
				max={entries[0] ? entries.length + 1 : 60}
				step="1"
				bind:value={cutoff}
			/>
			<label for="cutoff-slider">Vertical Spacing: </label>
		</span>
		<span class="settings-group" id="settings-info">
			<p>
				Double-click to zoom in on an entry. Right-click to hide.
				Left-click to view details. Scroll to zoom, shift + scroll to
				pan.
			</p>
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<span
				id="settings-show-hidden"
				on:click={() => {
					toggle_hidden_modal();
				}}><Hidden width="2.4rem" /></span
			>
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<span
				id="settings-reset"
				on:click={() => {
					if (confirm("Reset the timeline, deleting all entries?")) {
						init = false;
						init_display = false;
						entries = [];
						hidden_entries = [];
					}
				}}><Trash width="2.4rem" /></span
			>
		</span>
	</footer>
{/if}

<svelte:window
	on:keydown={(evt) => {
		if (evt.key == "Escape" && hidden_modal_visible) toggle_hidden_modal();
	}}
/>
