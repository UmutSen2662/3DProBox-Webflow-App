<script lang="ts">
	import EditAttributesModal from "./EditAttributesModal.svelte";
	import SelectUserModels from "./SelectUserModels.svelte";
	import { ClickableTile} from "carbon-components-svelte";
	import CreateIframeModal from "./CreateModal.svelte";
	import { onMount } from "svelte";
	import './styles.css';

	// State vars
	// List of all iframe custom elements on a given page (along with a map of their HTML attributes)
	let iframeElements: Promise<IFrameElement>[];

	// Flag to see if we've checked the page for iframes yet
	let hasCheckedIframes = false;

	// Flag to see if the modal to edit iframe attributes is open
	let isEditModalOpen = false;

	// The selected iframe element to edit
	let iframeSelected: IFrameElement;

	// Flag to see if the modal to create iframe custom elements is open
	let isCreateModalOpen = false;

	// Flag to see if the user model selectoe is open
	let isUserModelOpen = false;

	// New iframe sent by SelectUserModels
	let newIframe: string = "";

	// The container element to insert/append the iframe to
	let rootElementToInsert: AnyElement | null;

	// Search the page for all Models
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
			

			// Get a list of iframes
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
	<h1>{"3DProBox Model Manager"}</h1>
	<div class="main-btn-set">
		<button on:click={() => {isUserModelOpen = true}}>Search user models</button>
		<button on:click={openCreateFlow}>Embed model</button>
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
		{#if iframeSelected}
		<EditAttributesModal
			listAllModels={listAllModels}
			iframeSelected={iframeSelected}
			bind:isEditModalOpen={isEditModalOpen}
		/>
		{/if}
	{:else if hasCheckedIframes && iframeElements?.length <= 0 }
		<p>No Models found</p>
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
