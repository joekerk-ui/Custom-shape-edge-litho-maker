<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LithoForge Ultra HD - Solid Manifold Maker</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- Using JSDelivr for maximum reliability -->
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/exporters/STLExporter.js"></script>

    <style>
        body { background-color: #050508; color: #e2e8f0; font-family: system-ui, -apple-system, sans-serif; height: 100vh; display: flex; flex-direction: column; overflow: hidden; }
        .glass { background: rgba(10, 10, 18, 0.95); backdrop-filter: blur(20px); border-bottom: 1px solid rgba(255, 255, 255, 0.08); }
        input[type=range] { accent-color: #6366f1; cursor: pointer; height: 4px; width: 100%; }
        .shape-btn.active { background-color: #4f46e5; border-color: #818cf8; color: white; box-shadow: 0 0 15px rgba(79, 70, 229, 0.4); }
        
        .custom-scrollbar::-webkit-scrollbar { width: 6px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: rgba(0,0,0,0.3); }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #3f3f51; border-radius: 10px; }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover { background: #6366f1; }
        
        #canvas-container canvas { display: block; width: 100% !important; height: 100% !important; }
        .control-label { font-size: 10px; font-weight: 900; color: #71717a; text-transform: uppercase; letter-spacing: 0.05em; display: flex; justify-content: space-between; align-items: center; margin-bottom: 0.2rem; }
        .value-badge { font-family: monospace; color: #a5b4fc; background: rgba(99, 102, 241, 0.1); padding: 1px 5px; border-radius: 3px; font-size: 10px; }
        #drop-zone.drag-over { border-color: #6366f1; background: rgba(99, 102, 241, 0.1); transform: scale(1.01); }
    </style>
</head>
<body>

    <header class="h-16 px-5 glass z-40 flex items-center justify-between shadow-2xl shrink-0">
        <div class="flex items-center gap-3">
            <div class="bg-indigo-600 p-1.5 rounded-lg shadow-[0_0_15px_rgba(79,70,229,0.3)]">
                <i class="fas fa-train text-white text-xl"></i>
            </div>
            <div class="hidden sm:block">
                <h1 class="text-base font-black tracking-tight text-white leading-none italic uppercase">LithoForge <span class="text-indigo-400 font-light">Ultra HD</span></h1>
                <p class="text-[8px] text-zinc-500 font-mono mt-0.5 uppercase tracking-widest text-nowrap font-bold">WTIU O-Scale Engine</p>
            </div>
        </div>
        
        <div class="flex items-center gap-2">
            <button id="render-btn" disabled class="flex items-center gap-2 bg-zinc-100 hover:bg-white disabled:opacity-20 text-black px-4 sm:px-6 py-2 rounded-full transition-all font-black text-[10px] tracking-widest shadow-lg active:scale-95 uppercase">
                <i class="fas fa-hammer"></i> <span class="hidden xs:inline">Generate</span>
            </button>
            <button id="export-btn" disabled class="flex items-center gap-2 bg-indigo-600 hover:bg-indigo-500 disabled:opacity-20 text-white px-4 sm:px-6 py-2 rounded-full transition-all font-black text-[10px] tracking-widest shadow-lg active:scale-95 uppercase">
                <i class="fas fa-save"></i> <span class="hidden xs:inline">Export STL</span>
            </button>
        </div>
    </header>

    <main class="flex flex-1 overflow-hidden relative">
        <!-- Compact Sidebar -->
        <aside class="w-64 sm:w-72 bg-[#08080c] border-r border-white/5 flex flex-col h-full z-20 shadow-2xl shrink-0">
            <div class="flex-1 overflow-y-auto p-4 space-y-6 custom-scrollbar min-h-0">
                
                <!-- 1. Source Image -->
                <section>
                    <div class="control-label"><span>1. Source Image</span> <i class="fas fa-image"></i></div>
                    <label id="drop-zone" for="image-input" class="relative border border-dashed border-zinc-800 hover:border-indigo-500/50 bg-zinc-900/20 rounded-xl p-3 transition-all flex flex-col items-center justify-center gap-2 cursor-pointer h-28 overflow-hidden group">
                        <div id="upload-placeholder" class="flex flex-col items-center gap-1 group-hover:scale-105 transition-transform text-center pointer-events-none">
                            <i class="fas fa-upload text-xl text-zinc-700 group-hover:text-indigo-500 transition-colors"></i>
                            <span class="text-[9px] text-zinc-500 font-bold uppercase">Drag WTIU Photo</span>
                        </div>
                        <img id="img-preview" class="hidden w-full h-full object-contain rounded-lg" alt="Preview">
                        <input id="image-input" type="file" class="hidden" accept="image/*">
                    </label>
                </section>

                <!-- 2. Geometry Type -->
                <section>
                    <div class="control-label"><span>2. Profile</span> <i class="fas fa-shapes"></i></div>
                    <div class="grid grid-cols-1 gap-1.5">
                        <div class="grid grid-cols-2 gap-1.5">
                            <button data-shape="Alpha" class="shape-btn active py-2 rounded-lg text-[9px] font-black uppercase border border-zinc-800 bg-zinc-900 text-zinc-500 transition-all">Exact</button>
                            <button data-shape="Rectangle" class="shape-btn py-2 rounded-lg text-[9px] font-black uppercase border border-zinc-800 bg-zinc-900 text-zinc-500 transition-all">Rect</button>
                        </div>
                        <button data-shape="Curved" class="shape-btn py-2 rounded-lg text-[9px] font-black uppercase border border-zinc-800 bg-zinc-900 text-zinc-500 transition-all">
                            <i class="fas fa-dot-circle mr-1 text-[8px]"></i> Curved Wrap
                        </button>
                    </div>
                </section>

                <!-- 3. Mesh Settings -->
                <section class="space-y-4">
                    <div class="control-label"><span>3. Details</span> <i class="fas fa-sliders-h"></i></div>
                    
                    <div class="space-y-4">
                        <div>
                            <div class="control-label text-zinc-400">Base Plate <span id="base-thick-val" class="value-badge">1.0mm</span></div>
                            <input id="base-thick" type="range" min="0.4" max="4" step="0.1" value="1.0">
                        </div>

                        <div>
                            <div class="control-label text-zinc-400">Litho Height <span id="max-thick-val" class="value-badge">3.0mm</span></div>
                            <input id="max-thick" type="range" min="1" max="10" step="0.1" value="3.0">
                        </div>

                        <div id="curve-control" class="hidden border-l-2 border-indigo-500/30 pl-3 py-0.5 space-y-4">
                            <div>
                                <div class="control-label text-zinc-400">Curve Radius <span id="curve-radius-val" class="value-badge">100mm</span></div>
                                <input id="curve-radius" type="range" min="20" max="500" step="1" value="100">
                            </div>
                        </div>

                        <div>
                            <div class="control-label text-zinc-400">Smoothing <span id="smooth-val" class="value-badge">1.5px</span></div>
                            <input id="smooth-slider" type="range" min="0" max="10" step="0.5" value="1.5">
                        </div>

                        <div>
                            <div class="control-label text-zinc-400">Resolution <span id="res-val" class="value-badge">200px</span></div>
                            <input id="res-slider" type="range" min="50" max="400" step="10" value="200">
                        </div>

                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <span class="control-label italic text-zinc-500">Width (mm)</span>
                                <input id="model-width" type="number" value="100" class="w-full bg-zinc-900 border border-zinc-800 rounded-lg p-1.5 text-[10px] text-indigo-300 focus:outline-none">
                            </div>
                            <div>
                                <span class="control-label italic text-zinc-500">Height (mm)</span>
                                <input id="model-height" type="number" value="100" class="w-full bg-zinc-900 border border-zinc-800 rounded-lg p-1.5 text-[10px] text-indigo-300 focus:outline-none">
                            </div>
                        </div>
                    </div>
                </section>
                
                <div class="pt-4 opacity-20 text-[7px] text-center font-bold uppercase tracking-widest pointer-events-none">
                    Scroll for more settings
                </div>
            </div>
        </aside>

        <!-- Viewport Area -->
        <div class="flex-1 relative bg-[radial-gradient(circle_at_center,_#11111a_0%,_#050508_100%)]">
            <div id="canvas-container" class="w-full h-full"></div>
            
            <div id="loading-overlay" class="hidden absolute inset-0 bg-black/80 backdrop-blur-md flex items-center justify-center z-50">
                <div class="flex flex-col items-center gap-4 text-center">
                    <div class="w-12 h-12 rounded-full border-4 border-indigo-500/10 border-t-indigo-500 animate-spin"></div>
                    <div class="space-y-1">
                        <p id="loading-text" class="font-bold text-white text-sm tracking-widest uppercase italic">Sculpting Solid Volume</p>
                        <div class="w-40 h-1 bg-zinc-900 rounded-full overflow-hidden">
                            <div id="loading-bar" class="w-0 h-full bg-indigo-500 transition-all duration-300"></div>
                        </div>
                    </div>
                </div>
            </div>

            <div id="toast" class="absolute bottom-6 left-1/2 -translate-x-1/2 px-5 py-2.5 bg-zinc-900 border border-zinc-800 rounded-full text-[9px] font-black text-indigo-400 tracking-widest shadow-2xl opacity-0 transition-opacity uppercase pointer-events-none text-nowrap">
                <span id="toast-text">Ready</span>
            </div>
        </div>
    </main>

    <canvas id="hidden-canvas" class="hidden"></canvas>

    <script>
        // State
        let state = {
            image: null,
            shape: 'Alpha',
            maxThickness: 3.0,
            baseThickness: 1.0,
            curveRadius: 100,
            resolution: 200,
            smoothing: 1.5,
            width: 100,
            height: 100
        };

        let scene, camera, renderer, controls, mesh;

        function initEngine() {
            const container = document.getElementById('canvas-container');
            if (!container) return;

            if (typeof THREE === 'undefined' || !THREE.OrbitControls || !THREE.STLExporter) {
                setTimeout(initEngine, 500);
                return;
            }

            scene = new THREE.Scene();
            camera = new THREE.PerspectiveCamera(45, container.clientWidth / container.clientHeight, 0.1, 10000);
            camera.position.set(0, -200, 200);

            renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
            renderer.setSize(container.clientWidth, container.clientHeight);
            renderer.setPixelRatio(window.devicePixelRatio);
            container.appendChild(renderer.domElement);

            scene.add(new THREE.AmbientLight(0xffffff, 0.6));
            const mainLight = new THREE.DirectionalLight(0xffffff, 1.0);
            mainLight.position.set(50, 50, 500);
            scene.add(mainLight);

            const grid = new THREE.GridHelper(500, 50, 0x1a1a2e, 0x111111);
            grid.rotation.x = Math.PI / 2;
            scene.add(grid);

            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;

            function animate() {
                requestAnimationFrame(animate);
                if (controls) controls.update();
                if (renderer && scene && camera) renderer.render(scene, camera);
            }
            animate();

            window.addEventListener('resize', () => {
                if (!container || !camera || !renderer) return;
                camera.aspect = container.clientWidth / container.clientHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(container.clientWidth, container.clientHeight);
            });
            
            showToast("System Ready");
        }

        function handleFile(file) {
            if (!file || !file.type.startsWith('image/')) return;
            const reader = new FileReader();
            reader.onload = (ev) => {
                state.image = ev.target.result;
                const preview = document.getElementById('img-preview');
                const placeholder = document.getElementById('upload-placeholder');
                const renderBtn = document.getElementById('render-btn');
                if (preview) { preview.src = ev.target.result; preview.classList.remove('hidden'); }
                if (placeholder) placeholder.classList.add('hidden');
                if (renderBtn) renderBtn.disabled = false;
                showToast("Trace Prepared");
            };
            reader.readAsDataURL(file);
        }

        async function processImage() {
            if (!state.image) return;
            const loading = document.getElementById('loading-overlay');
            const lText = document.getElementById('loading-text');
            const lBar = document.getElementById('loading-bar');
            loading.classList.remove('hidden');

            try {
                lBar.style.width = '20%';
                const img = new Image();
                img.src = state.image;
                await new Promise((res) => img.onload = res);

                const canvas = document.getElementById('hidden-canvas');
                const ctx = canvas.getContext('2d', { willReadFrequently: true });
                const resW = state.resolution;
                const resH = Math.round(state.resolution * (img.height / img.width));
                
                canvas.width = resW;
                canvas.height = resH;
                if (state.smoothing > 0) ctx.filter = `blur(${state.smoothing}px)`;
                ctx.drawImage(img, 0, 0, resW, resH);
                const pixels = ctx.getImageData(0, 0, resW, resH).data;

                lBar.style.width = '40%';
                lText.innerText = "Stitching Solid Volume...";

                const geometry = new THREE.BufferGeometry();
                const vertices = [];
                const indices = [];
                
                const isOpaque = (x, y) => {
                    if (state.shape === 'Rectangle') return true;
                    if (x < 0 || x >= resW || y < 0 || y >= resH) return false;
                    return pixels[(y * resW + x) * 4 + 3] > 120;
                };

                const gridIndices = new Int32Array(resW * resH).fill(-1);
                let vCount = 0;

                for (let y = 0; y < resH; y++) {
                    for (let x = 0; x < resW; x++) {
                        if (isOpaque(x, y)) {
                            const u = x / (resW - 1);
                            const v = 1 - (y / (resH - 1));
                            const idx = (y * resW + x) * 4;
                            const bri = (0.299 * pixels[idx] + 0.587 * pixels[idx+1] + 0.114 * pixels[idx+2]) / 255;
                            const frontZ = (1 - bri) * state.maxThickness + state.baseThickness;
                            const backZ = 0;

                            const calcPos = (uu, vv, zz) => {
                                if (state.shape === 'Curved') {
                                    const r = state.curveRadius + zz;
                                    const theta = (uu - 0.5) * (state.width / state.curveRadius);
                                    return [r * Math.sin(theta), (vv - 0.5) * state.height, r * Math.cos(theta) - state.curveRadius];
                                }
                                return [(uu - 0.5) * state.width, (vv - 0.5) * state.height, zz];
                            };

                            const fp = calcPos(u, v, frontZ);
                            const bp = calcPos(u, v, backZ);
                            gridIndices[y * resW + x] = vCount;
                            vertices.push(...fp, ...bp);
                            vCount++;
                        }
                    }
                }

                lBar.style.width = '70%';

                for (let y = 0; y < resH; y++) {
                    for (let x = 0; x < resW; x++) {
                        const i00 = gridIndices[y * resW + x];
                        if (i00 === -1) continue;
                        const i10 = (x < resW - 1) ? gridIndices[y * resW + (x + 1)] : -1;
                        const i01 = (y < resH - 1) ? gridIndices[(y + 1) * resW + x] : -1;
                        const i11 = (x < resW - 1 && y < resH - 1) ? gridIndices[(y + 1) * resW + (x + 1)] : -1;

                        if (i10 !== -1 && i01 !== -1 && i11 !== -1) {
                            indices.push(i00 * 2, i01 * 2, i11 * 2, i00 * 2, i11 * 2, i10 * 2);
                            indices.push(i00 * 2 + 1, i11 * 2 + 1, i01 * 2 + 1, i00 * 2 + 1, i10 * 2 + 1, i11 * 2 + 1);
                        }

                        const checkWall = (nx, ny, va, vb) => {
                            const other = (nx < 0 || nx >= resW || ny < 0 || ny >= resH) ? -1 : gridIndices[ny * resW + nx];
                            if (other === -1) {
                                indices.push(va * 2, va * 2 + 1, vb * 2 + 1, va * 2, vb * 2 + 1, vb * 2);
                            }
                        };

                        if (i10 !== -1) checkWall(x, y - 1, i10, i00);
                        if (i10 !== -1) checkWall(x, y + 1, i00, i10);
                        if (i01 !== -1) checkWall(x - 1, y, i00, i01);
                        if (i01 !== -1) checkWall(x + 1, y, i01, i00);
                    }
                }
                
                geometry.setAttribute('position', new THREE.Float32BufferAttribute(vertices, 3));
                geometry.setIndex(indices);
                geometry.computeVertexNormals();

                if (mesh) { scene.remove(mesh); mesh.geometry.dispose(); mesh.material.dispose(); }
                mesh = new THREE.Mesh(geometry, new THREE.MeshPhongMaterial({ color: 0xffffff, side: THREE.DoubleSide, shininess: 30 }));
                scene.add(mesh);
                document.getElementById('export-btn').disabled = false;
                showToast("Solid Generated");
            } catch (err) { console.error(err); } finally { setTimeout(() => loading.classList.add('hidden'), 300); }
        }

        function exportSTL() {
            if (!mesh) return;
            const exporter = new THREE.STLExporter();
            const binaryData = exporter.parse(mesh, { binary: true });
            const blob = new Blob([binaryData], { type: 'application/octet-stream' });
            const url = URL.createObjectURL(blob);
            const link = document.createElement('a');
            link.href = url;
            link.download = `LithoForge_Manifold_${Date.now()}.stl`;
            document.body.appendChild(link);
            link.click();
            setTimeout(() => { document.body.removeChild(link); window.URL.revokeObjectURL(url); }, 2000);
        }

        function showToast(msg) {
            const toast = document.getElementById('toast');
            const toastText = document.getElementById('toast-text');
            if (!toast || !toastText) return;
            toastText.innerText = msg;
            toast.style.opacity = '1';
            setTimeout(() => { toast.style.opacity = '0'; }, 3000);
        }

        function setupUI() {
            initEngine();
            const el = {
                imageInput: document.getElementById('image-input'),
                dropZone: document.getElementById('drop-zone'),
                maxThick: document.getElementById('max-thick'),
                maxThickVal: document.getElementById('max-thick-val'),
                baseThick: document.getElementById('base-thick'),
                baseThickVal: document.getElementById('base-thick-val'),
                smoothSlider: document.getElementById('smooth-slider'),
                smoothVal: document.getElementById('smooth-val'),
                resSlider: document.getElementById('res-slider'),
                resVal: document.getElementById('res-val'),
                curveRadius: document.getElementById('curve-radius'),
                curveRadiusVal: document.getElementById('curve-radius-val'),
                modelWidth: document.getElementById('model-width'),
                modelHeight: document.getElementById('model-height'),
                renderBtn: document.getElementById('render-btn'),
                exportBtn: document.getElementById('export-btn'),
                curveControl: document.getElementById('curve-control')
            };

            el.imageInput.addEventListener('change', (e) => handleFile(e.target.files[0]));
            el.dropZone.addEventListener('dragover', (e) => { e.preventDefault(); el.dropZone.classList.add('drag-over'); });
            el.dropZone.addEventListener('dragleave', () => el.dropZone.classList.remove('drag-over'));
            el.dropZone.addEventListener('drop', (e) => { e.preventDefault(); el.dropZone.classList.remove('drag-over'); handleFile(e.dataTransfer.files[0]); });

            document.querySelectorAll('.shape-btn').forEach(btn => {
                btn.onclick = () => {
                    document.querySelectorAll('.shape-btn').forEach(b => b.classList.remove('active'));
                    btn.classList.add('active');
                    state.shape = btn.dataset.shape;
                    if (state.shape === 'Curved') el.curveControl.classList.remove('hidden');
                    else el.curveControl.classList.add('hidden');
                };
            });

            el.maxThick.oninput = (e) => { state.maxThickness = parseFloat(e.target.value); el.maxThickVal.innerText = state.maxThickness.toFixed(1) + 'mm'; };
            el.baseThick.oninput = (e) => { state.baseThickness = parseFloat(e.target.value); el.baseThickVal.innerText = state.baseThickness.toFixed(1) + 'mm'; };
            el.curveRadius.oninput = (e) => { state.curveRadius = parseFloat(e.target.value); el.curveRadiusVal.innerText = state.curveRadius + 'mm'; };
            el.smoothSlider.oninput = (e) => { state.smoothing = parseFloat(e.target.value); el.smoothVal.innerText = state.smoothing.toFixed(1) + 'px'; };
            el.resSlider.oninput = (e) => { state.resolution = parseInt(e.target.value); el.resVal.innerText = state.resolution + 'px'; };
            el.modelWidth.onchange = (e) => state.width = parseFloat(e.target.value) || 100;
            el.modelHeight.onchange = (e) => state.height = parseFloat(e.target.value) || 100;
            el.renderBtn.onclick = processImage;
            el.exportBtn.onclick = exportSTL;
        }

        window.addEventListener('load', setupUI);
    </script>
</body>
</html>
