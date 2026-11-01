# Local Voice Assistant for Home Assistant (English, fully offline)

**Your setup:** Home Assistant OS (HAOS) in a VM on a UGREEN DXP2800 (Intel N100).
**End result:** you say "Okay Nabu, turn on the living room light" and it happens, with no cloud, no subscription, and nothing leaving your network.

**What you'll install**

| Piece | What it does | Where it runs |
|---|---|---|
| Speech-to-Phrase | Turns your voice into text (fast, home-control only) | App inside HAOS |
| Piper | Turns replies into spoken voice | App inside HAOS |
| Assist | Built-in Home Assistant engine that understands commands | Already in HAOS |
| Voice PE (Voice Preview Edition) | The speaker/mic you talk to, wake word runs on the device | Hardware, about US$59-69 |
| Whisper (optional) | Slower but open-ended speech-to-text, for later | App inside HAOS |

**Note:** newer Home Assistant versions call "Add-ons" **"Apps"**. They are the same thing. Menu names below use the newer wording; if you see "Add-ons" instead, use that.

---

## Step 0: Before you start (5 minutes)

1. **Make a backup.** Settings → System → Backups → Create backup. If something goes wrong you can roll back.
2. **Check your HA version.** Settings → About. Use a recent version (2026.x is fine). If an update is pending, update first.
3. **Check VM resources.** On your NAS, open the VM app, shut down the HAOS VM, and look at its settings:
   - **CPU:** 4 cores
   - **RAM:** 4 GB or more (6 GB is comfortable if you try Whisper later)
   - **CPU type/model:** if the VM manager lets you pick one, choose **host / host-passthrough** (not a generic or emulated CPU). This matters most for Whisper speed.
   - **Network:** must be **bridged** (the VM gets its own IP on your home network), not NAT.
4. **Give the VM a fixed IP.** In your router, set a DHCP reservation for the HAOS VM. This keeps the voice device from losing HA when the IP changes.
5. Make sure the VM is set to **start automatically** with the NAS. Then start the VM again.

---

## Step 1: Install Speech-to-Phrase (speech-to-text)

1. Settings → **Apps** (or Add-ons) → **Install app** (bottom right) → search **Speech-to-Phrase**.
2. Click it → **Install**.
3. When installed, turn on **Start on boot** and **Watchdog**, then click **Start**.
4. Open the app's **Configuration** tab and make sure the language is **English** (usually `en`/`en_US`; the default is fine).
5. Open the **Log** tab and wait until it says it's ready (the first start trains a model and may take a few minutes on an N100).

## Step 2: Install Piper (text-to-speech)

1. Settings → Apps → Install app → search **Piper** → **Install**.
2. Turn on **Start on boot** and **Watchdog** → **Start**.
3. In **Configuration**, pick an English voice. The default (an `en_US` medium-quality voice) is fine. Medium quality is a good balance on an N100.

## Step 3: Connect them to Home Assistant

1. Go to Settings → **Devices & services**. You should see a **Wyoming Protocol** card (or two) saying "Discovered".
2. Click **Add** / **Configure** on each one, then **Submit**.
3. You should now see Wyoming entries for Speech-to-Phrase and Piper.

If nothing shows up: Settings → Devices & services → **Add integration** → **Wyoming Protocol**. Use host `core-speech-to-phrase` with port `10300` (and `core-piper` with port `10200` for Piper). If it still fails, check that both apps show **Started**.

## Step 4: Create your voice assistant

1. Settings → **Voice assistants** → **Add assistant**.
2. Fill in:
   - **Name:** `Local English`
   - **Language:** English
   - **Conversation agent:** Home Assistant
   - **Speech-to-text:** Speech-to-Phrase (English)
   - **Text-to-speech:** Piper (English)
3. **Create**.
4. Back on the Voice assistants list, click **⋮ → Set as preferred** on this assistant.

## Step 5: Choose which devices voice can control

Speech-to-Phrase **only understands names of things you expose**. Fewer, cleaner names means better recognition.

1. Settings → Voice assistants → **Expose** tab.
2. Click **Expose entities** and tick lights, fans, switches, etc. you want to control.
3. **Rename** anything with a confusing name (click the entity → settings). Good names are short, English, and easy to say: "Living room light", "Bedroom fan".
   - Avoid names like `light_lr_2`, or Malay/Manglish names the engine may mishear.
   - Don't expose sensors you never speak to; they just add noise.
