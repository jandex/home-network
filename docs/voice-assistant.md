# Voice assistant

Voice is a pipeline on the Mini PC. The Raspberry Pi is a microphone and a speaker. Claude is the language model inside that pipeline. Claude does not run Home Assistant.

## What was rejected

Home Assistant can expose itself to Claude over MCP, or hand Claude a broad set of tools. That was considered and dropped.

| Approach | Decision |
|----------|----------|
| MCP server, Claude with full tool access | Not used. A prompt-injected or over-broad model could change automations, not just answer. |
| Anthropic conversation agent in one Assist pipeline | Used. The model only sees entities explicitly exposed to Assist. |
| Second router "so voice is separate" | Not used. The satellite sits on VLAN 10 because it must reach Home Assistant. Isolation is the firewall, not another NAT. |

## Path of one sentence

```text
You speak (English or Filipino)
        │
        ▼
Raspberry Pi satellite          wake word + mic, Wyoming protocol
        │  VLAN 10, wired or strong Wi-Fi to the Mini PC
        ▼
Home Assistant Assist pipeline
        ├── Speech-to-text     Whisper (local) or Home Assistant Cloud
        ├── Conversation       Anthropic Claude
        │                      system prompt: understand Filipino and English,
        │                      map what was said onto exposed entities,
        │                      reply in the language the person used
        └── Text-to-speech     ElevenLabs (or Azure if cost bites)
        │
        ▼
Audio back to the Pi speaker
```

Filipino is not a separate box. Speech-to-text has to recognise Filipino (`fil-PH` on Home Assistant Cloud, or a Whisper model that actually hears it). Claude then treats the transcript as the command. A translation service in the middle is extra latency and an extra failure.

The reply voice is ElevenLabs. Pick or design an original voice. Do not upload a real actor's voice, including game voice actors. That breaks the vendor's terms and creates a publicity-rights problem. Personality, if you want one, belongs in the system prompt ("calm, short answers"), not in a cloned celebrity.

## Pipeline settings that matter

| Setting | Value |
|---------|--------|
| Conversation agent | Anthropic. Not the built-in agent, not MCP. |
| Exposed entities | Lights, climate, selected scenes. Not locks, not the router, not cameras, not automations, until each one has a reason. |
| STT language | Filipino if that is the default in the house. Add a second pipeline with an English wake word only if both languages are used all day. |
| TTS | ElevenLabs voice profile. One voice until it is reliable. |
| Satellite VLAN | 10 |
| API keys | Home Assistant secrets store. Never the Git repo. |

Claude is billed per use. A chatty system prompt plus every entity in the house will cost more and behave worse. Expose less.

## Hardware

| Piece | Minimum |
|-------|---------|
| Satellite | Raspberry Pi 4 or 5, USB microphone, small powered speaker |
| Software on the Pi | Wyoming satellite. Home Assistant discovers it under the Wyoming integration. |
| Brain | Already the Mini PC. Do not install a second Home Assistant on the Pi. |
| First client, before the Pi exists | Home Assistant Companion on a phone, same Wi-Fi, VLAN 10. Prove the pipeline, then spend on the Pi. |

The phone is the test rig. The Pi is the furniture.

## Network rules that affect voice

- The Pi must reach the Mini PC on VLAN 10. The IoT VLAN cannot.
- The Mini PC must reach Anthropic and ElevenLabs on the internet. VLAN 10 is allowed to WAN.
- Do not put the API keys on an IoT device.
- If the WAN is down, local Zigbee automations still run. Voice that depends on Claude will not. That is acceptable. Do not "fix" it by giving a cloud model LAN admin rights.

## Failure modes worth writing down

| Symptom | Likely cause |
|---------|----------------|
| Wake word works, no answer | Pipeline has no conversation agent, or the Anthropic key is dead |
| Answer is text only | TTS entity not selected on the pipeline, or the speaker entity is wrong |
| Filipino is heard as nonsense | STT language is English-only |
| Model turns on random devices | Too many entities exposed. Cut the list. |
| Works on the phone, dead on the Pi | Pi is on the wrong SSID (often `iot`) |
