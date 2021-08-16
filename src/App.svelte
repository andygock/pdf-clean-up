<script>
  // compare contente of two buffers with same size
  // return 0 if same content, return -1 if different
  const bufferCompare = (b1, b2) => {
    if (b1.byteLength !== b2.byteLength) {
      throw new Error("Needle and haystack is different size");
    }

    for (let i = 0; i < b1.byteLength; i += 1) {
      if (b1[i] !== b2[i]) return -1;
    }

    // buffers are identical
    return 0;
  };

  // look for matching needle buffer in haystack buffer and return position of start of match
  // returns -1 if no match found
  const byteMatch = (haystack, needle) => {
    if (needle.byteLength > haystack.byteLength) {
      throw new Error("Needle size is larger than haystack size");
    }
    for (let i = 0; i < haystack.byteLength - needle.length; i += 1) {
      if (
        bufferCompare(haystack.slice(i, i + needle.byteLength), needle) === 0
      ) {
        // console.log("Match at position:", i);
        return i;
      }
    }

    // no match found
    return -1;
  };

  // search and replace buffer contents, this will modify the buffer
  const bufferReplace = (haystack, needleBefore, needleAfter) => {
    if (needleBefore.byteLength !== needleAfter.byteLength) {
      throw new Error("Before and after replace contents are different sizes");
    }
    const matchPos = byteMatch(haystack, needleBefore); // returns position of match, -1 if no match found

    if (matchPos >= 0) {
      // match found, replace buffer contents at position match
      for (let i = matchPos; i < matchPos + needleBefore.byteLength; i += 1) {
        haystack[i] = needleAfter[i];
      }
      return matchPos;
    }

    // no match found
    return -1;
  };

  // JS part of PDF object which represents "this.print({bUI: false, bSilent: true, bShrinkToFit: true});"
  const needle = new Uint8Array([
    0x30, 0x20, 0x6f, 0x62, 0x6a, 0x0a, 0x3c, 0x3c, 0x2f, 0x4c, 0x65, 0x6e,
    0x67, 0x74, 0x68, 0x20, 0x35, 0x38, 0x2f, 0x46, 0x69, 0x6c, 0x74, 0x65,
    0x72, 0x2f, 0x46, 0x6c, 0x61, 0x74, 0x65, 0x44, 0x65, 0x63, 0x6f, 0x64,
    0x65, 0x3e, 0x3e, 0x0a, 0x73, 0x74, 0x72, 0x65, 0x61, 0x6d, 0x0a, 0x78,
    0x9c, 0x2b, 0xc9, 0xc8, 0x2c, 0xd6, 0x2b, 0x28, 0xca, 0xcc, 0x2b, 0xd1,
    0xa8, 0x4e, 0x0a, 0xf5, 0xb4, 0x52, 0x48, 0x4b, 0xcc, 0x29, 0x4e, 0xd5,
    0x51, 0x48, 0x0a, 0xce, 0xcc, 0x49, 0xcd, 0x2b, 0xb1, 0x52, 0x28, 0x29,
    0x2a, 0x05, 0x73, 0x33, 0x80, 0x6a, 0xb2, 0x43, 0xf2, 0xdd, 0x32, 0xa1,
    0x62, 0xb5, 0x9a, 0xd6, 0x00, 0x90, 0xfa, 0x15, 0x51, 0x0a, 0x65, 0x6e,
    0x64, 0x73, 0x74, 0x72, 0x65, 0x61, 0x6d, 0x0a, 0x65, 0x6e, 0x64, 0x6f,
    0x62, 0x6a, 0x0a,
  ]);

  const needleNew = new Uint8Array([
    0x30, 0x20, 0x6f, 0x62, 0x6a, 0x0a, 0x3c, 0x3c, 0x2f, 0x4c, 0x65, 0x6e,
    0x67, 0x74, 0x68, 0x20, 0x35, 0x38, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20,
    0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20,
    0x20, 0x3e, 0x3e, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20,
    0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20,
    0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20,
    0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20,
    0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20,
    0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x0a, 0x65, 0x6e,
    0x64, 0x73, 0x74, 0x72, 0x65, 0x61, 0x6d, 0x0a, 0x65, 0x6e, 0x64, 0x6f,
    0x62, 0x6a, 0x0a,
  ]);

  // https://gist.github.com/aindong/6669cdf72871e947640ce2d16fb68848
  const downloadPDF = (buffer, filename) => {
    const pdf = new Blob([buffer], { type: "application/pdf" });
    const a = window.document.createElement("a");
    a.href = window.URL.createObjectURL(pdf);
    a.download = filename;
    document.body.appendChild(a);
    a.click(); // IE: "Access is denied"; see: https://connect.microsoft.com/IE/feedback/details/797361/ie-10-treats-blob-url-as-cross-origin-and-denies-access
    document.body.removeChild(a);
  };

  const processFile = async (file) => {
    // https://developer.mozilla.org/en-US/docs/Web/API/File
    const fileBuffer = await file.arrayBuffer();
    const buffer = new Uint8Array(fileBuffer);
    if (bufferReplace(buffer, needle, needleNew) !== -1) {
      downloadPDF(buffer, file.name || "modified.pdf");
    } else {
      // no replacement performed
      alert("Suspect code not detected.");
    }
  };

  const handleDrop = (ev) => {
    ev.preventDefault();

    // ref: https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API/File_drag_and_drop

    if (ev.dataTransfer.items) {
      // Use DataTransferItemList interface to access the file(s)
      if (ev.dataTransfer.items.length > 1) {
        alert("Error: Can not handle more than one file at once!");
        return;
      }
      for (let i = 0; i < ev.dataTransfer.items.length; i++) {
        // If dropped items aren't files, reject them
        if (ev.dataTransfer.items[i].kind === "file") {
          let file = ev.dataTransfer.items[i].getAsFile();
          processFile(file);
        }
      }
    } else {
      // Use DataTransfer interface to access the file(s)
      if (ev.dataTransfer.files.length > 1) {
        alert("Error: Can not handle more than one file at once!");
        return;
      }
      for (let i = 0; i < ev.dataTransfer.files.length; i++) {
        let file = ev.dataTransfer.files[i];
        processFile(file);
      }
    }
  };
</script>

<div id="drop" on:drop={handleDrop} on:dragover={(e) => e.preventDefault()}>
  <div>
    Drag one PDF file here to remove annoying JavaScript "print now" commands.
    <p>
      PDF files generated by Reckon Hosted contain the following JavaScript code
      which attempts to automatically force the PDF client to open a print
      dialog. This attempts to remove this behaviour, by removing this
      JavaScript code.
    </p>
    <p>
      All modifications is performed inside this browser and no data is stored
      or transmitted elsewhere.
    </p>
    <p>Source: <a href="https://github.com/andygock/pdf-clean-up">GitHub</a></p>
  </div>
</div>

<style>
  #drop {
    background-color: lightyellow;
    height: 100vh;
    font-size: xx-large;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  p {
    font-size: small;
  }
</style>