4. **Assign areas** (Settings → Areas & zones) so you can say "turn off the lights in the bedroom".

After changing names or exposure, wait about a minute while Speech-to-Phrase retrains automatically.

## Step 6: Test from your phone first (no hardware needed)

1. Open the **Home Assistant Companion app** on your phone.
2. Tap the **Assist** icon (speech bubble/microphone at the top) → allow the microphone.
3. Say or type: "Turn on the living room light" (use a real device name).
4. Open Settings → Voice assistants → your assistant → **⋮ → Debug** to see what it heard and what it did.

If this works, the server side is done. If not, go to **Troubleshooting** below before buying hardware.

---

## Step 7: Set up the Voice PE (room microphone and speaker)

1. Plug the Voice PE into a USB-C power supply (the box does not include one). It will boot into pairing mode.
2. Make sure the Home Assistant **Companion app** on your phone has Bluetooth turned on and is on your home Wi-Fi.
3. A notification **"New device discovered"** appears. Tap it → follow the prompts to give it your **2.4 GHz Wi-Fi** name and password. It does not support 5 GHz-only networks.
4. In Home Assistant: Settings → Devices & services → **ESPHome**. Click the Voice PE device and **Submit/Add**.
5. On the same device page, find **Assistant** and set it to **Local English** (or "Preferred").
6. The default wake word is **"Okay Nabu"**, running on the device itself. Try it: say **"Okay Nabu, turn on the bedroom fan"**.
7. If an **Update** is offered on the device page, install it. Don't unplug it while updating.

**Placement tips:** put it in the open, off the floor, a few meters from the TV or noisy appliances. Sensitivity can be adjusted on the device page if it triggers too often or not enough.

---

## Optional (later): add Whisper as a backup assistant

Speech-to-Phrase only understands home-control phrases. Whisper understands open-ended speech but is slower on an N100.

1. Install the **Whisper** app (Settings → Apps → Install app → "Whisper"), enable **Start on boot**, and set the language to English. Pick a small model such as **base** or **small**. Start it. Wait for the first-run model download.
2. Add it in Devices & services → Wyoming Protocol.
3. Create a **second assistant** (Voice assistants → Add assistant) named `English Whisper`, using Whisper + Piper.
4. Keep **Local English** as the everyday one. Switch a device to the Whisper one only if you need open-ended phrases.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Wyoming cards never appear | Check both apps show **Started** in Settings → Apps. Add them manually with the host/port values in Step 3. |
| "I didn't understand that" | Use the exact device or area name. Check the entity is **exposed** (Step 5). Look at **Debug** to see what it heard. |
| Mishears a device name | Rename to something simpler. Avoid similar-sounding names. |
| Phone Assist can't use the mic | Use the **Companion app**, not a browser on plain `http://` (browsers block the microphone without HTTPS). Check the app has mic permission. |
| Voice PE won't pair | Use a 2.4 GHz Wi-Fi. Move the phone close. Power-cycle the Voice PE by unplugging it for 60 seconds. |
| Voice PE LED ring keeps spinning | Check HA is reachable (same network, fixed IP). Power-cycle it. As a last resort, re-flash it via Settings → Apps → ESPHome → "Install via cable" (needs a USB cable and a Chrome/Safari browser over HTTPS). |
| Random device turns on by itself | Lower the wake-word sensitivity on the Voice PE device page. Check your automations don't trigger when the assistant state changes to "listening". |
| Slow responses | Confirm 4 vCPU/4 GB RAM, host CPU type if available, and that you are not running Whisper when you only need Speech-to-Phrase. |
| No spoken reply | Check Piper is running and that the assistant's text-to-speech is set to Piper. |

---

## Cheat sheet: where things are in Home Assistant

- **Apps (install/start/logs):** Settings → Apps
- **Wyoming connections:** Settings → Devices & services → Wyoming Protocol
- **Assistants:** Settings → Voice assistants
- **Which devices voice can control:** Settings → Voice assistants → Expose tab
- **Voice PE device:** Settings → Devices & services → ESPHome
- **See what the assistant heard:** Voice assistants → ⋮ → Debug

## Notes on the tech

- Piper's development repo was archived in late 2025, but it still works fine as a Home Assistant app.
- Speech-to-Phrase can't handle free-form phrases (e.g. naming timers freely, arbitrary shopping-list items). Use Whisper for those.
- A local LLM is not recommended on an N100. Assist's built-in engine handles normal home control.
- 
