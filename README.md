# NEXUMI — AI Desktop Companion

### Technical Documentation | Version 1.0

## Overview

Nexumi is an AI-powered desktop companion built for Arch Linux. It combines voice interaction, desktop automation, memory management, and AI conversation into a single interactive assistant.

The assistant runs as a draggable, always-on-top 3D desktop character that can respond to voice commands, automate tasks, remember information, perform web searches, and interact naturally with the user.

Nexumi is designed to feel lightweight, responsive, and practical for everyday desktop use while still offering advanced AI-driven functionality.

---

# 1. System Requirements

| Requirement         | Specification                                |
| ------------------- | -------------------------------------------- |
| Operating System    | Arch Linux (X11 or Wayland)                  |
| RAM                 | Minimum 8 GB (zRAM recommended)              |
| Python              | Version 3.10 or newer                        |
| GPU                 | Optional — CPU inference supported           |
| Display Environment | KDE, XFCE, i3, or any compatible WM/DE       |
| Audio               | ALSA or PulseAudio with a working microphone |

---

# 2. Installation

## 2.1 Install System Dependencies

Install the required packages before setting up the Python environment.

```bash
sudo pacman -S python python-pip portaudio alsa-utils ffmpeg xdotool
sudo pacman -S python-pyqt5 python-pyqt5-webengine qt5-webengine

# Optional 3D renderer
# yay -S hime-display
```

---

## 2.2 Project Setup

Clone the repository and initialize the environment.

```bash
git clone https://github.com/notprakash-git/nexumi
cd ~/nexumi

chmod +x setup.sh start_nexumi.sh stop_nexumi.sh

# One-time setup
./setup.sh
```

The setup script automatically:

* Creates a Python virtual environment
* Installs required Python packages
* Downloads the Vosk speech model
* Configures the base project structure

---

## 2.3 Configure API Keys

Copy the example environment file and add your credentials.

```bash
cp config/api_keys.env.example config/api_keys.env
nano config/api_keys.env
```

Required keys:

* `OPENROUTER_API_KEY` — available from OpenRouter
* `PICOVOICE_ACCESS_KEY` — available from Picovoice Console

---

## 2.4 Add a VRM Character Model

Place a `.vrm` character model inside:

```text
assets/3d_model/nexumi.vrm
```

The free Hayakawa Aoi model from VRoid Hub works correctly with Nexumi.

If no model is installed, the application automatically falls back to a placeholder model.

---

## 2.5 Configure the Wake Word

Nexumi uses Porcupine wake word detection.

Steps:

1. Create an account on Picovoice Console
2. Train a custom wake word using the phrase:

```text
nexumi
```

3. Download the Linux `.ppn` file
4. Place the file here:

```text
assets/nexumi_en_linux.ppn
```

5. Add your `PICOVOICE_ACCESS_KEY` inside:

```text
config/api_keys.env
```

---

# 3. Usage

## 3.1 Starting and Stopping Nexumi

```bash
cd ~/nexumi

# Start Nexumi
./start_nexumi.sh

# Stop Nexumi
./stop_nexumi.sh
```

---

### Manual Startup (Debugging)

```bash
source venv/bin/activate

python -m src.ui.websocket_server
python -m src.core.wakeword
python -m src.ui.desktop_mate
```

---

## 3.2 Voice Commands

After the wake word is detected, Nexumi listens for commands.

| Voice Command                        | Action                       |
| ------------------------------------ | ---------------------------- |
| “Nexumi, search for [query]”         | Performs a web search        |
| “Nexumi, click on [target]”          | Clicks a UI element          |
| “Nexumi, type [text]”                | Types text at the cursor     |
| “Nexumi, scroll down/up”             | Scrolls the active window    |
| “Nexumi, open [application]”         | Launches an application      |
| “Nexumi, take a screenshot”          | Saves a screenshot           |
| “Nexumi, remember that [fact]”       | Stores information in memory |
| “Nexumi, what do you know about me?” | Retrieves saved memory       |

---

## 3.3 Chat Panel

Right-click the character window and select:

```text
Open Chat
```

The chat panel allows text-based interaction using the same AI processing pipeline as voice input.

Conversation history is stored at:

```text
data/chat_history.json
```

---

## 3.4 Memory Panel

Right-click the character window and select:

```text
Memory / Data
```

From here you can:

* View saved memories
* Add new memory entries
* Delete stored data

Long-term memory is stored in:

```text
data/memory.db
```

---

# 4. Configuration

## 4.1 settings.json

Main configuration file:

```text
config/settings.json
```

