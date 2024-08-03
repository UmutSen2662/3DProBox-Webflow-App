<script lang="ts">
	import { ComposedModal, ModalHeader, ModalBody, Form } from "carbon-components-svelte";

    export let listAllModels;
	export let isEditModalOpen = false;
	export let iframeSelected: IFrameElement;

	// Set user-edited attributes back to iframe element
	const editSource = async (e: SubmitEvent) => {
		e.preventDefault();
		if (e.target && iframeSelected) {
			const c1 = document.getElementById("autoload") as HTMLInputElement;
			const c2 = document.getElementById("transparency") as HTMLInputElement;
			const c3 = document.getElementById("button_load") as HTMLInputElement;
			const c4 = document.getElementById("info_buttons") as HTMLInputElement;
			
			let source = iframeSelected.attrs.src || "";
			source = source.indexOf("?") != -1 ? source.slice(0, source.indexOf("?")) : source;
			source = source + `?autoload=${c1.checked}&bg_transparent=${c2.checked}&buttonload=${c3.checked}&info_buttons=${c4.checked}`;
			await iframeSelected.el.setAttribute("src", source);

			isEditModalOpen = false;
			listAllModels();
		}
	};
</script>

<ComposedModal open={isEditModalOpen} on:close={() => isEditModalOpen = false}>
	<ModalHeader title="Edit attributes" />
	<ModalBody class="modal-body">
		<Form class="iframe-attr-form" on:submit={editSource}>
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
			<button type="submit">Submit</button>
		</Form>
	</ModalBody>
</ComposedModal>