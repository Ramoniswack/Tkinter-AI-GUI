HIT137 - Tkinter AI GUI

Overview

This repository contains a small desktop GUI built with Tkinter that demonstrates several Hugging Face model workflows. The app provides four main capabilities: text classification, image classification, image captioning (image-to-text), and text-to-image generation. It is intended as an educational demo and a lightweight local tool for experimentation.

This README explains how to prepare an environment, run the app, and work on the code. It also describes the layout of the repository and the responsibilities of the team members listed in the project.

Scope and important notes

- The app loads model pipelines from the Hugging Face ecosystem and runs them locally. Some models are GPU-friendly and will be slow or may not run on low-memory CPUs. The code includes CPU-friendly fallbacks for some pipelines, but performance varies.
- Several adapters in the project intentionally disable the diffusers safety checker to avoid blank outputs during development. Do not expose unfiltered outputs in publicly accessible services without proper moderation.

Requirements

This guide assumes a Linux environment and a zsh shell. Commands are copy-paste ready for that setup.

- Python 3.10 or newer is recommended. The code was developed with Python 3.12 but should work with recent 3.x versions.
- Git
- A modern CPU. For better image generation performance, a CUDA GPU with recent drivers and an appropriate PyTorch build will help.

Quick start

1. Clone the repository (if you do not already have it locally)

```bash
git clone <your-repo-url> tk-hf-gui
cd tk-hf-gui
```

2. Create and activate a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

3. Install Python dependencies

Follow the commands that match your hardware. These instructions assume a Linux system and the zsh shell. Run one of the PyTorch install lines below first, then install the remaining Python packages.

Upgrade pip and install wheel tools

```bash
python3 -m pip install --upgrade pip setuptools wheel
```

CPU only installation (no CUDA)

```bash
python3 -m pip install --upgrade pip
python3 -m pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
```

CUDA GPU installation examples

Pick the line that matches your CUDA driver. If you are not sure, consult https://pytorch.org/get-started/locally for the command tailored to your system.

CUDA 11.8 example

```bash
python3 -m pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

CUDA 12.1 example

```bash
python3 -m pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

Install the remaining Python packages required by this project

```bash
python3 -m pip install transformers diffusers accelerate safetensors pillow imageio imageio-ffmpeg
# optional: xformers can speed up some pipelines if available for your Python and CUDA versions
# python3 -m pip install xformers
```

Optional: install from the requirements file (this will install the packages listed there)

```bash
python3 -m pip install -r requirements.txt
```

System packages you may need

```bash
# ffmpeg is required for video encoding and decoding
sudo apt update
sudo apt install -y ffmpeg git-lfs
# after installing git-lfs, run
git lfs install
```

If you will access private Hugging Face model weights, set your token

```bash
export HUGGINGFACE_HUB_TOKEN="<your-token-here>"
```

Disk and cache notes

Model weights are downloaded into your user cache by default. If you need to relocate or manage the cache, consult Hugging Face documentation.

Notes about optional packages

- For faster memory usage and speed on compatible systems, you may want to install xformers or other optional accelerators. Install them only if your environment supports it.
- If you plan to use GPU acceleration, install a PyTorch build that matches your CUDA driver. See https://pytorch.org for instructions.

Run the application

```bash
source .venv/bin/activate
python3 app.py
```

The Tkinter window will appear. Use the top Model dropdown to choose a model, click Load Model and then use the input panel to supply a prompt or an image. Click Run Selected Model to execute the selected model. Generated images and media are saved into the `assets` folder.

Project layout and explanation

Top level files

- `app.py` - main application entry point and UI wiring. It builds the Tkinter window, the layout, and manages model loading and background runs.
- `requirements.txt` - Python dependencies used by the project.
- `README.md` - this file.

Folders

- `assets/` - generated images and example media are stored here. The app saves generated files to this folder.
- `docs/` - notes and helper files. See model-specific files inside for additional context.
- `models/` - adapter classes for each model category. Each adapter encapsulates loading and running a particular pipeline. Key files:
  - `text_to_image.py` - text-to-image adapter (Stable Diffusion variants). CPU-friendly defaults are provided; there are also higher-quality presets in the code comments.
  - `image_to_text.py` - image captioning adapter (BLIP).
  - `image_classifier.py` - image classification adapter.
  - `text_sentiment.py` - text classification adapter.
  - `text_to_video.py` - text-to-video adapter. This file includes a GPU path that uses SDXL and Stable Video Diffusion and a CPU fallback.
  - `base.py` - base adapter class shared by other adapters.
