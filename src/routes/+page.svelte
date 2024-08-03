<script lang="ts">
	import EditAttributesModal from "./EditAttributesModal.svelte";
	import SelectUserModels from "./SelectUserModels.svelte";
	import { ClickableTile} from "carbon-components-svelte";
	import CreateIframeModal from "./CreateModal.svelte";
	import { onMount } from "svelte";
	import './styles.css';

	// Flags to show/hide modals
	let isEditModalOpen = false;
	let isCreateModalOpen = false;
	let isUserModelOpen = false;
	let hasCheckedIframes = false;

	let iframeElements: Promise<IFrameElement>[];
	let iframeSelected: IFrameElement;
	let newIframe: string = "";
	let rootElementToInsert: AnyElement | null;

	const listAllModels = async () => {
		// Get all elements on the page
		const pageElements = await webflow.getAllElements();
		hasCheckedIframes = true;

		if (pageElements.length > 0) {
			// Mapping each page element on the page to a boolean "true" if an element is an 3DProBox Model viewer
			const domElementTestMapping = await Promise.all(pageElements.map(async (el) => {
				if (el.type === "DOM") {
					if (await (el as DOMElement)?.getAttribute("title") === "3D ProBox Model Viewer") {
						return true;
					}
				}
			}));
			

			// Get a list of 3DProBox iframes
			const filteredElements: DOMElement[] = pageElements.filter((_, idx) => domElementTestMapping[idx]) as DOMElement[];

			// Create a new structure that has the iframe DOMElement object, as well as its attributes
			// on the <iframe> tag
			iframeElements = filteredElements.map(async (el) => {
				const attributesArr = await el.getAllAttributes();
				const attributesObj = attributesArr ? attributesArr.reduce((arr, attr) => ({
					...arr,
					[attr.name]: attr.value,
				}), {}) : null;
				return {
					el,
					attrs: attributesObj,
				} as IFrameElement;
			});
		} else {
			await webflow.notify({
				type: "Info",
				message: "There are no elements on the page",
			});
		}
	};

	// Find the correlated iframe in the Designer and scroll to it
	const selectIframeElement = (el: DOMElement) => {
		webflow.setSelectedElement(el);
	};

	// Ensure the user has a root container to insert the iframe into when we create one
	const openCreateFlow = () => {
		isCreateModalOpen = true;
		webflow.subscribe('selectedelement', (element: null | AnyElement) => {
			rootElementToInsert = element;
		});
	};

	onMount(() => {
		listAllModels();
	})
</script>

<div class="main-container">
	<div style="display: flex; justify-content: space-between; padding: 10px; width: 100%;">
		<p style="font-size: xx-large">{"3DProBox Model Manager"}</p>
		<div class="main-btn-set">
			<button on:click={() => {isUserModelOpen = true}}>Search user models</button>
			<button on:click={openCreateFlow}>Embed model</button>
		</div>
	</div>
	{#if hasCheckedIframes && iframeElements?.length > 0}
		<p>Click each iframe to find it on the Designer</p>
		<div role="group" aria-label="selectable tiles" class="iframe-container">
			{#each iframeElements as el}
				{#await el then iframeEl}
				<ClickableTile class="tile" on:click={(e) => {
					e.preventDefault();
					selectIframeElement(iframeEl.el);
				}}>
					<div style="pointer-events:none">
						<iframe
							title="preview-iframe"
							{...iframeEl.attrs}
						></iframe>
					</div>
					<div class="iframe-btn-group">
						<button on:click={() => {
							isEditModalOpen = true;
							iframeSelected = iframeEl;
						}}>Edit</button>
						<button on:click={async () => {
							await iframeEl.el.remove();
							listAllModels();
						}}>Delete</button>
					</div>
				</ClickableTile>
				{/await}
			{/each}
		</div>
	{:else if hasCheckedIframes && iframeElements?.length <= 0 }
		<p>No Models found</p>
	{/if}
	{#if iframeSelected}
	<EditAttributesModal
		listAllModels={listAllModels}
		iframeSelected={iframeSelected}
		bind:isEditModalOpen={isEditModalOpen}
	/>
	{/if}
	<CreateIframeModal
		listAllModels={listAllModels}
		bind:newIframe={newIframe}
		bind:isCreateModalOpen={isCreateModalOpen}
		bind:rootElementToInsert={rootElementToInsert}
	/>
	<SelectUserModels
		openCreateFlow={openCreateFlow}
		bind:isUserModelOpen={isUserModelOpen}
		bind:newIframe={newIframe}
	/>
</div>
