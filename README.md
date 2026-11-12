<div align="center">

# 🧊 image-to-3d-product-viewer

**Turn a single product photo into an interactive 3D model, right in your browser.**

Upload an image → get a `.glb` → rotate it with your mouse → embed it on any e-commerce store.

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-CUDA-EE4C2C?logo=pytorch&logoColor=white)](https://pytorch.org/)
[![Gradio](https://img.shields.io/badge/UI-Gradio-F97316?logo=gradio&logoColor=white)](https://www.gradio.app/)
[![Model](https://img.shields.io/badge/Model-Hunyuan3D--2mini-0A66C2)](https://github.com/Tencent/Hunyuan3D-2)
[![Kaggle](https://img.shields.io/badge/Runs%20on-Kaggle%20T4-20BEFF?logo=kaggle&logoColor=white)](https://www.kaggle.com/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

<!-- Add a screenshot or GIF: -->
<!-- <img src="docs/demo.png" alt="Demo" width="860"> -->

</div>

---

## 📑 Table of Contents

- [Features](#-features)
- [How It Works](#-how-it-works)
- [Requirements](#-requirements)
- [Quick Start](#-quick-start)
- [Settings](#%EF%B8%8F-settings)
- [Best Input Photos](#-best-input-photos)
- [Use on an E-commerce Site](#-use-on-an-e-commerce-site)
- [Troubleshooting](#-troubleshooting)
- [Limitations](#%EF%B8%8F-limitations)
- [Roadmap](#%EF%B8%8F-roadmap)
- [Credits](#-credits)
- [License](#-license)

---
<img width="1277" height="640" alt="image" src="https://github.com/user-attachments/assets/65507ee7-b357-40f1-b68f-d6b2b4ba7ee2" />

## ✨ Features

| | Feature | Details |
|---|---|---|
| 🖥️ | **Minimal UI** | Full-screen 3D viewer with a slim upload panel on the right |
| ✂️ | **Auto background removal** | `rembg` with the `isnet-general-use` model |
| 🧠 | **High-quality shapes** | Hunyuan3D-2mini, about 6 GB VRAM for shape generation |
| 🎨 | **Photo-accurate color** | Your photo is projected onto the mesh, orientation matched automatically |
| 🧹 | **Clean meshes** | Floating fragments removed automatically |
| 📦 | **Web-ready export** | One-click `.glb` download, works with `<model-viewer>` and Shopify |
| 📄 | **Single file** | The whole app lives in `app.py` |

---

## 🔬 How It Works

```
 Photo ──► Background removal ──► Hunyuan3D-2mini ──► Mesh cleanup ──► Orientation + color ──► .glb + 3D viewer
          (rembg)                 (shape generation)  (trimesh)         (silhouette matching)   (Gradio Model3D)
```

| Step | What happens | Tool |
|---|---|---|
| 1. Upload | User uploads one product image | Gradio `Image` |
| 2. Cut-out | Isolates the product, crops and centers it | `rembg` |
| 3. Shape | Single image to 3D mesh | Hunyuan3D-2mini |
| 4. Cleanup | Keeps the main body, drops floating pieces | `trimesh` |
| 5. Orient + color | Matches mesh to the photo silhouette, projects photo colors | NumPy, SciPy |
| 6. View | Drag to rotate, scroll to zoom | Gradio `Model3D` |

---

## 📋 Requirements

| Item | Needed |
|---|---|
| GPU | NVIDIA, 16 GB (Kaggle or Colab T4 works) |
| Python | 3.10+ (tested on Kaggle's Python 3.13 image) |
| Disk | About 8 GB for model weights, downloaded on first run |
| Internet | On, for model download and the public Gradio link |

---

## 🚀 Quick Start

### Kaggle (recommended)

1. Create a new notebook. Set **Accelerator → GPU T4** and turn **Internet → On**.
2. Paste the contents of `app.py` into a single cell.
3. Run the cell. The first run downloads about 8 GB of weights.
4. Open the public Gradio link and upload a product image.
5. Wait 30 to 60 seconds, then drag to rotate and scroll to zoom.
6. Download the `.glb` from the right panel.

### Local (CUDA GPU)

```bash
git clone https://github.com/YOUR_USERNAME/image-to-3d-product-viewer.git
cd image-to-3d-product-viewer
python app.py
```

---

## ⚙️ Settings

Edit these at the top of `app.py`:

| Setting | Default | Effect |
|---|---|---|
| `STEPS` | `50` | Diffusion steps. 30 is faster, 50 is better |
| `OCTREE` | `384` | Mesh detail (256 / 320 / 384 / 512). Higher is sharper, but a bigger file and more VRAM |
| `SEED` | `2025` | Change it to get a different shape variant |

---

## 📸 Best Input Photos

| ✅ Do | ❌ Don't |
|---|---|
| One product, centered, filling most of the frame | Scenes with several objects |
| Plain white or light grey background | Colored or busy backgrounds |
| Front 3/4 angle, even lighting | Strong shadows, extreme top-down views |
| Solid objects: shoes, bags, bottles, mugs | Liquids, splashes, glass, transparent items |

---

## 🛒 Use on an E-commerce Site

Generate the `.glb` once, host it, and embed it with Google's `<model-viewer>`:

```html
<script type="module" src="https://ajax.googleapis.com/ajax/libs/model-viewer/4.0.0/model-viewer.min.js"></script>

<model-viewer
  src="https://YOUR-HOST/product.glb"
  alt="3D view of the product"
  camera-controls
  auto-rotate
  touch-action="pan-y"
  shadow-intensity="1"
  style="width:100%; height:500px; background:#f2f2f2; border-radius:12px;">
</model-viewer>
```

| Platform | How to add |
|---|---|
| **Shopify** | Product → Media → Add → 3D model → upload the `.glb` |
| **WooCommerce / WordPress** | Paste the snippet, or wrap it in a shortcode |
| **Wix, Squarespace, BigCommerce, Magento** | Paste the snippet into an HTML or code block |
| **Custom site** | Paste the snippet |

> 💡 Host the `.glb` on your platform's media library, GitHub Pages, or S3 / Cloudflare R2 with CORS enabled (`Access-Control-Allow-Origin: *`). Keep files under about 15 MB by using `OCTREE = 256` or `320`.

---

## 🛠️ Troubleshooting

<details>
<summary><b>numpy / <code>_slice</code> import error</b></summary>

pip upgraded numpy while the notebook was running. Restart the session, then run the cell again. Do not upgrade numpy mid-session.
</details>

<details>
<summary><b><code>Missing key(s)</code> / <code>Unexpected key(s)</code> when loading the model</b></summary>

A newer `transformers` renamed some layers. `app.py` remaps the DINOv2 keys automatically. If a different module fails, open an issue with the first 5 missing and first 5 unexpected keys.
</details>

<details>
<summary><b>CUDA out of memory</b></summary>

Set `OCTREE = 320` and `num_chunks=10000`.
</details>

<details>
<summary><b><code>rembg</code> cannot download its model</b></summary>

Turn **Internet** on in the notebook settings.
</details>

<details>
<summary><b>No public Gradio link</b></summary>

Turn **Internet** on, or use `share=False` when running locally.
</details>

<details>
<summary><b>Color looks mirrored or on the wrong side</b></summary>

Open an issue with the printed `Orientation match IoU` value and a screenshot.
</details>

---

## ⚠️ Limitations

- A single photo shows only one side. The back is guessed by the model, and the back color is a mirrored, darker copy of the photo.
- Colors are vertex colors projected from the photo, not full PBR textures. Sharper textures need Hunyuan3D-Paint (16 GB+ VRAM and a CUDA compile) on a larger GPU.
- Shiny, glass and transparent products reconstruct poorly.
- Generation takes 30 to 60 seconds on a T4.
- The Kaggle public link stops working when the session ends, so it is not a production backend.

---

## 🗺️ Roadmap

- [ ] Batch script: folder of product images → compressed `.glb` files
- [ ] FastAPI `/generate` endpoint
- [ ] Standalone three.js / `<model-viewer>` front end
- [ ] Deploy on a GPU host (RunPod, Modal, Hugging Face Spaces GPU)
- [ ] Texture baking with Hunyuan3D-Paint on a larger GPU

---

## 🙏 Credits

- [Hunyuan3D-2](https://github.com/Tencent/Hunyuan3D-2) by Tencent
- [Gradio](https://www.gradio.app/)
- [rembg](https://github.com/danielgatis/rembg)
- [trimesh](https://trimesh.org/)
- [`<model-viewer>`](https://modelviewer.dev/) by Google

---

## 📄 License

The code in this repository is released under the **MIT License** (add a `LICENSE` file).

> ⚠️ The Hunyuan3D-2 model weights have their own license from Tencent, which includes usage and territory restrictions. Read it before using generated models commercially: <https://github.com/Tencent/Hunyuan3D-2>

---

<div align="center">

If this project helps you, please ⭐ the repo.

</div>
