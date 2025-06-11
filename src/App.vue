<template>
  <div id="app">
    <h1>Pyodide + Vue</h1>
    <pre>{{ output }}</pre>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const output = ref("Initializing...");

async function initPyodide({
  indexURL = "/pyodide/",
  packages = [],
  wheelFiles = [],
  mountFiles = {},
  verbose = false
} = {}) {
  // Load the Pyodide JS script dynamically
  await new Promise((resolve, reject) => {
    if (window.loadPyodide) return resolve(); // Already loaded

    const script = document.createElement("script");
    script.src = `${indexURL}pyodide.js`;
    script.onload = resolve;
    script.onerror = reject;
    document.head.appendChild(script);
  });

  // Initialize Pyodide runtime
  const pyodide = await loadPyodide({ indexURL });
  if (verbose) console.log("Pyodide loaded");

  // Load core packages
  if (packages.length > 0) {
    await pyodide.loadPackage(packages);
    if (verbose) console.log("Packages loaded:", packages);
  }

  // Install .whl Python wheel files via micropip
  if (wheelFiles.length > 0) {
    await pyodide.loadPackage("micropip");
    for (const wheel of wheelFiles) {
      const wheelPath = `${indexURL}${wheel}`;
      await pyodide.runPythonAsync(`
import micropip
await micropip.install("${wheelPath}")
`);
      if (verbose) console.log(`Installed wheel: ${wheel}`);
    }
  }

  // Mount files (key-value: { pathInFS: Uint8Array | string URL })
  for (const [path, data] of Object.entries(mountFiles)) {
    let content;
    if (data instanceof Uint8Array) {
      content = data;
    } else if (typeof data === "string") {
      const response = await fetch(data);
      content = new Uint8Array(await response.arrayBuffer());
    } else {
      throw new Error(`Unsupported file data for ${path}`);
    }
    pyodide.FS.writeFile(path, content);
    if (verbose) console.log(`Mounted file: ${path}`);
  }

  return pyodide;
}

onMounted(async () => {
  output.value = "Loading Pyodide...";

  const pyodide = await initPyodide({
    indexURL: "/pyodide/",
    packages: ["numpy", "micropip"],
    wheelFiles: ["rsa_json_encryption-0.1.0-py3-none-any.whl"],
    mountFiles: {
      "/tmp/private_key.pem": "/tmp/private_key.pem",
      "/tmp/public_key.pem": "/tmp/public_key.pem",
    },
    verbose: true
  });

  window.pyodide = pyodide;
  
  output.value = await pyodide.runPythonAsync(`
from rjson import *

public_key_path = '/tmp/public_key.pem'
private_key_path = '/tmp/private_key.pem'

public_key = PEMFileReader(
                public_key_path).load_public_pkcs8_key()
private_key = PEMFileReader(
                private_key_path).load_private_pkcs8_key()

encryptor = SimpleRSAChunkEncryptor(public_key, private_key)

plaintext = "Hello, RSA encryption with .pem support!"

encrypted_text = encryptor.encrypt_string(plaintext)

decrypted_text = encryptor.decrypt_string(encrypted_text)

res = "\\n".join([plaintext,encrypted_text[:50]+"...",decrypted_text])

res
  `);
});

</script>

<style scoped>
pre {
  background-color: #eee;
  padding: 1em;
}
</style>
