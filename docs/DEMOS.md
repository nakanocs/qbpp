---
layout: default
title: "Demos"
---

# Demos

> **Note:** These demos run on AWS Lambda with limited resources.
> Performance is typically **several times slower** than a standard PC.
> On a modern desktop, QUBO++ runs significantly faster.

> **Note:** PyQUBO++ is a Python frontend powered by the QUBO++ C++ engine.
> It is currently under active development and its API may change without notice.



<div id="demo-tabs" style="display:flex; gap:0.5rem; flex-wrap:wrap; margin-bottom:0.8rem;">
  <button class="demo-btn" onclick="loadDemo('https://uzfftqxjkl26ncncbfz42smpmy0zeygz.lambda-url.ap-northeast-1.on.aws/', this)">QUBO++ Playground</button>
  <button class="demo-btn" onclick="loadDemo('https://lsuxxbj2xmy5nrdnw7i53hxtiu0hazyg.lambda-url.ap-northeast-1.on.aws/', this)">N-Queens Problem</button>
  <button class="demo-btn" onclick="loadDemo('https://vk2x4g4ctfs3rpc2rhr6f5jnfy0meufu.lambda-url.ap-northeast-1.on.aws/', this)">Traveling Salesman Problem</button>
  <button class="demo-btn" onclick="loadDemo('https://pwnweogwdi7ykfx2dzxwewa4li0kdslm.lambda-url.ap-northeast-1.on.aws/', this)">Graph Problems</button>
  <button class="demo-btn" onclick="loadDemo('https://3nsgxoub3dgwgfb3dk27oc7yqu0zmmog.lambda-url.ap-northeast-1.on.aws/', this)">PyQUBO++ Playground</button>
</div>

<iframe id="demo-frame" style="width:100%; height:85vh; border:1px solid #ccc; border-radius:6px; display:none;"></iframe>
<p id="demo-placeholder" style="color:#888;">Select a demo above to load it here.</p>

<style>
.demo-btn {
  padding: 0.4rem 1rem;
  border: 1px solid #0969da;
  border-radius: 4px;
  background: #fff;
  color: #0969da;
  cursor: pointer;
  font-size: 0.95rem;
}
.demo-btn:hover {
  background: #f0f6ff;
}
.demo-btn.active {
  background: #0969da;
  color: #fff;
}
</style>

<script>
function loadDemo(url, btn) {
  var frame = document.getElementById('demo-frame');
  var placeholder = document.getElementById('demo-placeholder');
  frame.src = url;
  frame.style.display = 'block';
  placeholder.style.display = 'none';
  var btns = document.querySelectorAll('.demo-btn');
  for (var i = 0; i < btns.length; i++) btns[i].classList.remove('active');
  btn.classList.add('active');
}

// Handle save-file requests from demo iframes (showSaveFilePicker blocked in cross-origin iframes)
window.addEventListener('message', async function(e) {
  if (!e.data || e.data.type !== 'save-file') return;
  var frame = document.getElementById('demo-frame');
  if (!frame || !frame.contentWindow) return;
  if (e.source !== frame.contentWindow) return;
  if (!window.showSaveFilePicker) {
    frame.contentWindow.postMessage({type: 'save-file-result', error: 'unsupported'}, '*');
    return;
  }
  try {
    var name = e.data.name || 'source.cpp';
    var ext = name.split('.').pop() || '';
    var types = ext === 'py'
      ? [{description: 'Python Source', accept: {'text/plain': ['.py']}}]
      : [{description: 'C++ Source', accept: {'text/plain': ['.cpp','.hpp','.h','.cc','.cxx']}}];
    var handle = await window.showSaveFilePicker({
      suggestedName: name,
      types: types
    });
    var writable = await handle.createWritable();
    await writable.write(e.data.content);
    await writable.close();
    frame.contentWindow.postMessage({type: 'save-file-result', ok: true, name: handle.name}, '*');
  } catch (err) {
    if (err.name === 'AbortError') {
      frame.contentWindow.postMessage({type: 'save-file-result', cancelled: true}, '*');
    } else {
      frame.contentWindow.postMessage({type: 'save-file-result', error: err.message}, '*');
    }
  }
});
</script>
