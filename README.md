# Touchdown

**A dive from orbit to your character in close-up.**

Touchdown makes an "Earth Zoom" shot: one continuous camera move that starts in low Earth orbit, drops through the cloud deck, swoops over the landscape and settles on a waist-up shot of a character standing in a scene you describe. It runs locally on ComfyUI with open models from Black Forest Labs and Lightricks:

- **FLUX.2 [klein] 9B** paints the stills: the closing shot of your character in their scene, and optionally an orbit view.
- **LTX 2.5** turns them into the video.

The pipeline is driven from Python through the [Comfy SDK](https://platform.comfy.org) (`comfy-sdk`): each step builds a ComfyUI workflow graph, submits it, follows progress and downloads the outputs. Any ComfyUI that has the models can run it, local or cloud.

This is an entry for the Comfy Dev Platform Challenge (October 2026). It is work in progress.

## How it works

1. **The end still.** klein paints the final frame: the character (from a text description, or from a reference photo of a person) standing in the named place, at eye level.
2. **The dive.** LTX 2.5 image-to-video generates the camera move. Three modes are being compared:
   - `reverse`: LTX starts on the end still and pulls back up to orbit, and the clip is played backwards. The sharpest, most coherent frame is generated first, which is why creators use this trick.
   - `first-last`: klein also paints an orbit still, and LTX pins orbit at the first frame and the character at the last.
   - `end-only`: only the end still is pinned; the prompt alone sets up the orbit opening.
3. **Finish.** ffmpeg reverses the clip where needed. A Real-ESRGAN pass can then upscale to 4K.

A Wan 2.2 image-to-video run with an Earth zoom-out camera LoRA is kept as a comparison.

## Grounding in real places (optional, next step)

Generated Earth zooms invent the terrain under the camera. The companion project [geoblock-scenecompiler](https://github.com/jamiejk/geoblock-scenecompiler) builds a measured 3D scene for a real location instead: USGS 3DEP terrain, NAIP and HLS imagery, Overture building footprints and lidar tree heights, a Blue Marble globe with VIIRS clouds, and a Blender camera path. That blocking video then guides LTX 2.5 through its Layout To Render IC-LoRA, so the city and hills the camera passes are the real ones.

The plan is to get the generative dive looking right first, then add measured data back step by step, keeping only what improves the shot.

## Running it

The current script lives in geoblock-scenecompiler while the two are being split:

```bash
uv run python scripts/fakeout_earth_zoom.py --stills-only                         # check the stills first
uv run python scripts/fakeout_earth_zoom.py --mode reverse --seed 1 --output-dir outputs/runs/touchdown
```

It needs a ComfyUI instance with the FLUX.2 [klein] 9B and LTX 2.5 models, reachable through `COMFY_BASE_URL`. Tested on one RTX 4090 (24 GB). Standalone setup instructions will follow here.

## Status

- The `reverse` mode gives the most convincing single continuous shot so far. A known flaw: a second Earth-like planet sometimes appears in the opening sky.
- No clip is final yet.

## Licence

GNU General Public License v3.0. See [LICENSE](LICENSE).