| Field                   | Description               |
| ----------------------- | ------------------------- |
| `language`              | Interaction language      |
| `mood`                  | Personality mode          |
| `window.width / height` | Window dimensions         |
| `window.always_on_top`  | Keeps window above others |
| `wake_word.sensitivity` | Detection sensitivity     |
| `stt.vosk_model_path`   | Offline speech model path |
| `memory.short_term_k`   | Conversation memory size  |
| `websocket.port`        | Local WebSocket port      |

---

## 4.2 character.json

Defines Nexumi’s personality and conversational style.

Location:

```text
config/character.json
```

This file controls:

* Personality behavior
* Speech style
* Catchphrases
* Likes and dislikes
* System prompt customization

---

# 5. Architecture

## 5.1 Core Technology Stack

| Layer               | Technology                        |
| ------------------- | --------------------------------- |
| Wake Word Detection | Porcupine                         |
| Speech-to-Text      | Google Web Speech / Vosk          |
| AI Model            | Google Gemma 4 31B via OpenRouter |
| Long-Term Memory    | SQLite                            |
| Short-Term Memory   | LangChain                         |
| Text-to-Speech      | Chatterbox TTS                    |
| Desktop Automation  | PyAutoGUI, xdotool                |
| Web Search          | DuckDuckGo                        |
| 3D Rendering        | Hime Display / Three.js           |
| GUI Framework       | PyQt5                             |
| IPC Communication   | WebSocket                         |

---

## 5.2 Request Processing Pipeline

Every interaction follows a structured processing pipeline:

1. Wake word detection activates speech recognition
2. Speech is converted into text
3. The intent router classifies the request
4. The correct handler executes the task
5. The AI generates a response
6. Emotion analysis selects an expression state
7. TTS generates speech output
8. The 3D character updates its expression
9. UI components synchronize through WebSocket communication

---

## 5.3 Project Structure

| Path               | Purpose                          |
| ------------------ | -------------------------------- |
| `config/`          | Settings and API keys            |
| `assets/3d_model/` | VRM character models             |
| `src/core/`        | AI brain, STT, TTS, wake word    |
| `src/tools/`       | Automation and utility tools     |
| `src/integration/` | Emotion and renderer integration |
| `src/ui/`          | PyQt5 interface components       |
| `src/utils/`       | Logging and resource monitoring  |
| `data/`            | Chat history and memory database |
| `web/`             | Three.js web renderer            |
| `tests/`           | Test scripts                     |
| `logs/`            | Application logs                 |

---

## 5.4 WebSocket Communication

Nexumi uses JSON-based WebSocket messaging on:

```text
ws://localhost:8765
```

| Message Type         | Purpose                       |
| -------------------- | ----------------------------- |
| `wake_word_detected` | Starts STT recording          |
| `user_text`          | Sends user input              |
| `ai_message`         | Broadcasts AI responses       |
| `status`             | Updates application status    |
| `emotion`            | Updates character expressions |
| `menu_action`        | Changes personality modes     |
| `resource_warning`   | Sends system resource alerts  |

---

# 6. Testing

Activate the environment first:

```bash
source venv/bin/activate
```

Run integration tests:

```bash
python tests/test_integration.py
```

Run individual tests:

```bash
python tests/test_brain.py
python tests/test_expression.py
```

---

## Available Test Modules

| Test Script           | Coverage                        |
| --------------------- | ------------------------------- |
| `test_brain.py`       | AI response and memory handling |
| `test_tts.py`         | Speech synthesis                |
| `test_wakeword.py`    | Wake word validation            |
| `test_hime.py`        | VRM expression handling         |
| `test_expression.py`  | Emotion classification          |
| `test_integration.py` | Full pipeline integration       |

---

# 7. Troubleshooting

| Issue                  | Solution                                |
| ---------------------- | --------------------------------------- |
| No audio output        | Verify PulseAudio/ALSA settings         |
| Wake word not detected | Increase sensitivity in `settings.json` |
| LLM not responding     | Check API key and internet connection   |
| VRM model missing      | Verify `nexumi.vrm` exists              |
| High RAM usage         | Enable zRAM support                     |
| Vosk not working       | Re-download the speech model            |

---

# 8. License

Nexumi is released under the MIT License.

You are free to:

* Use the software
* Modify the source code
* Distribute the project
* Build upon the framework

Third-party technologies such as Porcupine, OpenRouter, Chatterbox TTS, and three-vrm remain subject to their own licenses.

VRM models downloaded from VRoid Hub are also subject to their respective creator terms and usage policies.

---

### Nexumi v1.0

AI Desktop Companion for Arch Linux
