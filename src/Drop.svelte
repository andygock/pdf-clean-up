<script>
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  export let multifile = false;
  export let readAsBuffer = false;
  export let readAsText = false;
  export let readAsArray = false;

  let hover = false;

  const sendBuffer = async (file) => {
    const fileBuffer = await file.arrayBuffer();
    const buffer = new Uint8Array(fileBuffer);
    dispatch("read", buffer);
  };

  const sendText = async (file) => {
    const text = await file.text();
    dispatch("read", text);
  };

  const sendArray = async (file) => {
    const text = await file.text();
    const lines = text.split(/\r?\n/);
    dispatch("read", lines);
  };

  const handleDrop = (ev) => {
    ev.preventDefault();

    // don't think dragleave occurs after a drop event
    hover = false;

    if (ev.dataTransfer.items) {
      if (!multifile && ev.dataTransfer.items.length > 1) {
        alert("Error: Multiple file drops not supported.");
        return;
      }
      dispatch("start", { fileCount: ev.dataTransfer.items.length });
      for (let i = 0; i < ev.dataTransfer.items.length; i++) {
        if (ev.dataTransfer.items[i].kind === "file") {
          // https://developer.mozilla.org/en-US/docs/Web/API/File
          let file = ev.dataTransfer.items[i].getAsFile();

          dispatch("file", file);

          if (readAsBuffer) sendBuffer(file);
          if (readAsText) sendText(file);
          if (readAsArray) sendArray(file);
        }
      }
      dispatch("complete");
    }
  };

  const handleDragover = (e) => {
    e.preventDefault();
    if (!hover) hover = true;
  };

  const handleDragleave = (e) => {
    e.preventDefault();
    hover = false;
  };
</script>

<div
  style="background-color: {hover ? 'palegreen' : 'inherit'}"
  on:drop={handleDrop}
  on:dragover={handleDragover}
  on:dragleave={handleDragleave}
>
  <slot />
</div>

<style>
  div {
    border: solid 1px lightgray;
    height: 100%;
    width: 100%;
  }
</style>
