# Prompt: Discobot takes sounds sent from Drum Foundry

This is a prompt for the maintainer of Discobot to give to Claude in Discobot's repository.

[Drum Foundry](https://aaronvandorn.github.io/Drum-Foundry/) is a drum synthesizer for designing one-shot samples. It has a **Send to Discobot** button for the current sound and a **Send kit to Discobot** button for a whole kit. Each opens Discobot in a new tab and hands the WAV files over, so they land on the drum lanes without being saved and imported by hand. Drum Foundry's half is done and live. Discobot's half is what this prompt asks for. Until it exists, Drum Foundry waits 45 seconds, then tells the user to save the files and use Import Kit instead.

Nothing changes for anyone who opens Discobot in the usual way.

Copy everything below the line.

---

Discobot (this repository) should **accept drum sounds sent from another web app** and put them on its drum lanes through the existing **Import Kit** dialog. The first app to send them is Drum Foundry (https://aaronvandorn.github.io/Drum-Foundry/), but the handshake is not specific to it.

Read `AGENTS.md`, `docs/agents/samples.md` (Import Kit) and `docs/GUEST_PROTOCOL.md` (for the style of a cross-site protocol here) before starting.

## What the user sees

1. In Drum Foundry they press **Send to Discobot** (one sound) or **Send kit to Discobot** (up to 64).
2. Discobot opens in a new tab on the workstation, in whichever project opens normally.
3. The **Import Kit** dialog opens with the files already in it. Each file sits on the lane the sender suggested, or on the lane `matchKit` guesses from its name. Files with neither are listed under "Files on no lane" with their New row switches, exactly as Import Kit does today. The dialog says where the files came from: the sender's name and its site, such as "From Drum Foundry (aaronvandorn.github.io)".
4. They change any lane they like and press Import, or Cancel. Nothing is stored or changed until Import, as with any Import Kit.

## The handshake

Every message is sent with `window.postMessage`. Each is a plain object with `discobotImport: 1` and a `type`. The envelope is separate from the guests' `discobotGuest`, so the two are never mistaken for each other.

The sender opens Discobot's address with `#receive` on the end (`https://iw978599.github.io/discobot/#receive`) using `window.open`, so the new tab has `window.opener`.

**Discobot to sender**

| `type` | Fields | When |
|---|---|---|
| `ready` | none | Once, when the address had `#receive`, there is a `window.opener`, and the studio has opened its project and can show Import Kit. Sent to `'*'`: the sender's address is not known yet, and the message carries nothing private. |
| `done` | `imported` (number of files put on a lane or a new row; 0 if the user cancelled) | When the Import Kit dialog that the sounds opened closes. |
| `error` | `message` (string, short, for the user) | When the sounds cannot be shown: nothing valid in them, too many, too large, or not decodable. |

**Sender to Discobot**

| `type` | Fields | Meaning |
|---|---|---|
| `sounds` | `v` (1), `from` (string, the app's name), `files`: array of `{ name, mime, lane, data }` | Show these in Import Kit. `name` is a file name ending in `.wav`. `mime` is `'audio/wav'`. `lane` is a `DrumInstrument` id (`kick`, `snare`, `clap`, `closedHH`, `openHH`, `snare2` for Low Tom, `ride` for High Tom, `crash` for Cymbal) or `null`. `data` is an `ArrayBuffer`, passed in the transfer list. |

### Rules for Discobot

- Act only when the address had `#receive` and `window.opener` is set. Take `#receive` off the address straight away (`history.replaceState`), so a reload or a bookmark does not wait for a sender, and so it never reaches `App`'s choice of page.
- Accept `sounds` only when `event.source === window.opener`, only once per page load, and only from an origin that is `https`, or `http://localhost` / `http://127.0.0.1` while developing, and is not Discobot's own. Remember that origin and use it as the target of `done` and `error`.
- Everything in the message is untrusted. Check every field's type. Take at most `MAX_KIT_FILES` files, each 1 byte to 8 MB, and skip any that is not a WAV that `decodeAudioData` accepts. Tidy each name as a sample's name is tidied anywhere else. Accept a `lane` only if it is one of the eight ids above, and otherwise treat it as `null`. If no file survives, answer `error` and show nothing.
- Build `File` objects from the buffers and open `KitImportDialog` with them, passing the suggested lanes as `initial`. For files without a suggestion, use `matchKit` for their names. Where two files suggest one lane, keep the first; the other goes to "Files on no lane". From there everything happens as for an ordinary Import Kit: the same storing, the same rows and the same undo step.
- Ask nothing with `window.confirm`; the dialog itself is the confirmation. Nothing is remembered about the sender's site, and the next send asks again by opening the dialog again.
- Do not load anything from the sender's site and do not frame it. This only receives messages.
- A sender that sees no `ready` within 45 seconds gives up and tells the user to use Import Kit by hand. It also ignores any message whose `event.source` is not the tab it opened or whose origin is not Discobot's.

### The files Drum Foundry sends

Drum Foundry sends WAVs at the sample rate and bit depth the user chose for saving (22.05 to 96 kHz; 8, 16, 24-bit or 32-bit float; mono or stereo), so that what arrives is what was heard. If a file would be over 4 MB (`MAX_EMBEDDED_SAMPLE_BYTES`, the most a project file carries for one sample), it sends that file as 16-bit instead. A sound is at most 8 seconds, within `MAX_DRUM_SAMPLE_SECONDS`.

Each WAV carries an extra RIFF chunk called `dfpt` with Drum Foundry's own settings, which Discobot can ignore: `decodeAudioData` skips chunks it does not know.

File names lead with the drum they are (`kick-…`, `snare-…`, `closed-hat-…`, `open-hat-…`, `cymbal-…`, `low-tom`, `perc-…`, `fx-…`), and in a kit they are numbered (`01-kick-long-808.wav`). So `matchKit` alone already places them, even in a kit saved as a .zip and imported by hand.

## Testing

- Add a small sender page beside `docs/guest-example.html` (for instance `docs/send-example.html`), served by `server/dev.ts` as the example guest is. It opens Discobot with `#receive`, waits for `ready` and sends two or three tiny generated WAVs, one with a `lane`, one with `lane: null`, and one with a bad `lane`. A browser test then checks:
  - the dialog opens with the right lanes;
  - Import stores them and the sender receives `done` with the count;
  - Cancel answers `done` with 0;
  - a second `sounds` is ignored;
  - a message from another window is ignored;
  - opening Discobot without `#receive` sends nothing.
- A unit test for the checks on the message (types, sizes, lanes, names) beside the other untrusted-input tests.
- Document the handshake in a short `docs/SEND_PROTOCOL.md`, link it from `AGENTS.md` and `docs/agents/samples.md`, and add the item to `docs/ROADMAP.md`.
