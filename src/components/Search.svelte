<script lang="ts">
import I18nKey from "@i18n/i18nKey";
import { i18n } from "@i18n/translation";
import Icon from "@iconify/svelte";
import { url } from "@utils/url-utils.ts";
import { onMount } from "svelte";
import type { SearchResult } from "@/global";

let keywordDesktop = "";
let keywordMobile = "";
let result: SearchResult[] = [];
let isSearching = false;
let pagefindLoaded = false;
let initialized = false;
let hasSearched = false;
let debounceTimer: ReturnType<typeof setTimeout> | undefined;

const fakeResult: SearchResult[] = [
	{
		url: url("/"),
		meta: {
			title: "Lab: Blind SQLi Practice Notes",
			description: "A short walkthrough of payload testing and validation in a local lab.",
			category: "Labs",
			tags: "sqli, web, recon",
		},
		excerpt: "Search uses <mark>mock data</mark> in development mode.",
	},
	{
		url: url("/"),
		meta: {
			title: "Theory: Recon Prioritization For Bug Bounty",
			description:
				"A framework for quickly choosing high-signal targets and avoiding noisy scans.",
			category: "Theory",
			tags: "methodology, bug-bounty",
		},
		excerpt: "Run <mark>pnpm build</mark> and <mark>pnpm preview</mark> to test real Pagefind search.",
	},
];

const togglePanel = () => {
	const panel = document.getElementById("search-panel");
	panel?.classList.toggle("float-panel-closed");
};

const setPanelVisibility = (show: boolean, isDesktop: boolean): void => {
	const panel = document.getElementById("search-panel");
	if (!panel || !isDesktop) return;

	if (show) {
		panel.classList.remove("float-panel-closed");
	} else {
		panel.classList.add("float-panel-closed");
	}
};

const queueSearch = (keyword: string, isDesktop: boolean): void => {
	if (debounceTimer) {
		clearTimeout(debounceTimer);
	}
	debounceTimer = setTimeout(() => {
		void search(keyword, isDesktop);
	}, 170);
};

const parseTags = (tags?: string): string[] => {
	if (!tags) return [];
	return tags
		.split(",")
		.map((tag) => tag.trim())
		.filter(Boolean)
		.slice(0, 3);
};

const search = async (keyword: string, isDesktop: boolean): Promise<void> => {
	const query = keyword.trim();
	hasSearched = query.length > 0;

	if (!query) {
		setPanelVisibility(false, isDesktop);
		result = [];
		return;
	}

	if (!initialized) {
		return;
	}

	isSearching = true;

	try {
		let searchResults: SearchResult[] = [];

		if (import.meta.env.PROD && pagefindLoaded && window.pagefind) {
			const response = await window.pagefind.search(query);
			searchResults = await Promise.all(
				response.results.map((item) => item.data()),
			);
		} else if (import.meta.env.DEV) {
			searchResults = fakeResult;
		} else {
			searchResults = [];
		}

		result = searchResults;
		setPanelVisibility(true, isDesktop);
	} catch (error) {
		console.error("Search error:", error);
		result = [];
		setPanelVisibility(true, isDesktop);
	} finally {
		isSearching = false;
	}
};

onMount(() => {
	const initializeSearch = () => {
		initialized = true;
		pagefindLoaded =
			typeof window !== "undefined" &&
			!!window.pagefind &&
			typeof window.pagefind.search === "function";
		if (keywordDesktop) queueSearch(keywordDesktop, true);
		if (keywordMobile) queueSearch(keywordMobile, false);
	};

	const handlePagefindReady = () => {
		initializeSearch();
	};

	const handlePagefindError = () => {
		initializeSearch();
	};

	const handleShortcut = (event: KeyboardEvent) => {
		if (
			event.key === "/" &&
			!(event.target instanceof HTMLInputElement) &&
			!(event.target instanceof HTMLTextAreaElement) &&
			!(event.target instanceof HTMLElement && event.target.isContentEditable)
		) {
			event.preventDefault();
			const input = document.getElementById("search-input-desktop") as HTMLInputElement | null;
			input?.focus();
			setPanelVisibility(true, true);
		}

		if (event.key === "Escape") {
			setPanelVisibility(false, true);
		}
	};

	if (import.meta.env.DEV) {
		initializeSearch();
	} else {
		document.addEventListener("pagefindready", handlePagefindReady);
		document.addEventListener("pagefindloaderror", handlePagefindError);

		setTimeout(() => {
			if (!initialized) {
				initializeSearch();
			}
		}, 2000);
	}

	document.addEventListener("keydown", handleShortcut);

	return () => {
		document.removeEventListener("pagefindready", handlePagefindReady);
		document.removeEventListener("pagefindloaderror", handlePagefindError);
		document.removeEventListener("keydown", handleShortcut);
		if (debounceTimer) {
			clearTimeout(debounceTimer);
		}
	};
});


$: if (initialized) {
	queueSearch(keywordDesktop, true);
}


