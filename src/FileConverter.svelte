<script>
  import Drop from "./Drop.svelte";
  import { Base64 } from "js-base64";
  import JSZip from "jszip";
  import { createEventDispatcher } from "svelte";
  import ProgressBar from "@okrad/svelte-progressbar";

  const dispatch = createEventDispatcher();

  let links = []; // HTML <a> elements as string
  let buffers = [];
  let zipProcessing = false;
  let numberOfFilesDropped = 0;

  // progress bar, set to percentage of completed processing
  // https://github.com/okrad/svelte-progressbar#readme
  let series = 0;

  // track processing of PDFs, set true when all PDFs are processed
  let processing = false;

  export let process = (data) => data;

  let zip = new JSZip();

  // update progress bar
  $: {
    if (numberOfFilesDropped) {
      // set it always >= 15% so it looks like something is happening
      series = Math.max(
        15,
        Math.round((100 * buffers.length) / numberOfFilesDropped)
      );
    } else {
      series = 0;
    }
  }

  // https://gist.github.com/aindong/6669cdf72871e947640ce2d16fb68848
  const download = (buffer, { filename, type }) => {
    const blob = new Blob([buffer], { type });
    const a = window.document.createElement("a");
    a.href = window.URL.createObjectURL(blob);
    a.download = filename;
    document.body.appendChild(a);
    a.click(); // IE: "Access is denied"; see: https://connect.microsoft.com/IE/feedback/details/797361/ie-10-treats-blob-url-as-cross-origin-and-denies-access
    document.body.removeChild(a);
  };

  // return dataurl string
  const createDataURL = (buffer, opts) => {
    const prefix =
      "data:" +
      (!opts?.type || opts?.type === ""
        ? "application/octet-stream"
        : opts.type) +
      ";base64,";
    const base64 = Base64.fromUint8Array(buffer); // see https://www.npmjs.com/package/js-base64
    return prefix + base64;
  };

  // start of drop operation, before any processing occurs
  const handleStart = ({ detail }) => {
    clearResults();
    numberOfFilesDropped = detail.fileCount;
    processing = true;
    dispatch("start");
  };

  // end of drop operation - not end of processing operations!
  const handleComplete = ({ detail }) => {
    dispatch("complete");
  };

  const handleFile = async ({ detail: file }) => {
    const fileBuffer = await file.arrayBuffer();
    const buffer = new Uint8Array(fileBuffer);

    // perform the file conversion
    const { data: bufferModified, modified } = process(buffer, { file });

    const dataurl = createDataURL(bufferModified, { type: file.type });

    // create link and add to results
    if (modified) {
      // changes made to file
      links.push(
        `<a href="${dataurl}" download="${file.name}">${file.name}</a>`
      );
    } else {
      // no change made to file
      links.push(
        `<a href="${dataurl}" download="${file.name}">${file.name}</a> - suspect code not found, no change made`
      );
    }
    links = links;

    // add file to list of buffers
    buffers.push({ filename: file.name, buffer: bufferModified });
    buffers = buffers;

    // if all files completed, disable processing flag
    if (buffers.length === numberOfFilesDropped) {
      processing = false;
    }
  };

  const clearResults = () => {
    links = [];
    buffers = [];
  };

  const generateZip = async () => {
    zipProcessing = true;

    // takes all buffers and create a zip file
    buffers.forEach(({ filename, buffer }) => {
      zip.file(filename, buffer, {
        binary: true,
        compressionOptions: {
          level: 5,
        },
      });
    });

    const u8 = await zip.generateAsync({ type: "uint8array" });

    // init user download
    download(u8, { filename: "all.zip", type: "application/zip" });
    zipProcessing = false;
  };
</script>

<div class="dropzone">
  <Drop
    on:file={handleFile}
    on:start={handleStart}
    on:complete={handleComplete}
    multifile
  >
    <div class="drop-inner">
      <slot>Drag and drop files here</slot>
      {#if processing}<ProgressBar {series} />{/if}
      {#if links.length}
        <h2>Output</h2>

        <div>
          <button on:click={(e) => generateZip()} disabled={zipProcessing}
            >{zipProcessing ? "Please wait..." : "Download all as ZIP"}</button
          >
        </div>
        {#each links as row}
          {@html row}<br />
        {/each}
      {/if}
    </div>
  </Drop>
</div>

<style>
  .dropzone {
    background-color: white;
    position: fixed;
    padding: 0;
    margin: 0;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
  }
  .drop-inner {
    color: black;
    padding: 1rem;
  }
</style>
