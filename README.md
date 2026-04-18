<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>LithoForge Ultra HD - Compact Solid Maker</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/exporters/STLExporter.js"></script>

    <style>
        :root { --sidebar-w: 240px; --header-h: 44px; }
        body { background-color: #050508; color: #e2e8f0; font-family: system-ui, sans-serif; height: 100dvh; display: flex; flex-direction: column; overflow: hidden; margin: 0; }
        .glass { background: rgba(10, 10, 18, 0.98); backdrop-filter: blur(10px); border-bottom: 1px solid rgba(255, 255, 255, 0.08); }
        input[type=range] { accent-color: #6366f1; cursor: pointer; height: 4px; width: 100%; margin: 0; }
        .shape-btn.active { background-color: #4f46e5; border-color: #818cf8; color: white; }
        .toggle-btn.active { background-color: #10b981; border-color: #34d399; color: white; }
        .control-label { font-size: 8px; font-weight: 800; color: #71717a; text-transform: uppercase; letter-spacing: 0.02em; display: flex; justify-content: space-between; line-height: 1; margin-bottom: 2px; }
        .value-badge { font-family: monospace; color: #a5b4fc; font-size: 8px; }
        .setting-card { background: rgba(255, 255, 255, 0.02); border-radius: 6px; padding: 4px 6px; border: 1px solid rgba(255, 255, 255, 0.03); }
        #canvas-container canvas { display: block; width: 100% !important; height: 100% !important; }
        #drop-zone.drag-over { border-color: #6366f1; background: rgba(99, 102, 241, 0.1); }
    </style>
</head>
<body>

    <header class="h-[var(--header-h)] px-4 glass z-40 flex items-center justify-between shrink-0">
        <div class="flex items-center gap-2">
            <i class="fas fa-lightbulb text-indigo-500 text-base"></i>
            <h1 class="text-[10px] font-black tracking-tighter text-white uppercase italic">LithoForge <span class="text-indigo-400 font-light">Ultra HD</span></h1>
        </div>
        
        <div class="flex items-center gap-2">
            <button id="render-btn" disabled class="bg-zinc-100 hover:bg-white disabled:opacity-10 text-black px-3 py-1.5 rounded transition-all font-black text-[9px] uppercase tracking-wider active:scale-95">
                <i class="fas fa-hammer mr-1"></i> Generate
            </button>
            <button id="export-btn" disabled class="bg-indigo-600 hover:bg-indigo-500 disabled:opacity-10 text-white px-3 py-1.5 rounded transition-all font-black text-[9px] uppercase tracking-wider active:scale-95">
                <i class="fas fa-save mr-1"></i> Export
            </button>
        </div>
    </header>

    <main class="flex flex-1 overflow-hidden relative">
        <aside class="w-[var(--sidebar-w)] bg-[#08080c] border-r border-white/5 flex flex-col h-full z-20 shrink-0 p-2 gap-2 overflow-hidden">
            
            <!-- 1. Source Image (Micro) -->
            <label id="drop-zone" for="image-input" class="relative border border-dashed border-zinc-800 hover:border-indigo-500/50 bg-zinc-900/20 rounded-lg p-1.5 transition-all flex flex-col items-center justify-center cursor-pointer h-16 shrink-0 group">
                <div id="upload-placeholder" class="flex flex-col items-center pointer-events-none">
                    <i class="fas fa-cloud-upload-alt text-lg text-zinc-700 group-hover:text-indigo-500"></i>
                    <span class="text-[8px] text-zinc-500 font-bold uppercase">WTIU PNG / JPG</span>
                </div>
                <img id="img-preview" class="hidden w-full h-full object-contain rounded" alt="Preview">
                <input id="image-input" type="file" class="hidden" accept="image/*">
            </label>

            <!-- 2. Shape Buttons & Toggle -->
            <div class="flex flex-col gap-1 shrink-0">
                <div class="grid grid-cols-3 gap-1">
                    <button data-shape="Alpha" class="shape-btn active py-1 rounded bg-zinc-900 text-[8px] font-black uppercase border border-zinc-800 text-zinc-500">Edge</button>
                    <button data-shape="Rectangle" class="shape-btn py-1 rounded bg-zinc-900 text-[8px] font-black uppercase border border-zinc-800 text-zinc-500">Rect</button>
                    <button data-shape="Curved" class="shape-btn py-1 rounded bg-zinc-900 text-[8px] font-black uppercase border border-zinc-800 text-zinc-500">Curve</button>
                </div>
                <button id="nightlight-toggle" class="toggle-btn w-full py-1 rounded bg-zinc-900 text-[8px] font-black uppercase border border-zinc-800 text-zinc-500 transition-colors">
                    <i class="fas fa-plug mr-1"></i> Add Nightlight Mount
                </button>
            </div>

            <!-- 3. Mesh Grid (Compact) -->
            <div class="flex flex-col gap-1.5 overflow-hidden">
                <div class="grid grid-cols-2 gap-1.5">
                    <div class="setting-card">
                        <div class="control-label">Base <span id="base-thick-val" class="value-badge">1.0mm</span></div>
                        <input id="base-thick" type="range" min="0.4" max="4" step="0.1" value="1.0">
                    </div>
                    <div class="setting-card">
                        <div class="control-label">Litho <span id="max-thick-val" class="value-badge">3.0mm</span></div>
                        <input id="max-thick" type="range" min="1" max="10" step="0.1" value="3.0">
                    </div>
                </div>

                <div id="curve-control" class="hidden setting-card border-indigo-500/40">
                    <div class="control-label">Radius <span id="curve-radius-val" class="value-badge">100mm</span></div>
                    <input id="curve-radius" type="range" min="20" max="500" step="1" value="100">
                </div>

                <div class="grid grid-cols-2 gap-1.5">
                    <div class="setting-card">
                        <div class="control-label">Smooth <span id="smooth-val" class="value-badge">1.5px</span></div>
                        <input id="smooth-slider" type="range" min="0" max="10" step="0.5" value="1.5">
                    </div>
                    <div class="setting-card">
                        <div class="control-label">Detail <span id="res-val" class="value-badge">200px</span></div>
                        <input id="res-slider" type="range" min="50" max="400" step="10" value="200">
                    </div>
                </div>

                <div class="grid grid-cols-2 gap-1.5">
                    <div class="setting-card">
                        <span class="control-label italic">Width (mm)</span>
                        <input id="model-width" type="number" value="100" class="w-full bg-black border border-zinc-800 rounded px-1 py-0.5 text-[9px] text-indigo-300 outline-none">
                    </div>
                    <div class="setting-card">
                        <span class="control-label italic">Height (mm)</span>
                        <input id="model-height" type="number" value="100" class="w-full bg-black border border-zinc-800 rounded px-1 py-0.5 text-[9px] text-indigo-300 outline-none">
                    </div>
                </div>
            </div>

            <div class="mt-auto border-t border-white/5 pt-1 opacity-20 text-[7px] text-center uppercase tracking-tighter">WTIU Nightlight Engine</div>
        </aside>

        <!-- Viewport Area -->
        <div class="flex-1 relative bg-[#050508]">
            <div id="canvas-container" class="w-full h-full"></div>
            
            <div id="loading-overlay" class="hidden absolute inset-0 bg-black/90 backdrop-blur-sm flex items-center justify-center z-50">
                <div class="flex flex-col items-center gap-2">
                    <div class="w-6 h-6 border-2 border-indigo-500/20 border-t-indigo-500 rounded-full animate-spin"></div>
                    <p class="text-white text-[8px] font-bold uppercase tracking-widest">Processing...</p>
                </div>
            </div>

            <div id="toast" class="absolute bottom-2 left-1/2 -translate-x-1/2 px-3 py-1 bg-zinc-900 border border-zinc-800 rounded text-[7px] font-black text-indigo-400 tracking-widest shadow-2xl opacity-0 transition-opacity uppercase pointer-events-none">
                <span id="toast-text">Ready</span>
            </div>
        </div>
    </main>

    <canvas id="hidden-canvas" class="hidden"></canvas>

    <script>
        let state = {
            image: null,
            shape: 'Alpha',
            nightlight: false,
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
                setTimeout(initEngine, 300);
                return;
            }
            scene = new THREE.Scene();
            camera = new THREE.PerspectiveCamera(40, container.clientWidth / container.clientHeight, 0.1, 10000);
            camera.position.set(0, -250, 200);
            
            renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
            renderer.setSize(container.clientWidth, container.clientHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
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
                if (renderer) renderer.render(scene, camera);
            }
            animate();
            
            window.addEventListener('resize', () => {
                const w = container.clientWidth;
                const h = container.clientHeight;
                camera.aspect = w / h;
                camera.updateProjectionMatrix();
                renderer.setSize(w, h);
            });
        }

        function handleFile(file) {
            if (!file || !file.type.startsWith('image/')) return;
            const reader = new FileReader();
            reader.onload = (ev) => {
                state.image = ev.target.result;
                document.getElementById('img-preview').src = ev.target.result;
                document.getElementById('img-preview').classList.remove('hidden');
                document.getElementById('upload-placeholder').classList.add('hidden');
                document.getElementById('render-btn').disabled = false;
                showToast("Image Traced");
            };
            reader.readAsDataURL(file);
        }

        async function processImage() {
            if (!state.image) return;
            const loading = document.getElementById('loading-overlay');
            loading.classList.remove('hidden');
            await new Promise(r => setTimeout(r, 50));

            try {
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

                // Function to handle position calculation including Curved warping
                const calcPos = (uu, vv, zz) => {
                    if (state.shape === 'Curved') {
                        const r = state.curveRadius + zz;
                        const theta = (uu - 0.5) * (state.width / state.curveRadius);
                        return [r * Math.sin(theta), (vv - 0.5) * state.height, r * Math.cos(theta) - state.curveRadius];
                    }
                    return [(uu - 0.5) * state.width, (vv - 0.5) * state.height, zz];
                };

                // 1. Generate Lithophane Vertices
                for (let y = 0; y < resH; y++) {
                    for (let x = 0; x < resW; x++) {
                        if (isOpaque(x, y)) {
                            const u = x / (resW - 1);
                            const v = 1 - (y / (resH - 1));
                            const idx = (y * resW + x) * 4;
                            const bri = (0.299 * pixels[idx] + 0.587 * pixels[idx+1] + 0.114 * pixels[idx+2]) / 255;
                            const frontZ = (1 - bri) * state.maxThickness + state.baseThickness;
                            const backZ = 0;

                            const fp = calcPos(u, v, frontZ);
                            const bp = calcPos(u, v, backZ);
                            gridIndices[y * resW + x] = vCount;
                            vertices.push(...fp, ...bp);
                            vCount++;
                        }
                    }
                }

                // 2. Stitch Lithophane Faces
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

                        // Walls
                        const checkWall = (nx, ny, va, vb) => {
                            if ((nx < 0 || nx >= resW || ny < 0 || ny >= resH) || gridIndices[ny * resW + nx] === -1) {
                                indices.push(va * 2, va * 2 + 1, vb * 2 + 1, va * 2, vb * 2 + 1, vb * 2);
                            }
                        };
                        if (i10 !== -1) { checkWall(x, y - 1, i10, i00); checkWall(x, y + 1, i00, i10); }
                        if (i01 !== -1) { checkWall(x - 1, y, i00, i01); checkWall(x + 1, y, i01, i00); }
                    }
                }

                // 3. Optional Nightlight Mount (25mm wide, 15mm deep)
                if (state.nightlight) {
                    const mountW = 25;
                    const mountD = 15;
                    const mountT = 2.5; // Thickness of the mount
                    const startV = vCount;

                    // Mount position relative to the bottom center
                    const centerU = 0.5;
                    const bottomV = 0.0; // bottom of the model
                    
                    // Calculation for mount vertices
                    const corners = [
                        [-mountW/2, 0], [mountW/2, 0], 
                        [mountW/2, -mountD], [-mountW/2, -mountD]
                    ];

                    // Generate vertices for the mounting tab (top and bottom faces)
                    corners.forEach(c => {
                        const [lx, ly] = c;
                        // Transform relative coordinates into model space
                        const u_off = centerU + (lx / state.width);
                        const v_off = bottomV + (ly / state.height);
                        
                        const pTop = calcPos(u_off, v_off, mountT);
                        const pBottom = calcPos(u_off, v_off, 0);
                        
                        vertices.push(...pTop, ...pBottom);
                    });

                    // Stitch mount faces (indices are relative to startV)
                    // Top (0, 1, 2, 3), Bottom (4, 5, 6, 7) - each multiplied by 2 for the offset
                    const m = startV * 2;
                    indices.push(m, m+2, m+4, m, m+4, m+6); // Top face
                    indices.push(m+1, m+5, m+3, m+1, m+7, m+5); // Bottom face
                    
                    // Side walls for the mount
                    const pairs = [[0,1], [1,2], [2,3], [3,0]];
                    pairs.forEach(p => {
                        const a = m + p[0]*2;
                        const b = m + p[1]*2;
                        indices.push(a, a+1, b+1, a, b+1, b);
                    });
                }
                
                geometry.setAttribute('position', new THREE.Float32BufferAttribute(vertices, 3));
                geometry.setIndex(indices);
                geometry.computeVertexNormals();
                if (mesh) { scene.remove(mesh); mesh.geometry.dispose(); mesh.material.dispose(); }
                mesh = new THREE.Mesh(geometry, new THREE.MeshPhongMaterial({ color: 0xffffff, side: THREE.DoubleSide, shininess: 30 }));
                scene.add(mesh);
                document.getElementById('export-btn').disabled = false;
                showToast("Solid Generated");
            } catch (err) { console.error(err); } finally { loading.classList.add('hidden'); }
        }

        function exportSTL() {
            if (!mesh) return;
            const exporter = new THREE.STLExporter();
            const blob = new Blob([exporter.parse(mesh, { binary: true })], { type: 'application/octet-stream' });
            const url = URL.createObjectURL(blob);
            const link = document.createElement('a');
            link.href = url;
            link.download = `WTIU_Nightlight_${Date.now()}.stl`;
            link.click();
            URL.revokeObjectURL(url);
        }

        function showToast(msg) {
            const toast = document.getElementById('toast');
            document.getElementById('toast-text').innerText = msg;
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
                curveControl: document.getElementById('curve-control'),
                nightlightBtn: document.getElementById('nightlight-toggle')
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

            el.nightlightBtn.onclick = () => {
                state.nightlight = !state.nightlight;
                el.nightlightBtn.classList.toggle('active', state.nightlight);
                showToast(state.nightlight ? "Nightlight Mount Enabled" : "Mount Disabled");
            };

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
