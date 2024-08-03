<script lang="ts">
    import { ComposedModal, ModalHeader, ModalBody, ClickableTile, TextInput } from "carbon-components-svelte";
    import { onMount } from "svelte";
    import Fuse from "fuse.js";

    export let openCreateFlow;
	export let isUserModelOpen = false;
	export let newIframe: string = "";

    const fakeModels = async() => {return [
        {
            name: "can",
            tags: ["Food & drink", "test"],
            embed: `<iframe src="https://embed.3dprobox.com/models/RXZGosXENpDBSyDf3VS9nsDt?info_buttons=true" title="3D ProBox Model Viewer" style="width: 100%; height: 100%;" frameborder="0" allow="web-share; xr-spatial-tracking" loading="lazy" scrolling="no" referrerpolicy="origin-when-cross-origin" allowfullscreen="allowfullscreen"></iframe>`,
        },
        {
            name: "monkey",
            tags: ["Animals & Pets", "Art & Abstract", "test"],
            embed: `<iframe src="https://embed.3dprobox.com/models/FhgnKv8B4vZpPtn9A8tqUvSZ?info_buttons=true" title="3D ProBox Model Viewer" style="width: 100%; height: 100%;" frameborder="0" allow="web-share; xr-spatial-tracking" loading="lazy" scrolling="no" referrerpolicy="origin-when-cross-origin" allowfullscreen="allowfullscreen"></iframe>`,
        },
        {
            name: "drift car",
            tags: ["Cars & Vehicles", "test"],
            embed: `<iframe src="https://embed.3dprobox.com/models/6HQY27ArNvaSEndSsU9DPM3v?info_buttons=true" title="3D ProBox Model Viewer" style="width: 100%; height: 100%;" frameborder="0" allow="web-share; xr-spatial-tracking" loading="lazy" scrolling="no" referrerpolicy="origin-when-cross-origin" allowfullscreen="allowfullscreen"></iframe>`,
        },
    ]};

    interface api { name: string, tags: string[], embed: string }
    let search: string = "";
    let iframes: api[] = [];
    // The search checker
    onMount(async () => {
        const input = document.getElementsByClassName("search")[0] as HTMLInputElement;
        const form = document.getElementById("form") as HTMLFormElement;
        const models = await fakeModels();
        const fuse = new Fuse(models, {
            keys: ["name", "tags"],
        });

        iframes = models
        form.addEventListener("submit", (e) => {
            e.preventDefault();
            search = input.value;
            iframes = search == "" ? models : fuse.search(search).map((obj) => obj.item);
        })
    });

    function stripSource(iframe: string) {
        if (iframe == null) {
            return ""
        };
        let src_begin = iframe.indexOf('src="') + 5;
        let src_end = iframe.indexOf("?");
        if (src_end === -1) {
            src_end = iframe.indexOf('" title="');
        };
        let raw_source = iframe.slice(src_begin, src_end);
        return raw_source + `?info_buttons=false`
    };

    const selectIframeElement = (embed: string) => {
        newIframe = embed;
        isUserModelOpen = false;
        openCreateFlow();
	};
</script>

<ComposedModal open={isUserModelOpen} on:close={() => isUserModelOpen = false}>
	<ModalHeader title="Select a Model" class="modal-header">
        <form id="form"> <TextInput placeholder="Search using model name or tag" class="search" /> </form>
    </ModalHeader>
	<ModalBody class="modal-body">
        <div class="iframe-container">
            {#each iframes as iframe}
            <ClickableTile class="tile" on:click={(e) => {
                e.preventDefault();
                selectIframeElement(iframe.embed);
            }}>
                <div style="pointer-events:none">
                    <iframe
                        title="preview-iframe"
                        src = {stripSource(iframe.embed)}
                        style="width: 100%; height: 100%; border: none;"
                    ></iframe>
                </div>
                {iframe.name}
            </ClickableTile>
            {/each}
        </div>
	</ModalBody>
</ComposedModal>