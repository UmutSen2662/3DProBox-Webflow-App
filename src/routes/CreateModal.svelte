<script lang="ts">
    import { ComposedModal, ModalHeader, ModalBody, TextArea, Form } from "carbon-components-svelte";
  	import { onMount } from "svelte";

    export let listAllModels;
	export let rootElementToInsert: AnyElement | null;
	export let isCreateModalOpen = false;
	export let newIframe: string;

	let textarea: HTMLTextAreaElement;
	onMount(() => {
		textarea = document.querySelector("textarea") as HTMLTextAreaElement;
	});

	const onOpen = () => {
		if (textarea) {
			textarea.value = newIframe;
		};
		newIframe = "";
	};

	// Create an iframe DOMElement and insert it into the root container
	const createIframe = async (e: SubmitEvent) => {
		e.preventDefault();
		if (!rootElementToInsert || !rootElementToInsert?.children) {
			webflow.notify({
				type: "Error",
				message: "No root element to insert the iframe into",
			});
			return;
		}
		if (textarea.value == "") {
			webflow.notify({
				type: "Error",
				message: "No embed code found",
			});
			return;
		}
		if (rootElementToInsert?.children) {
			// Set attributes from iframe code snippet onto the element preset
			// Again, we don't need to await every setAttribute() call, so we can
			// track a list of the Promises returned from that method, and await for
			// them to resolve all at once
			const template = document.createElement('template');
			template.innerHTML = textarea.value.trim();
			const result = template.content.children[0];

			let fine = false;
			if (result?.hasAttributes()){
				if (result.attributes.length > 0) {
					fine = true;
				};
			};
			if (!fine) {
				webflow.notify({
					type: "Error",
					message: "Invalid embed code",
				});
				return;
			};

			// Create the iframe from a DOM element preset
			const newDomElement = await rootElementToInsert.append(webflow.elementPresets.DOM);
			await newDomElement.setTag('iframe');
			const attributePromises: Promise<null>[] = [];
			
			for (const attr of result.attributes) {
				let value = attr.value;
				if (attr.name == "src"){
					const c1 = document.getElementById("autoload") as HTMLInputElement;
					const c2 = document.getElementById("transparency") as HTMLInputElement;
					const c3 = document.getElementById("button_load") as HTMLInputElement;
					const c4 = document.getElementById("info_buttons") as HTMLInputElement;
					value = value.indexOf("?") != -1 ? value.slice(0, value.indexOf("?")) : value;
					value = value + `?autoload=${c1.checked}&bg_transparent=${c2.checked}&buttonload=${c3.checked}&info_buttons=${c4.checked}`;
				}
				attributePromises.push(newDomElement.setAttribute(attr.name, value));
			}
		
			
			isCreateModalOpen = false;
			await Promise.all(attributePromises);
			listAllModels();
		}
	};
</script>

<ComposedModal open={isCreateModalOpen} on:open={onOpen} on:close={() => isCreateModalOpen = false}>
	<ModalHeader label="Embed" title="Embed 3DProBox Model" />
	<ModalBody class="modal-body">
		<Form class="model-embed-form" on:submit={createIframe}>
			<TextArea labelText="Add embed code" name="new-iframe-code" placeholder={`Add iframe embed code you got from 3DProBox`} />
			<button class="model-embed" type="submit">Submit</button>
		</Form>
		<div class="model-settings">
			<hr>
			<div>
				<input type="checkbox" name="Autoload" id="autoload" />
				<label for="autoload"> Autoload</label>
			</div>
			<hr>
			<div>
				<input type="checkbox" name="Button Load" id="button_load" />
				<label for="button_load"> Button Load</label>
			</div>
			<hr>
			<div>
				<input type="checkbox" name="Info Buttons" id="info_buttons" />
				<label for="info_buttons"> Info Buttons</label>
			</div>
			<hr>
			<div>
				<input type="checkbox" name="Transparent Background" id="transparency" />
				<label for="transparency"> Background transparency</label>
			</div>
			<hr>
		</div>
	</ModalBody>
</ComposedModal>