$: if (initialized) {
	queueSearch(keywordMobile, false);
}
</script>

<!-- search bar for desktop view -->
<div id="search-bar" class="hidden lg:flex transition-all items-center h-11 mr-2 rounded-lg
      bg-black/[0.04] hover:bg-black/[0.06] focus-within:bg-black/[0.06]
      dark:bg-white/5 dark:hover:bg-white/10 dark:focus-within:bg-white/10
">
    <Icon icon="material-symbols:search" class="absolute text-[1.25rem] pointer-events-none ml-3 transition my-auto text-black/30 dark:text-white/30"></Icon>
    <input id="search-input-desktop" placeholder="{i18n(I18nKey.search)} title, tag, category..." bind:value={keywordDesktop} on:focus={() => setPanelVisibility(true, true)}
           class="transition-all pl-10 text-sm bg-transparent outline-0
         h-full w-40 active:w-60 focus:w-60 text-black/50 dark:text-white/50 pr-10"
    >
    <span class="hidden xl:block text-[11px] text-30 pr-3">/</span>
</div>

<!-- toggle btn for phone/tablet view -->
<button on:click={togglePanel} aria-label="Search Panel" id="search-switch"
        class="btn-plain scale-animation lg:!hidden rounded-lg w-11 h-11 active:scale-90">
    <Icon icon="material-symbols:search" class="text-[1.25rem]"></Icon>
</button>

<!-- search panel -->
<div id="search-panel" class="float-panel float-panel-closed search-panel absolute md:w-[30rem]
top-20 left-4 md:left-[unset] right-4 shadow-2xl rounded-2xl p-2">

    <!-- search bar inside panel for phone/tablet -->
    <div id="search-bar-inside" class="flex relative lg:hidden transition-all items-center h-11 rounded-xl
      bg-black/[0.04] hover:bg-black/[0.06] focus-within:bg-black/[0.06]
      dark:bg-white/5 dark:hover:bg-white/10 dark:focus-within:bg-white/10
  ">
        <Icon icon="material-symbols:search" class="absolute text-[1.25rem] pointer-events-none ml-3 transition my-auto text-black/30 dark:text-white/30"></Icon>
		<input placeholder="{i18n(I18nKey.search)} title, tag, category..." bind:value={keywordMobile}
               class="pl-10 absolute inset-0 text-sm bg-transparent outline-0
               focus:w-60 text-black/50 dark:text-white/50"
        >
    </div>

    <!-- search results -->
	{#if isSearching}
		<div class="search-feedback-card mt-2">
			<div class="text-sm font-semibold text-75">Searching...</div>
			<div class="text-xs text-50 mt-1">Checking title, description, category, and tags.</div>
		</div>
	{:else if hasSearched && result.length === 0}
		<div class="search-feedback-card mt-2">
			<div class="text-sm font-semibold text-75">No matching notes found</div>
			<div class="text-xs text-50 mt-1">Try fewer keywords or search by category such as Labs, Theory, or Projects.</div>
		</div>
	{:else if initialized && result.length > 0}
		{#each result as item}
			<a href={item.url}
			   class="transition first-of-type:mt-2 lg:first-of-type:mt-0 group block
		   rounded-xl text-lg px-3 py-2 hover:bg-[var(--btn-plain-bg-hover)] active:bg-[var(--btn-plain-bg-active)]">
				<div class="transition text-90 inline-flex font-bold group-hover:text-[var(--primary)] mb-1">
					{item.meta.title}<Icon icon="fa6-solid:chevron-right" class="transition text-[0.75rem] translate-x-1 my-auto text-[var(--primary)]"></Icon>
				</div>

				<div class="flex flex-wrap gap-1.5 mb-1.5">
					{#if item.meta.category}
						<span class="search-chip">{item.meta.category}</span>
					{/if}
					{#each parseTags(item.meta.tags) as tag}
						<span class="search-chip">#{tag}</span>
					{/each}
				</div>

				{#if item.meta.description}
					<div class="transition text-sm text-50 mb-1.5">{item.meta.description}</div>
				{/if}

				<div class="transition text-sm text-50">
					{@html item.excerpt}
				</div>
			</a>
		{/each}
	{:else if !initialized}
		<div class="search-feedback-card mt-2">
			<div class="text-sm font-semibold text-75">Search index is loading</div>
			<div class="text-xs text-50 mt-1">Results will appear once the Pagefind index is ready.</div>
		</div>
	{/if}
</div>

<style>
  input:focus {
    outline: 0;
  }
  .search-panel {
    max-height: calc(100vh - 100px);
    overflow-y: auto;
  }
  .search-feedback-card {
	border-radius: 0.75rem;
	border: 1px dashed var(--line-divider);
	padding: 0.75rem;
	background: var(--btn-plain-bg-hover);
  }
  .search-chip {
	border-radius: 999px;
	padding: 0.125rem 0.5rem;
	font-size: 0.7rem;
	line-height: 1rem;
	color: var(--btn-content);
	background: var(--btn-regular-bg);
  }
</style>
