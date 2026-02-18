---
layout: default
title: "Demos"
---

# Demos

<div id="demo-tabs" style="display:flex; gap:0.5rem; flex-wrap:wrap; margin-bottom:0.8rem;">
  <button class="demo-btn" onclick="loadDemo('https://lsuxxbj2xmy5nrdnw7i53hxtiu0hazyg.lambda-url.ap-northeast-1.on.aws/', this)">N-Queens Problem</button>
  <button class="demo-btn" onclick="loadDemo('https://vk2x4g4ctfs3rpc2rhr6f5jnfy0meufu.lambda-url.ap-northeast-1.on.aws/', this)">Traveling Salesman Problem</button>
  <button class="demo-btn" onclick="loadDemo('https://pwnweogwdi7ykfx2dzxwewa4li0kdslm.lambda-url.ap-northeast-1.on.aws/', this)">Graph Problems</button>
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
</script>