- `ui/` - Tkinter UI components split into files for better structure. Key files:
  - `input_frame.py` - user input widgets and logic to gather prompts and image paths.
  - `output_frame.py` - displays model output and previews generated images or video thumbnails.
  - `info_frame.py` - model information and static help text.
  - `preferences.py` - lightweight preferences dialog used by the app.
- `utils/` - reusable helper code
  - `decorators.py` - simple logging and timing decorators used by adapters.
  - `theme.py` - styling for the UI (colors and ttk styles).

How the adapters and UI work together

- The GUI calls `adapter.load()` to instantiate and prepare a pipeline. Loading may be slow on first run because model weights are fetched from the Hugging Face hub.
- The GUI gathers input from `InputFrame.get_payload()` and normalizes it to a simple string where appropriate. The adapter `run()` method receives the normalized input and returns a dictionary with at least a `result` string and optional preview keys such as `image_path` or `video_path`.
- The `OutputFrame` shows textual results and renders a preview of the media returned by adapters.

Team and roles

Project members

- milansapkota16-lang
- bishalnayabha0
- lovesonpok
- suman2120

Suggested roles and responsibilities

- lovesonpok - Project lead and lead backend and performance engineer. He is the main point of contact for architecture decisions and backend implementation. Responsibilities: lead the overall design, implement and optimize model adapters, tune performance for CPU and GPU, manage optional acceleration packages, and coordinate integration testing on target hardware.
- milansapkota16-lang - Developer and integrator. Responsibilities: work with the lead on adapter design, implement features across adapters, and help with integration and testing.
- bishalnayabha0 - UI and user experience lead. Responsibilities: Tkinter layout, input/output widgets, preview behavior, accessibility, and theme tuning.
- suman2120 - Documentation and testing lead. Responsibilities: README and docs, local setup instructions, troubleshooting guides, and writing small unit or smoke tests to catch regressions.

How the team works together

- Feature design and task assignment: team members discuss a feature or issue, then a single member implements the first version and opens a short description in the project issue tracker or a local task list.
- Review cycle: another team member reviews the change for correctness, style, and UI/UX. The reviewer runs the code locally and provides feedback. Small suggested edits are applied directly.
- Integration and testing: the integrator runs the app on a target machine, verifies behavior for CPU and GPU if available, and finalizes the change.
- Documentation and hand-off: the documentation lead updates the README or inline docs describing any configuration changes or new optional dependencies.

Development notes and tips

- If you want higher quality images, try using SDXL models or increase `num_inference_steps` and `guidance_scale` in `text_to_image.py`. For higher-quality results you may need a GPU with enough VRAM.
- If you use GPU acceleration, make sure the installed PyTorch build matches your CUDA drivers. Install a matching `torch` version before installing other packages where possible.
- The project uses simple decorators for timing and logging in `utils/decorators.py`. These are helpful for quick profiling and for showing how long a call took.

Common troubleshooting

- If you see errors about missing packages, check `requirements.txt` and install missing dependencies in the virtual environment.
- If a model load fails due to memory, try smaller image sizes in the adapter or use the CPU fallback where implemented.
- If image preview stays on screen after switching models, use the Clear button or restart the app. The app now clears the preview when you click Load Model, but behavior may vary if an in-progress operation was running.

Extending the project

- Add model options to the UI so users can control steps, guidance scale, negative prompts, seeds, and image size without editing code.
- Add a gallery view of previously generated assets in `assets`.
- Add automated tests for adapter output shapes and for small smoke tests of UI initialization.

License and ethical notice

This repository includes code that uses Hugging Face model weights. Respect the licenses associated with the models you are using. Some adapters in the code disable safety filters for development. This is not recommended for public or user-facing deployment.

Contact and support

If you need help reproducing the environment, contact the project authors listed in the Team and Roles section. Provide a short description of the issue, the output from running the app, and any tracebacks you see.

End of README
