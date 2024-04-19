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

	let cutoff = 20;

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
	<a class='btn white' id="backlink" href="https://martcho.uk/"><svg version="1.1" id="Layer_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
		width="100%" viewBox="0 0 300 300" enable-background="new 0 0 300 300" xml:space="preserve">
   <path opacity="1.000000" stroke="none" 
	   d="
   M174.977158,26.954788 
	   C168.516998,41.736927 164.746735,56.959564 161.089737,72.184875 
	   C158.720444,82.048973 156.690369,92.022240 155.120010,102.042450 
	   C153.658875,111.365753 153.033081,120.818848 151.980911,130.208237 
	   C151.737030,132.384781 151.254974,134.534637 150.838501,136.952240 
	   C151.856995,136.821198 152.616348,136.926697 152.763077,136.675049 
	   C156.929230,129.528580 163.661896,129.518097 170.702667,129.958511 
	   C176.219315,130.303604 183.164139,137.373627 182.999969,142.833862 
	   C182.843903,148.024918 177.632278,151.401474 173.158646,148.909470 
	   C169.688431,146.976440 166.430359,143.149796 162.362183,146.417160 
	   C159.277100,148.894958 157.194656,152.622345 154.675262,155.800949 
	   C154.472000,156.057388 154.340042,156.370377 153.445114,157.924500 
	   C158.155212,158.671448 162.412384,159.055267 166.519196,160.056091 
	   C176.262466,162.430511 185.659424,160.088348 194.911118,157.814957 
	   C199.552582,156.674438 204.294861,154.283905 207.962646,151.224991 
	   C213.663162,146.470779 225.307800,146.904007 228.612411,154.208847 
	   C232.959061,163.817123 240.970276,164.898148 249.846878,163.878952 
	   C252.215942,163.606934 254.413223,161.857285 256.694427,160.796921 
	   C259.300598,159.585510 261.822540,158.100693 264.541748,157.245483 
	   C269.164673,155.791580 273.498322,158.174957 274.977875,162.892380 
	   C276.169281,166.691025 277.026337,170.595810 277.994904,174.462738 
	   C279.235748,179.416809 275.444519,181.155396 272.131042,181.759430 
	   C268.677368,182.389023 265.564880,180.325287 264.273376,176.633530 
	   C263.907074,175.586487 263.933624,174.402008 263.725250,172.881027 
	   C261.139465,173.627060 258.782135,174.630875 256.335510,174.950256 
	   C249.886139,175.792175 243.398071,176.853394 236.925217,176.854675 
	   C230.453400,176.855957 225.084274,172.668762 221.234970,168.280899 
	   C216.847031,163.279022 213.331650,165.102188 208.715836,166.650864 
	   C202.374313,168.778488 195.881241,170.602737 189.321457,171.882080 
	   C184.523193,172.817871 179.496658,173.092148 174.600784,172.905060 
	   C166.260773,172.586334 157.936462,171.747910 149.620865,170.965195 
	   C147.049759,170.723206 145.503891,171.607178 145.045090,174.040741 
	   C142.656525,186.710419 140.683868,199.469849 137.897583,212.049896 
	   C135.333313,223.627563 132.070129,235.059784 128.827194,246.474350 
	   C126.829887,253.504578 124.546257,260.479553 121.908333,267.292358 
	   C120.190300,271.729431 118.092712,276.190948 115.285889,279.992249 
	   C111.667389,284.892792 104.609657,283.943176 100.715790,278.175049 
	   C92.859673,266.537445 83.917046,255.809570 73.638657,246.309555 
	   C67.693275,240.814423 61.351070,235.493210 54.417084,231.398209 
	   C48.059143,227.643402 40.391876,227.755081 32.969078,227.862885 
	   C30.651054,227.896545 28.022282,226.726440 26.041323,225.369370 
	   C22.449326,222.908661 23.053736,217.513306 27.156342,216.540176 
	   C31.311600,215.554565 35.764343,215.646805 40.089584,215.611740 
	   C43.358765,215.585251 46.789234,215.566986 49.875553,216.472931 
	   C60.613075,219.624771 70.049561,225.183609 77.790367,233.333206 
	   C87.090042,243.123993 96.207596,253.087769 105.507141,262.878693 
	   C106.322365,263.737000 107.947540,264.300751 109.110405,264.178986 
	   C109.825745,264.104034 110.627815,262.547760 110.963631,261.517365 
	   C114.135628,251.784897 117.603951,242.125015 120.214912,232.240295 
	   C122.922882,221.988281 124.727280,211.499634 127.018585,201.134766 
	   C128.439316,194.707947 130.268066,188.363693 131.488480,181.902069 
	   C133.199646,172.842163 134.599152,163.720200 135.973892,154.600952 
	   C137.112656,147.047058 137.967926,139.450851 139.048325,131.887604 
	   C140.309113,123.061554 141.621796,114.242012 143.020416,105.436874 
	   C143.906952,99.855591 144.969147,94.301651 145.992386,88.742767 
	   C147.307343,81.599121 148.536148,74.436302 150.031647,67.330055 
	   C151.914566,58.382797 153.900528,49.452515 156.092224,40.576691 
	   C157.429626,35.160603 159.364700,29.894279 160.790283,24.497019 
	   C161.764450,20.808825 163.278946,18.619026 167.689514,18.886003 
	   C171.778854,19.133533 173.459503,21.260880 174.326385,24.747677 
	   C174.480042,25.365690 174.768524,25.950180 174.977158,26.954788 
   z"/>
	 </svg></a>
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
			<span class='timeline-year' id="timeline-start">{start}</span>
			<span class='timeline-year' id="timeline-end">{end}</span>
			<span
				class='timeline-year' id="timeline-zero"
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
			class=" hidden-modal {hidden_modal_visible}"
		>
			<div
				on:click={(evt) => {
					evt.stopPropagation();
				}}
				class="hidden-modal-content container-m bg border pop-m"
			>
				<span on:click={() => toggle_hidden_modal()} class="title entry-modal-close close btn red c-white"
					>&times;</span
				>
				<h3 class='title c-text'>Hidden Entries</h3>
				<p class='subtitle secondary space-v-l space-v-first'>Left-click to un-hide</p>
				{#each hidden_entries as entry}
					<div class='border pad-h-l pad-v-m hidden-entry hidden-entry-{entry.type}'
						on:click={() =>
							show_hidden_entry(hidden_entries.indexOf(entry))}
						>{entry.name}</div
					><br />
				{/each}
			</div>
		</div>
	{/if}
</main>
{#if init}
	<div id='settings-container'>
	<footer id="settings" class="container-s border pop-m pad-v-m space-v-l space-h-l yellow">
		<!-- svelte-ignore a11y-click-events-have-key-events -->
		<span
			class="settings-group btn blue space-h-l space-h-first c-white"
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
		<span class="settings-group space-h-l" id="settings-category-toggles">
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
			<label for="cutoff-slider" class='subtitle'>Vertical Spacing: </label>
			<input
				id="cutoff-slider"
				type="range"
				min="1"
				max={entries[0] ? entries.length + 1 : 60}
				step="1"
				bind:value={cutoff}
			/>
		</span>
		<span class="settings-group" id="settings-info">
			<p class='subtitle space-h-m space-h-first'>
				Double-click to zoom in on an entry. Right-click to hide.
				Left-click to view details. Scroll to zoom, shift + scroll to
				pan.
			</p>
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<span
				id="settings-show-hidden" class='btn space-h-m c-white blue'
				on:click={() => {
					toggle_hidden_modal();
				}}><Hidden width="2.4rem" /></span
			>
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<span
				id="settings-reset"
				class='btn red c-white'
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
	</div>
{/if}

<svelte:window
	on:keydown={(evt) => {
		if (evt.key == "Escape" && hidden_modal_visible) toggle_hidden_modal();
	}}
/>
