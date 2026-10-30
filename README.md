# BPTS — Blueberry Pi ToothStream

Ansible project for configuring a Raspberry Pi as a Bluetooth A2DP audio receiver. BPTS is based on a working setup on a Raspberry Pi 3 Model B running Debian 12 (Bookworm), with audio from an Android phone sent through the Pi's 3.5 mm jack to an amplifier.

Originally created by [mr-mop](https://github.com/mr-mop). See [NOTICE](NOTICE) for attribution details.

## Requirements

- Raspberry Pi 3 Model B with onboard Bluetooth and the 3.5 mm analogue output
- Debian 12 (Bookworm)
- A dedicated, good-quality 5 V / 2.5 A micro-USB power supply
- Ansible on the control machine
- SSH access to an existing Pi account with sudo privileges

Avoid powering the Pi from an amplifier's USB port unless it is rated to supply the Pi's requirement. The reference setup produced undervoltage warnings and Bluetooth UART errors when powered that way; those messages stopped after switching to a dedicated supply. Use a short, good-quality cable to limit voltage drop.

The WirePlumber file uses the Lua drop-in format from the reference Bookworm installation. Other Debian releases may use a different format.

## Configure the inventory

Copy the example inventory and edit the address and SSH account:

```sh
cp inventory/hosts.ini.example inventory/hosts.ini
```

By default, the audio services run as the SSH account (`ansible_user`). You can change that in `group_vars/all.yml`. Set the Bluetooth alias there too.

## Apply the playbook

```sh
ansible-playbook playbook.yml
```

Reboot the Pi after the first run. Then pair with the Bluetooth name set in `bluetooth_audio_alias`, connect from the phone, and play audio.

## What it configures

- Installs BlueZ, PipeWire, WirePlumber, Bluetooth audio support, and audio utilities.
- Adds the audio account to the `audio` group and enables lingering so PipeWire can run at boot without an interactive login.
- Enables the PipeWire sockets and WirePlumber user service for that account.
- Keeps a minimal PAM session open for the audio account. On the reference Pi, WirePlumber registered its Bluetooth A2DP endpoints only while an SSH login session existed; this service uses the same PAM session stack so Bluetooth remains available after SSH disconnects. It runs `sleep` only and does not open a shell or network listener.
- Selects the Pi's analogue output using ALSA.
- Configures WirePlumber to accept Bluetooth A2DP audio and offers the codec preferences from `group_vars/all.yml`. Negotiated codecs depend on both devices and the available codec implementations.
- Enables Bluetooth at boot and runs a system service that powers on the adapter, enables pairing and discovery, and sets the Bluetooth alias.
- Manages `dtparam=krnbt=off` in `/boot/firmware/config.txt` for the UART setup used on the reference Pi 3. Set `bluetooth_audio_manage_krnbt: false` to leave that boot setting untouched, or `bluetooth_audio_krnbt_off: false` to remove it.

The playbook does not include the original hostname, Linux username, phone Bluetooth address, or Pi serial number. Pairing is performed from the phone and stored by BlueZ on the Pi.

## License

Apache License 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
